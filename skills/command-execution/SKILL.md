---
name: command-execution
description: Guidelines for executing commands and running scripts. Follow when running shell commands, installing packages, or using project scripts.
type: guideline
---

# Command Execution Guidelines

**This is a strict guideline.** Follow these rules exactly.

Guidelines for AI agents when executing commands and running scripts.

---

## Core Rules

### 1. Respect the Project's Declared Execution Environment

Read the project's `AGENTS.md`, setup/architecture decisions, toolchain files and root scripts before running tools. The project's authorized setup decides where commands execute:

- **Container-managed project:** run its package, build and test commands inside its existing devcontainer. Do not install its dependencies on the host.
- **Host-native project:** use its declared isolated toolchain (for example mise plus a project-local `.venv` or `node_modules`). Run its root scripts on the host; do not install project packages globally.
- **Unclear or conflicting setup:** stop and ask. Do not migrate execution modes or remove a devcontainer as a side effect of running a command.

For a container-managed project:

```bash
# ✅ Correct: execute inside the container
docker exec -it <container_name> pnpm install
docker exec -it <container_name> pnpm build
docker exec -it <container_name> pytest

# ❌ Wrong for that container-managed project
pnpm install          # bypasses its declared container environment
pip install requests  # may modify an unrelated host Python
npm run build         # may use the wrong host Node version
```

**Host administration is a separate scope.** These tools may run on the host when authorized:
- `git` — version control is host-level
- `docker` / `docker compose` — managing containers
- `gh` — GitHub CLI for repo operations
- `chezmoi`, `uv`/`uvx`, APM and credential tooling — host configuration and skill administration, not product dependency installation
- File operations (`cat`, `ls`, `grep`, `find`, etc.)

**Cross-project work does not authorize project execution or migration.** Inspect each project's current state and permissions first. Projects own their refreshes; host boot/login automation must not update them.

### 2. Use Project's Developer API

**Always use root `package.json` scripts:**

```bash
# ✅ Correct
pnpm lint
pnpm provision:dev
pnpm build:frontend

# ❌ Wrong
cd infrastructure && npx cdk deploy
cd frontend && npm run build
```

**Why**: Root scripts are the developer API. Bypassing them means the API can break without anyone noticing.

### 2. Never Use `cd`

```bash
# ❌ Wrong
cd infrastructure && pnpm synth

# ✅ Correct
pnpm synth:dev

# ✅ Or use working_dir parameter
execute_bash(command="pnpm synth", working_dir="infrastructure")
```

**Why**: `cd` doesn't persist. Use root scripts or `working_dir` parameter.

### 3. Environment Convention

`.envrc` sets `ENVIRONMENT=local` for local dev server detection. Infrastructure commands targeting AWS use explicit `:dev` or `:prod` suffixes in root scripts, which set `ENVIRONMENT=dev` or `ENVIRONMENT=prod` respectively.

```bash
# ✅ Correct — suffix determines the target environment
pnpm deploy:dev
pnpm ses-template:prod
pnpm ssm-upload:dev

# ❌ Wrong — never rely on .envrc ENVIRONMENT for infra commands
pnpm deploy          # would use ENVIRONMENT=local, which is wrong
ENVIRONMENT=dev pnpm --filter=infrastructure run deploy  # bypasses root API
```

**Root scripts are idempotent**: if an operation can be both "create" and "update" (like SES templates), the script handles both — no separate `:deploy` / `:update` variants.

### 4. Discover Scripts First

```bash
# Check available scripts
cat package.json | grep -A 50 '"scripts"'
```

Look for: `lint`, `test`, `build`, `provision:*`, `deploy:*`

---

## Installation

```bash
# ✅ Workspace root (shared)
pnpm add -D -w eslint

# ✅ Specific workspace
pnpm add -D eslint --filter frontend

# ❌ Wrong
cd frontend && pnpm add eslint
```

---

## When Direct Execution is Necessary

1. Check if root script exists first
2. Use `working_dir` parameter if available
3. Document why you're not using root scripts

---

## Summary

**Golden Rules:**
1. ✅ Use root `package.json` scripts (the developer API)
2. ✅ Never use `cd` in commands
3. ✅ Check available scripts first
4. ✅ Use `working_dir` parameter for direct execution

**Remember**: Root scripts are the project's public API. Bypassing them breaks the contract.

---

## Progressive Improvement

If the developer corrects a behavior that this skill should have prevented, suggest a specific amendment to this skill to prevent the same correction in the future.
