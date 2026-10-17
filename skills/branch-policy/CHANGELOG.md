## 2026-10-17 — Distinguish project changes from infrastructure maintenance

**Problem:** Requiring a feature branch for every documentation/setup change created unnecessary branches and misapplied a production workflow to main-only infrastructure.
**Fix:** Keep feature/fix work on branches, allow authorized standalone setup/docs on dev, and honor explicit repository/user rules for infrastructure and dirty worktrees.
**Verified:** Reviewed the policy and aligned the pre-push reminder. No branches were changed to qualify for publication.
