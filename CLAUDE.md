# softvoyagers/claude-skills

Claude Code marketplace providing multi-agent workflow commands and skills.

## Repository Structure

| Path | Description |
| ---- | ----------- |
| `.claude-plugin/` | Marketplace and plugin configuration for Claude Code |
| `commands/` | Slash command definitions — each `.md` file is a multi-agent orchestration workflow |
| `skills/` | Skill definitions — each subdirectory contains a `SKILL.md` with frontmatter |
| `package.json` | Package metadata and versioning |

## Commands

All commands follow the **Orchestrator pattern**: Claude coordinates specialized agents across phases with quality gates.

- `bug-fix.md` — 4-phase bug fix (Diagnose → Fix → Validate → Ship)
- `new-feature.md` — 5-phase feature implementation (Discovery → Architecture → Implement → Review → Ship)
- `discover-feature.md` — Read-only customer needs analysis from 5 perspectives
- `discover-feature-and-deliver-most-wanted.md` — Discovery + implementation of highest-impact feature
- `research-issue.md` — 4-phase issue research producing root cause hypotheses

## Skills

- `use-gamma-slides/` — Gamma API slide deck generation

## Conventions

- Commands use `$ARGUMENTS` for user input
- Agent types: `Explore` (read-only research), `general-purpose` (can write code)
- Test naming: `MethodName_Scenario_ExpectedResult`
- Test structure: Arrange / Act / Assert
- Minimal change surface — only touch what's necessary

## Modifying Commands or Skills

- Keep commands tech-stack agnostic — they should auto-detect project tooling
- Skills must have a `SKILL.md` with YAML frontmatter (`name`, `description` required)
- Skill directory name must match the `name` field in frontmatter
- Update `package.json` version when releasing changes
- Update `.claude-plugin/plugin.json` version to match
