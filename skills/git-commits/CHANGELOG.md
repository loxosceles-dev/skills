## 2026-10-17 — Classify branches and documentation by repository policy

**Problem:** Small feature fixes were treated as direct-dev maintenance, while all planning documentation was assumed to be untracked.
**Fix:** Classify features/fixes separately from standalone setup/docs, respect repository/user workflows, and distinguish local planning from durable documentation without imposing one layout universally.
**Verified:** Static consistency review with branch-policy; no unrelated working-tree changes were staged.
