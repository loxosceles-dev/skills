---
name: pr-reviewer
description: Post a PR review on GitHub. Fetches the diff, runs parallel sub-agents, validates issues before posting, and leaves inline comments. The developer fixes; you review again until satisfied. Never commits, branches, or modifies code.
type: pattern
---

# PR Reviewer

**This is a strict pattern.** Follow the pipeline exactly. When any other skill contradicts the rules below, this skill takes precedence.

---

## Skill handover — READ THIS FIRST

This skill covers **posting** a review. It does NOT cover replying to comment threads after fixes are pushed.

When the dev pushes fixes and you need to reply to each comment thread:
**STOP. Switch to the `pr-fixer` skill.** That skill owns the reply workflow: triage comments → fix one by one → reply to each thread with commit hash.

The boundary is clear:
- `pr-reviewer` = posting the initial review and re-reviewing the diff
- `pr-fixer` = replying to individual comment threads after fixes land

---

## Role

You are a reviewer. You read code and leave comments. You do not fix anything. You do not commit. You do not branch. The developer does the work; your job is to tell them what needs to change and hold the bar until it is met.

A review cycle is: you comment → dev pushes changes → you re-review → repeat until satisfied → approve.

---

## Composition

This skill has two entry modes:

- Standalone PR review: run the full pipeline below.
- Composed under `code-reviewer`: `preflight` still runs first, but if the caller already produced Standards and Spec findings with the `code-review` skill against the same PR diff, skip this skill's `standards` and `spec` stages. Reuse those findings, then run `bug-detector`, `logic-security`, `validation`, and `post`.

Never run two independent Standards and Spec passes against the same diff. That creates duplicate comments and conflicting verdicts.

---

## Validation and findings retention

Validation establishes whether a candidate is a real actionable issue; it does
not decide whether the developer should be told about a verified issue.

- Retain every verified actionable finding discovered while reviewing the change
  or necessary surrounding context. Pre-existing code, an unchanged line, lower
  severity, or deferring the fix does not make a finding disappear. Label its
  origin, evidence, severity, and disposition so the developer can decide what to
  fix. Do not turn review into an unrelated whole-repository audit.
- Verify factual correctness and reachability/material relevance. For standards
  findings, verify the cited rule applies. A reproducible failing condition is
  enough; a bug need not fail for every possible input.
- A disproven or unsupported candidate is not a verified finding. Investigate
  before making a claim; exclude false positives, unsupported speculation, and
  tooling-only noise. Do not present a rejected-candidate list or validation
  discard log as the review result.
- An existing thread avoids a duplicate post, not visibility: reference that
  thread in the local findings and state whether it is resolved or outstanding.
- Post a new finding inline when GitHub provides a valid diff anchor. Otherwise
  retain it under local `## Findings` with an exact file/line, evidence, and
  `local — outside diff` or the actual posting blocker. Do not invent an anchor
  or hide a finding because it cannot be posted inline.
- The final local report lists every verified actionable finding, including
  posted, existing-thread, and local-only items. Zero inline comments is not a
  clean review. Use `No verified actionable findings.` only when that is true.

---

## Pipeline

```
preflight -> standards + spec + bug-detector + logic-security (parallel) -> validation (parallel per issue) -> post
```

| Stage | Role | Model | Trigger |
|-------|------|-------|---------|
| preflight | lead-dev | Sonnet | Always first |
| standards | lead-dev | Sonnet | After preflight passes |
| spec | lead-dev | Sonnet | After preflight passes (parallel with standards) |
| bug-detector | lead-dev | Opus | After preflight passes (parallel) |
| logic-security | lead-dev | Opus | After preflight passes (parallel) |
| validation | lead-dev | Opus (bugs) / Sonnet (guidelines) | After all 4 review agents complete |
| post | lead-dev | Sonnet | After all issues validated |

---

## Subagent Stage Configuration

```yaml
stages:
  - name: preflight
    role: lead-dev
    model: claude-sonnet-4-5
    prompt: |
      Check all of these and stop immediately if any fail:
      - Is the PR closed? → output STOP: closed
      - Is it a draft? → output STOP: draft
      - Is it a trivial automated change (version bump, generated file, doc typo)? → output STOP: trivial

      If all pass, output: PROCEED
      Then fetch and output:
        gh pr view <PR_NUMBER> --json title,body,headRefOid,baseRefName,headRefName,files,comments,reviews
        gh pr diff <PR_NUMBER>
      Output the full SHA (40 chars) from headRefOid. Never abbreviate it.

      Summarise existing comments as: [existing] <file:line> — <reviewer> — <summary>
      This list is passed to all review agents so they can behave like a normal reviewer who has read the thread.

      Collect all guideline files:
        - Root CLAUDE.md, AGENTS.md, KIRO.md
        - Any of those files in directories containing changed files
        - Any additional standards files explicitly referenced in CLAUDE.md or AGENTS.md

      After collecting, you MUST output — before proceeding to any other stage:
        [preflight] Standards files collected: <list each file path found>
        [preflight] Standards files NOT FOUND: <list any files referenced but missing, or NONE>

      STOP HERE. Do not proceed to parallel stages until this output is visible.

  - name: standards
    role: lead-dev
    model: claude-sonnet-4-5
    depends_on: [preflight]
    prompt: |
      You are running the Standards axis of the code-review skill.
      Input: the diff, PR title+body, ALL guideline file contents listed in preflight's "Standards files collected" output, and the existing comments list from preflight.
      Task: audit for violations of every collected standards file.
      Scoping rule: only apply a guideline to files that share its directory path.
      Existing comments: behave like a normal reviewer who has read the thread. Add new issues, agree with prior comments, challenge them, or add nuance. Do not re-post the identical point verbatim.
      Output: list of issues with exact guideline quote and file:line. If none, output NONE.

  - name: spec
    role: lead-dev
    model: claude-sonnet-4-5
    depends_on: [preflight]
    prompt: |
      You are running the Spec axis of the code-review skill.
      Input: the diff, PR title+body, and existing comments list from preflight.
      Task: does the code faithfully implement what the PR title+body says it does?
      Look for: missing cases, wrong behavior, incomplete implementation.
      Existing comments: behave like a normal reviewer who has read the thread. Add new issues, agree, challenge, or add nuance. Do not re-post the identical point verbatim.
      Output: list of issues with file:line. If none, output NONE.

  - name: bug-detector
    role: lead-dev
    model: claude-opus-4-5
    depends_on: [preflight]
    prompt: |
      You are a bug detector. Start from the diff and read the surrounding context needed to verify a candidate.
      Input: the diff, PR title+body, and existing comments list from preflight.
      Task: scan for provable bugs and wrong results under reachable conditions.
      The diff is the entry point; read necessary surrounding context to validate a candidate. Retain verified pre-existing issues encountered there and label their origin.
      Existing comments: behave like a normal reviewer who has read the thread. Add new issues, agree, challenge, or add nuance. Do not re-post the identical point verbatim.
      Output: list of bugs with file:line and explanation. If none, output NONE.

  - name: logic-security
    role: lead-dev
    model: claude-opus-4-5
    depends_on: [preflight]
    prompt: |
      You are a logic and security reviewer. Start with the changed code and inspect necessary surrounding context.
      Input: the diff, PR title+body, and existing comments list from preflight.
      Task: find provable logic errors, security issues, and incorrect behavior relevant to the review. Retain verified pre-existing issues encountered along the way, labelled separately from introduced issues.
      High signal only. Cite evidence and reachable conditions; do not speculate.
      Existing comments: behave like a normal reviewer who has read the thread. Add new issues, agree, challenge, or add nuance. Do not re-post the identical point verbatim.
      Output: list of issues with file:line and explanation. If none, output NONE.

  - name: validation
    role: lead-dev
    model: claude-sonnet-4-5
    depends_on: [standards, spec, bug-detector, logic-security]
    prompt: |
      You are a validation agent. You receive all findings from the review stages that actually ran.
      In standalone mode that is standards, spec, bug-detector, and logic-security.
      When composed under `code-reviewer`, Standards and Spec findings may come from the `code-review` skill instead of this skill's own `standards` and `spec` stages.
      Apply the "Validation and findings retention" policy above to every candidate:
        1. Verify the factual claim and reachable failing condition or material relevance.
        2. Verify the guideline applies to the file's path (standards issues only).
        3. Keep every verified actionable issue, labelled introduced or pre-existing.
        4. Reference existing threads instead of posting identical comments again.

      Use Opus reasoning for bugs/logic issues. Use Sonnet reasoning for guideline violations.

      Output confirmed findings only:
        CONFIRMED: <issue summary> — <file:line> — <introduced/pre-existing> — <new/existing thread> — <evidence>

      Exclude disproven or unsupported candidates; do not emit a rejected-candidate report.
      Do not post comments in this stage.

  - name: post
    role: lead-dev
    model: claude-sonnet-4-5
    depends_on: [validation]
    prompt: |
      You are the posting agent. You receive confirmed findings from validation.

      For each confirmed issue, assign one stance:
        MUST FIX — blocks approval, real issue, clear problem
        CONSIDER — valid concern, not a blocker, defer to dev
        PUSHBACK — post as a question, not a demand

      Post one inline comment per new issue with a valid PR-diff anchor using:
        gh api repos/<owner>/<repo>/pulls/<PR_NUMBER>/comments \
          -f body="<comment>" \
          -f commit_id="<full-40-char-sha>" \
          -f path="<file>" \
          -F line=<line>

      Each comment must be focused on the specific code at that line. No generic summaries.

      Link format in comment body — strictly:
        https://github.com/<owner>/<repo>/blob/<full-40-char-sha>/path/to/file.ts#L66-L73
      Full SHA always — never abbreviated, never shell-interpolated.

      Do not post a separate summary comment. The inline comments are the review.
      If a verified finding is outside the diff, reference it in local Findings with exact file/line and evidence. Reference existing threads rather than duplicating them. If posting fails, retain the finding locally with the actual blocker.
      Present every verified actionable finding locally with its origin, stance, and posted/existing-thread/local status. No inline comments does not mean no issues.
      Only if there are no verified actionable findings or outstanding issues to report, post one PR comment saying "No new issues found." Do not output a rejected-candidate section or discard log.

      Never fix code. Never commit. Never create branches. Never approve while MUST FIX items are unresolved.
```

---

## 🚨 Mandatory: announce every step before you take it

Before every action, output a single line stating which stage you are running. No exceptions.

Examples:
- `[pr-reviewer preflight] Checking PR status and fetching diff`
- `[pr-reviewer standards] Auditing guideline compliance`
- `[pr-reviewer spec] Checking spec faithfulness`
- `[pr-reviewer bug-detector] Scanning for bugs`
- `[pr-reviewer logic-security] Checking logic and security`
- `[pr-reviewer validation] Validating issue N — <description>`
- `[pr-reviewer post] Posting inline comment on file.ts#L42`

If you skip this line you are not following this skill. The developer uses these to verify you are on track and to stop you if you are not.

---

## Pre-flight Outcomes

| Output | Action |
|--------|--------|
| `STOP: closed` | End. Do nothing. |
| `STOP: draft` | End. Do nothing. |
| `STOP: trivial` | End. Do nothing. |
| `PROCEED` | Continue to parallel stages. Existing comments are passed to all review agents to prevent duplication. |

---

## High Signal Only

False positives erode trust. Verify candidates against evidence before reporting them:

Flag only:
- Code that will definitely fail to compile or parse
- Code that demonstrably produces wrong results under reachable conditions
- Clear, unambiguous guideline violations you can quote exactly
- Provable security issues relevant to the review, labelled introduced or pre-existing

Never flag:
- Something that looks like a bug but might be correct
- Nitpicks a senior engineer would not raise
- Issues a linter already catches
- General code quality concerns unless required in a guideline
- Issues silenced by a lint-ignore comment
- Style preferences
- Missing test coverage as a general concern

---

## Re-review Cycle

When the dev pushes and asks for re-review:
1. Re-run from preflight
2. Focus the diff on what changed since your last review
3. Check that all MUST FIX items from the last round are addressed
4. If a MUST FIX is not addressed, re-post it — do not let it drop
5. If new issues appear in the new diff, post those too
6. Approve only when all MUST FIX items are resolved

---

## What You Never Do

- Never fix code
- Never commit
- Never create branches
- Never open issues (that is the dev's job after approval)
- Never request re-review yourself (let the dev do it)
- Never post a comment that restates verbatim what another reviewer already said
- Never post a detached summary comment — comments belong on the code they're about
- Never approve while MUST FIX items are unresolved
- Reference an already-reported issue's existing thread instead of duplicating its post; keep outstanding findings visible locally

---

## Progressive Improvement

If the developer corrects a behavior that this skill should have prevented, suggest a specific amendment to this skill to prevent the same correction in the future.
