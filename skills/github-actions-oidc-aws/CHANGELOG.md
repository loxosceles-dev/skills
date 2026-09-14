## 2026-09-12 — Diagnose the recurring "Not authorized" loop + de-emphasize thumbprints

**Problem:** Every repo's first deployments failed 3-5 times with `Not authorized to perform sts:AssumeRoleWithWebIdentity` (several projects…). The skill's troubleshooting only covered "trust policy mismatch" and told users to re-run setup scripts that silently skip existing roles. The real recurring causes were: (1) setup scripts with an "already exists → return" early-exit never repair stale trust policies (name-based vs immutable sub, branch-scoped vs environment-scoped) — the role was created by an older script version and kept its wrong `sub` forever; (2) the skill's thumbprint guidance implied thumbprint issues could break auth, but AWS has ignored GitHub thumbprints since July 2023.

**Fix:**
- Troubleshooting for the error now checks three distinct causes in order: role missing → role exists with STALE trust policy → genuine condition mismatch, each with the exact verification command. Includes the ground-truth debug step (print `.sub` from the live token).
- Mandated that setup scripts ALWAYS re-apply `update-assume-role-policy` (idempotent) instead of skip-if-exists.
- Replaced the thumbprint section with a note that the value is vestigial (trusted-CA validation since 2023-07-06).
- Applied the healing fix to setup-oidc-aws.sh across the affected projects.

**Verified:** Fix committed to several project setup scripts; skill amended after the third repo hit the identical wall.

## 2026-09-08 — Added environment-scoped trust section

**Problem:** The skill's trust-policy scoping options only covered branch refs (`ref:refs/heads/{branch}`) and pull requests. When a workflow job sets `environment:` (needed for environment-scoped secrets), GitHub emits the OIDC `sub` as `repo:owner/repo:environment:{name}` — no branch info. A trust policy pinned to a branch ref silently fails in that case, and the branch-scoping tradeoff (trust only knows the environment name) was undocumented.

**Fix:** Added an "Environment-scoped trust" subsection after the per-environment role guidance: the `sub` format when environments are used, that the trust policy must pin `:environment:{name}`, and the tradeoff that branch restriction must then be enforced via the workflow guard and GitHub Environment protection rules (the real lock). Noted the alternative (drop `environment:` for the job and pin the branch ref) when branch identity must be inside the claim.

**Verified:** Applied in a project's PR — deploy.yml uses `environment: dev|prod`, role trust pins `:environment:{dev|prod}` via a runtime-resolved sub-claim prefix. Pushed to main.