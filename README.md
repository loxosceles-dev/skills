# skills

Universal coding and project-setup skills and agents for any development project.

General engineering practices — conventions, patterns, and tooling that apply regardless of stack. Safe to share with other developers and install in any project.

## Install into a project

```sh
npx -y skills add loxosceles-dev/skills --agent claude-code github-copilot codex kiro-cli -y
```

## Structure

```
skills/            one directory per skill
  {skill-name}/    flat layout
    SKILL.md       the skill
    scripts/       helper scripts where applicable
    hooks/         git hooks where applicable
    CHANGELOG.md   changelog where applicable

.apm/              APM package: apm.yml, plugin.json, agent sources (.agent.md)
```

## Agents

Shipped with the package, installable per project.

| Agent | Purpose |
|-------|---------|
| `lead-dev` | Primary development agent — loads project skills, defers to specialists |
| `code-reviewer` | Multi-stage PR review — reviews the diff, validates issues, posts inline comments |
| `critic` | Adversarial review of plans, designs, code, decisions |
| `planner` | Design and architecture docs, auto-runs the critic pipeline |

## Skills

The `skills/` directory covers git workflow, code review, frontend code quality, CLI and infrastructure patterns, testing, and skill authoring. Browse `skills/` for the full list.

## License

MIT