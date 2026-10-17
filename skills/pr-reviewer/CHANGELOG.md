## 2026-10-17 — Retain verified findings in both reviewer entry modes

**Problem:** The composed reviewer agent retained an obsolete rejected-findings section and introduced-code-only validation. The standalone skill also suppressed pre-existing issues, so changing only a heading would leave the behavior intact.
**Fix:** Update the code-reviewer agent and this skill together: retain all verified actionable findings, label origin and posting status, reference existing threads, and report outside-diff findings locally instead of inventing GitHub anchors. Keep evidence validation and false-positive filtering without a rejected-candidate report.
**Verified:** Static policy/consistency checks and package validation. Deployment hash checks are recorded in the private rollout log; no live PR comments were posted as a test.

## 2026-08-31 — Fixed code-reviewer composition and pr-reviewer preflight commands

**Problem:** The reviewer workflow tried to concatenate Matt Pocock's `code-review` and the local PR review pipeline without a shared source of truth. That caused duplicated Standards and Spec passes, let the top-level reviewer assume local `HEAD` matched the PR diff, and left `pr-reviewer` with invalid preflight instructions (`headRefSha` and `gh pr review --json ...`).

**Fix:** Documented the composed `code-reviewer` entry mode for `pr-reviewer`, corrected the pipeline description so the active stages are explicit, and fixed preflight to use `gh pr view --json ...,headRefOid,...,comments,reviews`. Rewrote the `code-reviewer` agent prompt so PR preflight runs first, Matt's review owns Standards and Spec, and `pr-fixer` only runs on explicit remediation requests.

**Verified:** Compared the rewritten flow against the working `critic-dialogue` pipeline structure, checked `gh pr view --help` and `gh pr review --help`, and kept the deployed reviewer agent configs in sync.
