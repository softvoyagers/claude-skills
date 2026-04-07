# softvoyagers/claude-skills

Multi-agent workflow commands and skills for Claude Code. Orchestrate specialized agents to fix bugs, implement features, discover customer needs, and research issues — all end-to-end.

## Installation

```bash
claude plugin marketplace add softvoyagers/claude-skills
claude plugin install softvoyagers-skills
```

## Commands

| Command | Phases | Description |
|---------|--------|-------------|
| `/softvoyagers-skills:bug-fix` | Diagnose → Fix → Validate → Ship | Multi-agent bug diagnosis and minimal fix with regression tests |
| `/softvoyagers-skills:new-feature` | Discovery → Architecture → Implement → Review → Ship | End-to-end feature implementation with customer-focused requirements |
| `/softvoyagers-skills:discover-feature` | Research → Synthesis → Report | Read-only customer needs analysis from 5 perspectives (Virtual Customer, UX Designer, Product Owner, Engineer, QA) |
| `/softvoyagers-skills:discover-feature-and-deliver-most-wanted` | Research → Selection → Architecture → Implement → Review → Ship | Discover customer needs then build the highest-impact feature |
| `/softvoyagers-skills:research-issue` | Input Parsing → Gather → Analyze → Report | Research non-trivial issues with 4 parallel agents (codebase, web, docs, internal knowledge) |

## Skills

| Skill | Description |
|-------|-------------|
| `use-gamma-slides` | Generate slide presentations using the Gamma API — prompt-driven creation, style customization, and structured content |

## How It Works

Each command uses the **Orchestrator pattern**:

1. Claude acts as the orchestrator and never writes code directly
2. Specialized agents are launched in parallel where possible (Explore for read-only research, general-purpose for code changes)
3. Quality gates between phases ensure correctness before proceeding
4. Final phase creates a branch, commits, pushes, and opens a PR

### Conventions

All commands enforce:
- Test naming: `MethodName_Scenario_ExpectedResult`
- Test structure: Arrange / Act / Assert
- Minimal change surface — only touch what's necessary
- Follow existing codebase patterns and conventions
- Tech-stack agnostic — auto-detects project tooling

## License

MIT
