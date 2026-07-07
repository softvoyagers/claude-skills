# softvoyagers/claude-skills

Claude Code marketplace providing a multi-agent feature-discovery workflow skill, its dedicated agent library, and utility skills.

## Repository Structure

This repository is **both a marketplace and a single plugin rooted at the repo root**. `.claude-plugin/` holds both manifests; all component directories are its siblings at the repo root.

| Path | Description |
| ---- | ----------- |
| `.claude-plugin/marketplace.json` | Marketplace catalog; its one plugin entry uses `"source": "./"` (plugin lives at repo root) |
| `.claude-plugin/plugin.json` | Plugin manifest (metadata + version) |
| `agents/` | Dedicated subagent definitions — each `.md` is one model-tiered agent (see below) |
| `skills/` | Skill definitions — each subdirectory contains a `SKILL.md` with frontmatter |
| `package.json` | Package metadata and versioning |

**Layout rule (from the Claude Code docs):** only the two `*.json` manifests go inside `.claude-plugin/`. `agents/`, `skills/`, etc. must live at the repo root as siblings of `.claude-plugin/`, or auto-discovery breaks.

The multi-agent workflow is a **skill** (not a slash command) so it **auto-triggers** from the user's request via its `description`, while still being invokable explicitly.

## Agents (shared library)

The workflow skill doesn't use the generic `Explore` / `general-purpose` agent types. Instead it delegates to **dedicated agents** defined in `agents/*.md` and referenced as `subagent_type: softvoyagers-skills:sv-<name>`. Each agent pins its own model in frontmatter — the skill stays model-agnostic.

Model tiers (most-capable-first): **Fable** for open-ended discovery personas, **Opus** for reasoning / synthesis / critique, **Sonnet** for read-only code exploration.

| Agent | Model | Purpose |
| ----- | ----- | ------- |
| `sv-virtual-customer` | fable | End-user advocate — walks real workflows, surfaces pain points |
| `sv-ux-designer` | fable | UX audit |
| `sv-product-owner` | fable | Feature-gap / product-debt analysis |
| `sv-qa-analyst` | fable | Quality-gap analysis |
| `sv-tech-lead` | opus | Feasibility assessment (no code) |
| `sv-synthesis-analyst` | opus | Discovery synthesis / prioritization (carries forward across rounds) |
| `sv-discovery-critic` | opus | Completeness critic that gates the discovery loop |

**Layout constraint (hard):** all agent files must live **flat** directly in `agents/*.md` — no subfolders. The workflow skill hardcodes the two-segment reference `softvoyagers-skills:sv-<name>`; a subfolder would inject an extra `:segment:` and silently break every reference.

## Workflow skill (multi-agent orchestration)

`discover-feature` auto-triggers from the user's request and follows the **Orchestrator pattern**: Claude coordinates dedicated agents across phases with quality gates, and never writes code itself. The skill directory name matches the `name:` in its `SKILL.md` frontmatter.

- `discover-feature/` — read-only **RESEARCH↔SYNTHESIS↔CRITIQUE loop** (≤3 rounds) → prioritized report with Impact/Effort table + `Confidence` field.

### Self-correction loop

The discovery skill iterates until it converges. The loop is **orchestrator-owned, bounded by a hard round cap, and provably terminating** via a no-progress guard. The critic emits a structured verdict so the orchestrator can evaluate the convergence predicate objectively — the producers never grade their own exit condition.

## Utility skills

- `java-decompilation/` — Decompile `.class`/jar bytecode to readable `.java` source (Vineflower + CFR), analyze, and save into a repo.
- `humanizer/` — Remove AI-writing tells from text (em dashes, rule of three, AI vocabulary, promotional language, 30+ patterns), based on Wikipedia's "Signs of AI writing". Vendored from [blader/humanizer](https://github.com/blader/humanizer) (MIT).

## Conventions

- The workflow skill takes the user's request directly as input (no `$ARGUMENTS` substitution — that's a slash-command-only feature).
- Delegate to dedicated agents via `subagent_type: softvoyagers-skills:sv-<name>` — never the generic `Explore` / `general-purpose` types.
- Keep the discovery skill tech-stack agnostic — it auto-detects project tooling and reads the codebase without modifying it.
- Jira/MCP lookups are performed by the **orchestrator** (via `atlassian:*` skills) and injected into agent prompts — plugin subagents cannot reach Atlassian query tools.

## Modifying Skills or Agents

- Agents must have frontmatter with `name`, `description`, `model`, and `tools`; keep them flat in `agents/` (see layout constraint).
- Skills must have a `SKILL.md` with YAML frontmatter (`name`, `description` required); the directory name must match the `name`. The `description` is the **auto-trigger surface** — make it state *when* to use the skill and include natural trigger phrases.
- Bump the version in **three** places on release: `package.json`, `.claude-plugin/plugin.json`, and `.claude-plugin/marketplace.json` (`plugins[0].version`).
- **Verify plugin discovery after changes**: install the plugin and confirm both `agents/*.md` auto-discover (so `softvoyagers-skills:sv-<name>` resolves as a `subagent_type`) and `skills/*/SKILL.md` auto-discover (so each skill appears and can auto-trigger).
