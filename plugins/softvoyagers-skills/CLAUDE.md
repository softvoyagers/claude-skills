# softvoyagers/claude-skills

Claude Code marketplace providing multi-agent workflow skills, a shared library of dedicated agents, and utility skills.

## Repository Structure

| Path | Description |
| ---- | ----------- |
| `.claude-plugin/` | Marketplace and plugin configuration for Claude Code |
| `agents/` | Dedicated subagent definitions — each `.md` is one model-tiered agent (see below) |
| `skills/` | Skill definitions — each subdirectory contains a `SKILL.md` with frontmatter |
| `package.json` | Package metadata and versioning |

The multi-agent workflows are **skills** (not slash commands) so they **auto-trigger** from the user's request via their `description`, while still being invokable explicitly. The orchestration logic is unchanged — only the entry point moved from `commands/*.md` to `skills/<name>/SKILL.md`.

## Agents (shared library)

The workflow skills don't use the generic `Explore` / `general-purpose` agent types. Instead they delegate to **dedicated agents** defined in `agents/*.md` and referenced as `subagent_type: softvoyagers-skills:sv-<name>`. Each agent pins its own model in frontmatter — the skills stay model-agnostic.

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

**Layout constraint (hard):** all agent files must live **flat** directly in `agents/*.md` — no subfolders. The workflow skills hardcode the two-segment reference `softvoyagers-skills:sv-<name>`; a subfolder would inject an extra `:segment:` and silently break every reference.

## Workflow skills (multi-agent orchestration)

These four skills auto-trigger from the user's request and follow the **Orchestrator pattern**: Claude coordinates dedicated agents across phases with quality gates, and never writes code itself. The skill directory name matches the `name:` in its `SKILL.md` frontmatter.

- `bug-fix/` — Diagnose → **FIX↔VALIDATE self-correction loop** (≤2 iters) → Ship. Freezes a reproduction test as the contract; ship-floor pushes an `UNRESOLVED` branch instead of a PR if it still fails.
- `new-feature/` — Discovery → Architecture → **IMPLEMENT↔REVIEW loop** (≤2 iters, acceptance-gated) → Ship. Short-circuits discovery when given a `SELECTED FEATURE` block from the wrapper.
- `discover-feature/` — read-only **RESEARCH↔SYNTHESIS↔CRITIQUE loop** (≤3 rounds) → prioritized report with Impact/Effort table + `Confidence` field.
- `discover-feature-and-deliver-most-wanted/` — **thin wrapper**: inlines `discover-feature` → gate/auto-select #1 → inlines `new-feature`. No phases of its own.

### Self-correction loops

Three workflow skills iterate until they converge. Every loop is **orchestrator-owned, bounded by a hard iteration cap, and provably terminating** via a no-progress / monotonic-shrink guard. Review agents emit structured `VERDICT: BLOCK|CLEAR` + fingerprinted `CRITICAL: [...]` (or `CRITERIA: [...]`) so the orchestrator can evaluate the convergence predicate objectively. Producers never grade their own exit condition.

## Utility skills

- `java-decompilation/` — Decompile `.class`/jar bytecode to readable `.java` source (Vineflower + CFR), analyze, and save into a repo.
- `humanizer/` — Remove AI-writing tells from text (em dashes, rule of three, AI vocabulary, promotional language, 30+ patterns), based on Wikipedia's "Signs of AI writing". Vendored from [blader/humanizer](https://github.com/blader/humanizer) (MIT).

## Conventions

- Workflow skills take the user's request directly as input (no `$ARGUMENTS` substitution — that's a slash-command-only feature); the wrapper passes a `SELECTED FEATURE` block to `new-feature`.
- Delegate to dedicated agents via `subagent_type: softvoyagers-skills:sv-<name>` — never the generic `Explore` / `general-purpose` types.
- Test naming: `MethodName_Scenario_ExpectedResult`; structure: Arrange / Act / Assert.
- No comments in code. Minimal change surface — only touch what's necessary.
- Jira/MCP lookups are performed by the **orchestrator** (via `atlassian:*` skills) and injected into agent prompts — plugin subagents cannot reach Atlassian query tools.

## Modifying Skills or Agents

- Keep workflow skills tech-stack agnostic — they auto-detect project tooling.
- Agents must have frontmatter with `name`, `description`, `model`, and `tools`; keep them flat in `agents/` (see layout constraint).
- Skills must have a `SKILL.md` with YAML frontmatter (`name`, `description` required); the directory name must match the `name`. The `description` is the **auto-trigger surface** — make it state *when* to use the skill and include natural trigger phrases.
- Bump the version in **three** places on release: `package.json`, `.claude-plugin/plugin.json`, and `.claude-plugin/marketplace.json` (`plugins[0].version`).
- **Verify plugin discovery after changes**: install the plugin and confirm both `agents/*.md` auto-discover (so `softvoyagers-skills:sv-<name>` resolves as a `subagent_type`) and `skills/*/SKILL.md` auto-discover (so each skill appears and can auto-trigger).
