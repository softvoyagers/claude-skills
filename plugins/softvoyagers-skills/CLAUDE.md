# softvoyagers/claude-skills

Claude Code marketplace providing multi-agent workflow commands, a shared library of dedicated agents, and skills.

## Repository Structure

| Path | Description |
| ---- | ----------- |
| `.claude-plugin/` | Marketplace and plugin configuration for Claude Code |
| `agents/` | Dedicated subagent definitions — each `.md` is one model-tiered agent (see below) |
| `commands/` | Slash command definitions — each `.md` is a multi-agent orchestration workflow |
| `skills/` | Skill definitions — each subdirectory contains a `SKILL.md` with frontmatter |
| `package.json` | Package metadata and versioning |

## Agents (shared library)

Commands no longer use the generic `Explore` / `general-purpose` agent types. Instead they delegate to **dedicated agents** defined in `agents/*.md` and referenced as `subagent_type: softvoyagers-skills:sv-<name>`. Each agent pins its own model in frontmatter — commands stay model-agnostic.

Model tiers (most-capable-first): **Fable** for open-ended discovery personas, **Opus** for reasoning / architecture / implementation / review / synthesis, **Sonnet** for read-only code exploration.

| Agent | Model | Purpose |
| ----- | ----- | ------- |
| `sv-codebase-analyst` | sonnet | Read-only codebase map / affected files / patterns |
| `sv-root-cause-analyst` | opus | Bug diagnosis: root cause, blast radius, git history, sibling bugs |
| `sv-virtual-customer` | fable | End-user advocate (discovery) + user verification (review) |
| `sv-ux-designer` | fable | UX audit |
| `sv-product-owner` | fable | Feature-gap / product-debt analysis |
| `sv-qa-analyst` | fable | Quality gaps (discovery) + UX/quality risk (planning) |
| `sv-tech-lead` | opus | Architecture plan + feasibility assessment (no code) |
| `sv-implementer` | opus | Sole code writer (features & fixes), minimal surface |
| `sv-test-engineer` | opus | Reproduction + regression + feature tests (ADD-only in loops) |
| `sv-code-reviewer` | opus | Senior review → `VERDICT` + `CRITICAL` |
| `sv-adversarial-tester` | opus | Break-it + coverage-gap review → `VERDICT` + `CRITICAL` |
| `sv-acceptance-validator` | opus | Per-criterion `PASS`/`FAIL` gate |
| `sv-synthesis-analyst` | opus | Discovery synthesis / prioritization (carries forward across rounds) |
| `sv-discovery-critic` | opus | Completeness critic that gates the discovery loop |

**Layout constraint (hard):** all agent files must live **flat** directly in `agents/*.md` — no subfolders. The commands hardcode the two-segment reference `softvoyagers-skills:sv-<name>`; a subfolder would inject an extra `:segment:` and silently break every reference.

## Commands

All commands follow the **Orchestrator pattern**: Claude coordinates dedicated agents across phases with quality gates, and never writes code itself.

- `bug-fix.md` — Diagnose → **FIX↔VALIDATE self-correction loop** (≤3 iters) → Ship. Freezes a reproduction test as the contract; ship-floor pushes an `UNRESOLVED` branch instead of a PR if it still fails.
- `new-feature.md` — Discovery → Architecture → **IMPLEMENT↔REVIEW loop** (≤3 iters, acceptance-gated) → Ship. Short-circuits discovery when given a `SELECTED FEATURE` block from the wrapper.
- `discover-feature.md` — read-only **RESEARCH↔SYNTHESIS↔CRITIQUE loop** (≤2 rounds) → prioritized report with Impact/Effort table + `Confidence` field.
- `discover-feature-and-deliver-most-wanted.md` — **thin wrapper**: runs `discover-feature` → gate/auto-select #1 → runs `new-feature`. No phases of its own.

### Self-correction loops

Three commands iterate until they converge. Every loop is **orchestrator-owned, bounded by a hard iteration cap, and provably terminating** via a no-progress / monotonic-shrink guard. Review agents emit structured `VERDICT: BLOCK|CLEAR` + fingerprinted `CRITICAL: [...]` (or `CRITERIA: [...]`) so the orchestrator can evaluate the convergence predicate objectively. Producers never grade their own exit condition.

## Skills

- `use-gamma-slides/` — Gamma API slide deck generation.

## Conventions

- Commands use `$ARGUMENTS` for user input.
- Delegate to dedicated agents via `subagent_type: softvoyagers-skills:sv-<name>` — never the generic `Explore` / `general-purpose` types.
- Test naming: `MethodName_Scenario_ExpectedResult`; structure: Arrange / Act / Assert.
- No comments in code. Minimal change surface — only touch what's necessary.
- Jira/MCP lookups are performed by the **orchestrator** (via `atlassian:*` skills) and injected into agent prompts — plugin subagents cannot reach Atlassian query tools.

## Modifying Commands, Agents, or Skills

- Keep commands tech-stack agnostic — they auto-detect project tooling.
- Agents must have frontmatter with `name`, `description`, `model`, and `tools`; keep them flat in `agents/` (see layout constraint).
- Skills must have a `SKILL.md` with YAML frontmatter (`name`, `description` required); the directory name must match the `name`.
- Bump the version in **three** places on release: `package.json`, `.claude-plugin/plugin.json`, and `.claude-plugin/marketplace.json` (`plugins[0].version`).
- **Verify plugin-agent discovery after changes**: install the plugin and confirm `agents/*.md` auto-discover and that `softvoyagers-skills:sv-<name>` resolves as a `subagent_type` (this repo previously needed explicit fixes for command discovery).
