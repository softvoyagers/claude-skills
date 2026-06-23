---
name: discover-feature-and-deliver-most-wanted
description: |
  Discover customer needs then build the single highest-impact feature, shipped as
  a PR. Use when the user wants to BOTH research what to build AND build it in one
  go. Thin wrapper: runs the discover-feature research flow, gates/auto-selects the
  #1 opportunity, then runs the new-feature implementation flow. Trigger with
  phrases like "discover and build the most-wanted feature", "find and deliver the
  best feature", "research then build what users want most", "figure out what to
  build and ship it".
allowed-tools: Task, Bash, Read, Write, Edit, Grep, Glob, AskUserQuestion
version: 1.0.0
license: MIT
---

# discover-feature-and-deliver-most-wanted — Discover & Deliver

Discover what customers need, then build the single highest-impact feature. Input: a product area, user segment, or pain point from the user. Output: a PR with the most-wanted feature, grounded in customer research.

**Discovery focus**: the user's request that triggered this skill (a product area, user segment, or pain point).

---

## Execution Protocol

You are the **Orchestrator**. This skill is a **thin wrapper**: it runs two existing workflows back to back with a decision gate between them. It defines **no agents or phases of its own** — the `discover-feature` and `new-feature` skills own all the work (including their own self-correcting loops). To keep the gate and loop control in the orchestrator's hands, **inline each child skill's protocol at runtime**, in order (rather than invoking them as separate skills).

---

## STEP A — Discover (read-only)

Run the **`discover-feature`** skill's protocol on the discovery focus to full convergence. Its report yields a ranked table with **`Impact`** and **`Effort`** columns and a **`Confidence`** field (`CONVERGED` or `CAP-REACHED-WITH-RESIDUAL-GAPS`).

Read-only applies to this step only.

---

## STEP B — Gate & auto-select

Inspect the ranked #1 feature.

- **Auto-proceed to STEP C** only if **all** hold: the #1 feature is **HIGH impact** AND **LOW/MEDIUM effort** AND discovery `Confidence == CONVERGED`.
- **Otherwise present the top 3 and ASK THE USER** which to build. Using discover-feature's tier scales (Impact `HIGH`/`MED`/`LOW`, Effort `LOW`/`MED`/`HIGH`), "no clear winner" is concretely: two or more features sit within one impact tier **and** one effort tier of each other at the top of the ranking, **or** `Confidence == CAP-REACHED-WITH-RESIDUAL-GAPS`.
- **Escape hatch**: if **no** feature is HIGH impact, do **not** build — emit the discovery report (as the `discover-feature` skill would) and stop.

**Build the `SELECTED FEATURE` block (thin transform, not a new phase).** The `discover-feature` skill does not itself emit scope or acceptance criteria, so derive them here at the gate from the report's customer-problem / proposed-solution / impact / effort fields:

```
SELECTED FEATURE: <name>
CUSTOMER PROBLEM: <what users struggle with — verbatim from discovery>
SCOPE: IN: <minimum viable> | OUT: <deferred>
ACCEPTANCE CRITERIA: <3-7 testable criteria grounded in the customer problem; mark must-have>
```

This is orchestrator prose — do **not** reintroduce a standalone feature-selection phase body.

---

## STEP C — Deliver

Run the **`new-feature`** skill's protocol, passing the `SELECTED FEATURE` block verbatim as its input. Those acceptance criteria are **canonical**: new-feature must **short-circuit its Phase 1** to `sv-codebase-analyst` + `sv-qa-analyst` only (it does **not** re-run discovery or regenerate the criteria), then run its architecture → IMPLEMENT↔REVIEW loop → ship as normal.

This step writes code and opens the PR (commit trailer `Co-Authored-By: Claude Opus 4.8 (1M context)`). The PR body must add a **Discovery Evidence** section (which agents identified the need, key findings) and a **What's NOT included** section (the OUT scope).

The wrapper itself has no loop; both child workflows self-cap their own loops.

---

## Error Handling

- STEP A returns no HIGH-impact feature → escape hatch (emit report, do not build).
- Ambiguous gate (no clear winner / residual-gap confidence) → present top 3 and ask the user.
- Anything inside a child workflow → handled by that workflow's own error handling.
