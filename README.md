# softvoyagers/claude-skills

Multi-agent workflow skills and commands for Claude Code. Orchestrate specialized agents to fix bugs, implement features, discover customer needs, and research issues — all end-to-end.

## Installation

### Claude Code

```bash
claude plugin marketplace add softvoyagers/claude-skills
```

Then enable the plugin in Claude Code settings or via the setup script from [ai-agent-setup](https://github.com/dpozimski/ai-agent-setup).

## Available Commands

| Command | Description |
|---------|-------------|
| `/softvoyagers-skills:bug-fix` | Diagnose and fix bugs using 4-phase multi-agent orchestration (diagnose, fix, validate, ship) |
| `/softvoyagers-skills:new-feature` | Implement features end-to-end with 5-phase workflow (discovery, architecture, implement, review, ship) |
| `/softvoyagers-skills:discover-feature` | Research-only customer needs analysis from 5 perspectives (virtual customer, UX designer, product owner, engineer, QA) |
| `/softvoyagers-skills:discover-feature-and-deliver-most-wanted` | Discover customer needs then implement the highest-impact feature in a single session |
| `/softvoyagers-skills:research-issue` | Research non-trivial issues with 4 parallel agents (codebase, web, docs, internal knowledge) |

## Available Skills

| Skill | Description |
|-------|-------------|
| `gamma-hello-world` | Create a minimal working Gamma presentation API example |

## How It Works

Each command follows the **Orchestrator pattern**: Claude coordinates multiple specialized agents across phases, each with focused responsibilities. Agents run in parallel where possible, with quality gates between phases.

### Common Conventions

All commands enforce:
- Test naming: `MethodName_Scenario_ExpectedResult`
- Test structure: Arrange / Act / Assert
- Minimal change surface
- Existing codebase patterns and conventions

## License

MIT
