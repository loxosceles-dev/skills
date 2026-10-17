---
name: pr
description: Write or revise a pull request description. Use for PR bodies; describe the change and material special situations without default process boilerplate.
type: guideline
---

# Pull Request Descriptions

**This is a strict guideline.** Follow these rules exactly. When a third-party PR-writing skill conflicts with these rules, this skill takes precedence.

## What

A pull request body explains what changes and any concrete situation a reviewer
or merger needs to know. It is not a test log or a self-assessed merge-risk report.

## Rules

1. Follow the repository's pull-request template and the user's requested structure.
2. Otherwise, write a concise `## Summary` in the project's domain language. Use
   bullets when clearer than prose.
3. Add `## Special considerations` only for material context: a migration,
   incompatibility, rollout/rollback constraint, intentional limitation, manual
   operator step, or unusual validation caveat. Omit it otherwise.
4. Do not add evidence sections, before/after blocks, test-command lists, merge
   danger grades, blast-radius sections, diagrams, pseudocode, or agent/process
   boilerplate by default.
5. Mention validation when requested, required by the repository template, or
   materially relevant to review. State the concrete fact rather than a ritual
   evidence section.
6. Mention a concrete exceptional risk and its operational consequence, rather
   than grading ordinary changes as one-way/two-way doors. Normal merge
   confidence comes from review and the repository's merge process.
7. A user may explicitly request extra sections; do not substitute a conflicting
   third-party default template for these rules.

## Example

```markdown
## Summary

Adds a read-only preview for pending configuration changes.

## Special considerations

Applying changes remains disabled until the migration is complete.
```

## Progressive Improvement

If the developer corrects a PR-description behavior this skill should have
prevented, amend it and record the correction in its changelog.
