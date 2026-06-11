---
description: End-to-end feature implementation with specialized agents
---

# /new-feature — Multi-Agent Feature Implementation

Implement a new feature end-to-end using dedicated agents. Input: a feature description (or a `SELECTED FEATURE` block from the discover-and-deliver wrapper). Output: a PR with tested, reviewed code ready for human merge.

**Feature request**: $ARGUMENTS

---

## Execution Protocol

You are the **Orchestrator**. You coordinate dedicated subagents and gate quality — you never write feature code yourself. You delegate via the Task tool's `subagent_type`; each agent carries its own model:

| Role | `subagent_type` | Model |
| ---- | --------------- | ----- |
| Codebase map | `softvoyagers-skills:sv-codebase-analyst` | sonnet |
| User advocate / acceptance criteria | `softvoyagers-skills:sv-virtual-customer` | fable |
| UX & quality risk | `softvoyagers-skills:sv-qa-analyst` | fable |
| Architecture plan | `softvoyagers-skills:sv-tech-lead` | opus |
| Implementation (sole code writer) | `softvoyagers-skills:sv-implementer` | opus |
| Tests | `softvoyagers-skills:sv-test-engineer` | opus |
| Senior review | `softvoyagers-skills:sv-code-reviewer` | opus |
| Adversarial / coverage review | `softvoyagers-skills:sv-adversarial-tester` | opus |
| Acceptance gate | `softvoyagers-skills:sv-acceptance-validator` | opus |

### Conventions (enforce in every delegation)

- Test naming `MethodName_Scenario_ExpectedResult`; Arrange / Act / Assert.
- No comments in code. Minimal change surface. Use the project's established test framework and patterns.

---

## Phase 1: DISCOVERY

**Goal**: Understand the codebase, define user-grounded acceptance criteria, surface risk.

**Pre-supplied requirements short-circuit (when called from the wrapper):** If `$ARGUMENTS` contains an authoritative `SELECTED FEATURE` block (name + customer problem + scope IN/OUT + acceptance criteria), treat those acceptance criteria as **canonical** — do **NOT** regenerate them and do **NOT** re-run discovery. Reduce Phase 1 to **`sv-codebase-analyst`** + **`sv-qa-analyst`** only (the technical grounding discovery did not produce), then go to Phase 2.

Otherwise, launch **3 agents in parallel**:
1. **`sv-codebase-analyst`** — codebase map, similar patterns to follow, affected files, dependencies, constraints.
2. **`sv-virtual-customer`** (DISCOVERY mode) — the user problem, the before/after workflow, the minimum viable scope, and **3–7 testable acceptance criteria** grounded in real user actions and outcomes (not abstract Given/When/Then), plus the done-definition.
3. **`sv-qa-analyst`** (RISK mode) — workflow breaks, confusion risks, edge cases, regression risks, data-integrity hazards.

**Gate**: Synthesize into a requirements document. Mark each acceptance criterion **must-have** or nice-to-have. Ensure the criteria reflect real user scenarios.

---

## Phase 2: ARCHITECTURE

**Goal**: A detailed plan before any code is written.

Launch **`sv-tech-lead`** (ARCHITECTURE mode): file-by-file change plan, test strategy derived from the acceptance criteria, API contracts, implementation order, risk mitigation. It reads code to match patterns and does not write code.

**Gate**: Verify the plan addresses every acceptance criterion. Revise if gaps exist.

---

## Phase 3–4: IMPLEMENT ↔ REVIEW LOOP

A single orchestrator-owned loop. **You own a monotonic iteration counter. Hard cap: 3 iterations.**

### Each iteration

1. **Produce/refine** — `sv-implementer` writes the feature following the plan (sole code writer, minimal surface, no comments); `sv-test-engineer` adds tests per the test strategy (ADD-only; never weakens passing tests).
2. **Run the full suite yourself** and capture the exit code.
3. **Review panel — launch 3 agents in parallel:**
   - `sv-code-reviewer` → `VERDICT: BLOCK|CLEAR` + fingerprinted `CRITICAL: [...]`
   - `sv-adversarial-tester` (also owns the **coverage-gap** check — the test engineer does not grade its own coverage) → `VERDICT` + `CRITICAL`
   - `sv-acceptance-validator` → `CRITERIA: [<id>: PASS|FAIL: <why>]`, one entry per criterion.

### Convergence gate (you evaluate)

**CONVERGE** when **all** hold:
- every reviewer returned `VERDICT: CLEAR`, **and**
- the full test suite exits 0, **and**
- every **must-have** acceptance criterion is `PASS`.

On convergence → Phase 5.

### No-progress guard

After each iteration: if the fingerprinted CRITICAL set does **not strictly shrink**, **OR** no `FAIL` criterion moved toward `PASS`, **OR** any previously-seen CRITICAL/FAIL fingerprint reappears, **stop early** and ship-with-residuals rather than burning budget.

### Correction payload (pass to producers each iteration)

Phase-2 architecture plan · current `git diff` · verbatim failing-test/criterion output · each CRITICAL/FAIL with `file:line` and a direction · *"Address only these; do not alter passing behavior; keep the change minimal."* Route code CRITICALs → `sv-implementer`; coverage/test CRITICALs → `sv-test-engineer`.

After 3 iterations without convergence → Phase 5 ship-with-residuals, subject to the ship-floor.

---

## Phase 5: SHIP

Execute yourself — do NOT delegate.

**Ship-floor (overrides the cap):** if any **must-have** acceptance criterion is still `FAIL` at the cap, open the PR as a **DRAFT** with a prominent `NOT READY — failing: <criteria>` header. Do not silently ship it as ready.

1. `git checkout -b feature/<descriptive-name>`
2. `git add <specific files>` — never `git add .`
3. Commit:
   ```
   git commit -m "$(cat <<'EOF'
   feat: <concise description>

   - <key change 1>
   - <key change 2>

   Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
   EOF
   )"
   ```
4. `git push -u origin feature/<branch-name>`
5. Create PR (`gh pr create --draft` if the ship-floor triggered):
   ```
   gh pr create --title "feat: <description>" --body "$(cat <<'EOF'
   ## Summary
   <2-3 sentences>

   ## Changes
   <file-level summary>

   ## Acceptance Criteria
   <each criterion with PASS/FAIL from sv-acceptance-validator>

   ## Residual Issues
   <SUGGESTION items + anything frozen by the no-progress guard, or "None">

   ## Test Plan
   - [ ] All tests pass
   - [ ] Coverage gaps checked by adversarial review
   - [ ] Acceptance criteria validated (every must-have PASS)
   - [ ] Converged through the IMPLEMENT↔REVIEW loop (≤3 iterations)

   🤖 Generated with multi-agent feature implementation
   EOF
   )"
   ```
6. Output the PR URL and summary.

---

## Error Handling

- Agent failure: retry once, then proceed without and note the gap.
- Tests/criteria failing at the cap: ship-floor applies (DRAFT PR, NOT READY header).
- Ambiguous feature request: ask the user for clarification before Phase 1.
- No obvious location for the feature: `sv-tech-lead` decides in the architecture phase with rationale.
