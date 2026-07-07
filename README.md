# softvoyagers/claude-skills

Skills for Claude Code. A multi-agent **feature-discovery** workflow that orchestrates **dedicated, model-tiered agents** to surface customer needs — with a self-correcting research loop — plus writing and Java-decompilation utility skills. The workflow is a **skill**, so it **auto-triggers** from what you ask for (and can still be invoked explicitly).

## Installation

```bash
claude plugin marketplace add softvoyagers/claude-skills
claude plugin install softvoyagers-skills
```

## Workflow skill

Auto-triggers from your request — e.g. "what should we build next?" pulls in `discover-feature`. You can still invoke it explicitly as `/softvoyagers-skills:discover-feature`.

| Skill | Flow | Description |
|-------|------|-------------|
| `discover-feature` | **RESEARCH↔CRITIQUE loop** → Report | Read-only customer-needs analysis from multiple perspectives, looping until findings saturate; emits an Impact/Effort-ranked report |

## Utility skills

| Skill | Description |
|-------|-------------|
| `java-decompilation` | Decompile compiled Java (`.class` files / jars) to readable `.java` source with Vineflower (CFR cross-check), then analyze and save it into a repo |
| `humanizer` | Remove signs of AI-generated writing from text — em dashes, rule of three, AI vocabulary, promotional language, and 30+ other patterns — based on Wikipedia's "Signs of AI writing" guide |

## How It Works

The `discover-feature` skill uses the **Orchestrator pattern**:

1. Claude acts as the orchestrator and never writes code directly.
2. It delegates to **dedicated agents** (in the plugin's `agents/` directory), each pinned to the most capable model for its job — **Fable** for discovery personas, **Opus** for reasoning / synthesis / critique, **Sonnet** for read-only exploration.
3. **Quality gates** between phases ensure completeness before proceeding.
4. The skill runs a **bounded self-correction loop** — research → synthesize → critique → re-research — that iterates until findings saturate and provably terminates at a hard round cap.

### The agent library

A set of read-only `sv-*` agents (`sv-virtual-customer`, `sv-ux-designer`, `sv-product-owner`, `sv-qa-analyst`, `sv-tech-lead`, `sv-synthesis-analyst`, `sv-discovery-critic`) drives the discovery workflow, referenced as `subagent_type: softvoyagers-skills:sv-<name>`. Producers and the completeness critic are kept separate so no agent grades its own work.

### Conventions

The discovery skill is tech-stack agnostic — it auto-detects project tooling and reads the codebase without modifying it.

## License

MIT
