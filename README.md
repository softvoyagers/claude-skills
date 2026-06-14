# softvoyagers/claude-skills

Multi-agent workflow commands and skills for Claude Code. Orchestrate **dedicated, model-tiered agents** to fix bugs, implement features, and discover customer needs — end-to-end, with self-correcting loops.

## Installation

```bash
claude plugin marketplace add softvoyagers/claude-skills
claude plugin install softvoyagers-skills
```

## Commands

| Command | Flow | Description |
|---------|------|-------------|
| `/softvoyagers-skills:bug-fix` | Diagnose → **FIX↔VALIDATE loop** → Ship | Diagnose a bug, freeze a reproduction test, and iterate a minimal fix until reviews + tests converge |
| `/softvoyagers-skills:new-feature` | Discovery → Architecture → **IMPLEMENT↔REVIEW loop** → Ship | Build a feature with user-grounded acceptance criteria, iterating until every must-have criterion passes |
| `/softvoyagers-skills:discover-feature` | **RESEARCH↔CRITIQUE loop** → Report | Read-only customer-needs analysis from 5 perspectives, looping until findings saturate; emits an Impact/Effort-ranked report |
| `/softvoyagers-skills:discover-feature-and-deliver-most-wanted` | discover-feature → gate → new-feature | Thin wrapper: discover needs, auto-select the highest-impact feature, then build it |

## Skills

| Skill | Description |
|-------|-------------|
| `java-decompilation` | Decompile compiled Java (`.class` files / jars) to readable `.java` source with Vineflower (CFR cross-check), then analyze and save it into a repo |

## How It Works

Each command uses the **Orchestrator pattern**:

1. Claude acts as the orchestrator and never writes code directly.
2. It delegates to **dedicated agents** (in the plugin's `agents/` directory), each pinned to the most capable model for its job — **Fable** for discovery personas, **Opus** for reasoning / code / review, **Sonnet** for read-only exploration.
3. **Quality gates** between phases ensure correctness before proceeding.
4. Three commands run **bounded self-correction loops** — produce → review → correct → re-review — that iterate until they converge (zero blocking issues, tests pass, acceptance criteria met) and provably terminate at a hard iteration cap.
5. The final phase creates a branch, commits, pushes, and opens a PR.

### The agent library

A shared set of `sv-*` agents (e.g. `sv-root-cause-analyst`, `sv-tech-lead`, `sv-implementer`, `sv-test-engineer`, `sv-code-reviewer`, `sv-acceptance-validator`, `sv-virtual-customer`, `sv-synthesis-analyst`, `sv-discovery-critic`) is reused across commands, referenced as `subagent_type: softvoyagers-skills:sv-<name>`. Producers and reviewers are kept separate so no agent grades its own work.

### Conventions

All commands enforce:
- Test naming `MethodName_Scenario_ExpectedResult`; Arrange / Act / Assert structure.
- Minimal change surface — only touch what's necessary.
- Follow existing codebase patterns; tech-stack agnostic — auto-detects project tooling.

## License

MIT
