---
name: branch-policy
description: Branch model and merge flow for coding projects. Features merge to dev via PR, dev to main via PR; standalone setup and documentation go directly to dev. Use when branching, committing, pushing, or opening PRs, subject to the repository's actual workflow and explicit user restrictions.
---

# Branch Policy

Repository-specific workflow and explicit user restrictions take precedence over
this default coding-project model. Inspect the actual branches and working-tree
state first. Main-only infrastructure repositories may use a direct-main workflow
when authorized; do not invent a dev branch or transplant a production-release
process into a repository that does not use it.

## Branch Model

```
main        ← production. Updated ONLY by PR from dev. Never push directly.
dev         ← integration branch. Feature PRs plus standalone setup/docs commits.
feat/*      ← feature work, branched FROM dev
fix/*       ← bug fixes, branched FROM dev
chore/*     ← maintenance, branched FROM dev
```

## Branch Features; Standalone Setup and Docs Go to Dev

**Feature work, bug fixes, and refactors start on a branch from dev.**

Project setup and documentation belong with their related feature branch, or go
directly to dev when done separately. A small feature fix is still feature work;
size alone does not turn it into setup. Existing work stays on its current branch.

If you're on `dev` and starting feature work, create the feature branch first:
```bash
git checkout -b feat/descriptive-name dev
```

## One Feature = One Branch

A feature branch lives until the feature is complete. All related work happens on that branch:
- PR review feedback → fix on the same branch, push
- Copilot comments → fix on the same branch, push
- Multiple rounds of iteration → same branch

Do NOT create a new `fix/` branch for issues found during review of an open PR. Stay on the feature branch until the PR is merged.

A new branch is only needed when starting **genuinely new, unrelated work**.

## Flow

1. `git checkout -b feat/thing dev` — create branch BEFORE feature code changes
2. Make commits on the feature branch
3. Push feature branch, open PR **to dev** (never to main)
4. Address review feedback on the same branch, push again
5. Merge PR to dev when approved
6. When dev is stable, open PR **from dev to main**

## Push Rules

| Action | Allowed? |
|---|---|
| Push to feature branch | ✅ Always |
| Push to `dev` | ✅ For authorized standalone setup/docs; otherwise only with explicit user permission. |
| Push to `main` | ❌ Never. Main is updated only by merging a PR. |

Classify the change before acting: feature/fix work needs a branch; standalone
setup/docs can stay on dev. Commit and push only within the user's authorization.
If branch or dirty-state restrictions block publication, leave the changes local
and report the blocker; never move or discard unrelated work to qualify for push.

## PR Base Branch

| Source branch | PR base |
|---|---|
| `feat/*`, `fix/*`, `chore/*` | `dev` |
| `dev` | `main` |

If you're about to create a PR with `--base main` from a feature branch, **STOP** — the base must be `dev`.

---

## Common Mistakes to Avoid

1. **Ignoring the actual workflow** — Check the branch and working tree first; use repository/user rules before this default model.
2. **Treating a small feature fix as setup** — Features/fixes still use a branch and PR; standalone setup/docs are the exception.
3. **Opening PR to main from a feature branch** — Feature branches always target dev.
4. **Skipping classification** — Decide whether the task is feature/fix work or standalone setup/docs before branching.
5. **Creating a new branch for review fixes** — Stay on the current feature branch. One feature = one branch until merged.
