---
name: discover-feature
description: |
  Read-only, multi-perspective analysis of what customers/users actually need —
  no code, no file writes. Use when the user wants to research product gaps, pain
  points, or the highest-impact opportunities before building anything. A
  discovery panel (virtual customer, UX, product owner, QA, tech lead) loops to
  saturation and produces an Impact/Effort-ranked feature report. Trigger with
  phrases like "what should we build", "discover features", "what do users need",
  "find product gaps", "prioritize the backlog", "where are the opportunities".
allowed-tools: Task, Read, Grep, Glob, AskUserQuestion
version: 1.0.0
license: MIT
---

# discover-feature — Multi-Agent Feature Discovery

Discover what customers actually need by analyzing the product from multiple perspectives, iterating until the picture is complete. Input: a product area, user segment, or pain point from the user. Output: a prioritized, evidence-backed feature report.

**Discovery focus**: the user's request that triggered this skill (a product area, user segment, or pain point).

---

## Execution Protocol

You are the **Orchestrator**. This is a **research-only** skill — no code, no file writes, no branches. You run a discovery panel, synthesize, and let an independent critic decide whether to dig deeper, looping until findings saturate. You delegate via the Task tool's `subagent_type`; each agent carries its own model:

| Role | `subagent_type` | Model |
| ---- | --------------- | ----- |
| End-user pain points | `softvoyagers-skills:sv-virtual-customer` | fable |
| UX audit | `softvoyagers-skills:sv-ux-designer` | fable |
| Feature-gap / product debt | `softvoyagers-skills:sv-product-owner` | fable |
| Quality gaps | `softvoyagers-skills:sv-qa-analyst` | fable |
| Technical feasibility | `softvoyagers-skills:sv-tech-lead` | opus |
| Synthesis (prioritize/rank) | `softvoyagers-skills:sv-synthesis-analyst` | opus |
| Completeness critic (loop gate) | `softvoyagers-skills:sv-discovery-critic` | opus |

### Conventions (enforce in every delegation)

- **READ-ONLY**: every agent prompt must forbid file writes, edits, and branches.
- **Customer-first & evidence-based**: every recommendation traces to a real user problem with a `file:line` or supplied-source citation. Prioritize by customer value, not technical interest.

---

## RESEARCH ↔ SYNTHESIS ↔ CRITIQUE LOOP

A single orchestrator-owned loop. **You own a monotonic round counter. Hard cap: 3 rounds — stop after round 2 unless the round-1 critic flagged ≥3 grounded BLOCKING gaps** (so the common case is a broad round 1 + at most 1 targeted round; the cap is fixed at 3 and never grows past it).

### Round 1 — broad panel

Optionally enrich first: if Jira/customer-request context would help, **you** (the orchestrator, main thread) run the `atlassian:*` skills (e.g. `atlassian:search-company-knowledge`) and inject the results into `sv-product-owner`'s prompt. `sv-product-owner` does not reach external systems itself. If Jira is unavailable, proceed without it and say so.

Launch **5 agents in parallel**: `sv-virtual-customer` (DISCOVERY), `sv-ux-designer`, `sv-product-owner`, `sv-qa-analyst` (QUALITY-GAP), `sv-tech-lead` (FEASIBILITY).

### Synthesis + Critique (after each research round)

Launch **2 agents** (synthesis then critique — keep them separate so the producer never grades its own exit condition):
1. **`sv-synthesis-analyst`** — group by theme, extract the underlying need, score Impact × Frequency × Feasibility, and emit a **ranked table with explicit `Impact` and `Effort` columns**. In round N>1 it **must merge** round-1 findings from non-re-run personas with the new targeted findings (no signal dropped).
2. **`sv-discovery-critic`** — independently emit `gaps: [{persona, area, severity: BLOCKING|MINOR, why, grounded}]`.

### Convergence gate (you evaluate over the critic's schema)

**CONVERGE** when **either**:
- `round == cap`, **or**
- the critic returns **zero BLOCKING gaps**.

A gap counts as BLOCKING only if it names BOTH a specific persona to re-run AND a concrete question; otherwise downgrade it to MINOR and record it under "Known limitations" — never loop on it.

### Anti-oscillation + monotonic shrink (guarantees termination)

- A persona/area **already re-researched** in a prior round **cannot** be flagged BLOCKING again — auto-downgrade to MINOR. The BLOCKING-gap pool therefore shrinks every round.
- Round N+1 re-research is scoped **strictly** to closing the prior BLOCKING gaps. Newly discovered adjacent opportunities go to a **Backlog** section, **not** back into the loop.

On non-convergence, run round N+1 with only the personas the critic named, then re-synthesize + re-critique.

---

## REPORT

Output (read-only — no commits):

```
# Feature Discovery Report: [Area/Segment]
**Confidence**: CONVERGED | CAP-REACHED-WITH-RESIDUAL-GAPS

## Executive Summary
<3-5 sentences: key findings, top opportunities, recommended next steps>

## Top Feature Opportunities (ranked)
| # | Feature | Customer problem | Impact | Effort | Evidence |
|---|---------|------------------|--------|--------|----------|
| 1 | ...     | ...              | HIGH   | LOW    | <agents + file:line> |
...
(For each, also: proposed solution (brief), dependencies.)

## Customer Pain Points  ## Technical Landscape  ## Quality Gaps  ## Product Debt
## Known Limitations
<MINOR gaps / areas not fully explored>
## Backlog (out-of-scope adjacent opportunities surfaced during the loop)
## Recommended Next Steps
```

**Gate**: Every recommendation traces to a customer need identified by at least one agent. No feature is recommended purely for technical reasons. The report contains no code changes, branches, or PRs. The `Impact`/`Effort` table and `Confidence` field are mandatory so a wrapper can consume the result deterministically.

---

## Error Handling

- Agent failure: retry once, then proceed without and note the gap.
- Fewer than 3 substantive panel results: note the gaps and proceed with available data.
- Jira/MCP unavailable: the orchestrator notes it; `sv-product-owner` proceeds without that context.
- Ambiguous input: ask the user for clarification before round 1.
