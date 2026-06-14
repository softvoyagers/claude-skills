---
description: Multi-agent bug diagnosis and minimal fix with regression tests
---

# /bug-fix — Multi-Agent Bug Fix

Diagnose and fix a bug end-to-end using dedicated agents. Input: a bug report or error description. Output: a PR with a minimal, tested fix ready for human merge.

**Bug report**: $ARGUMENTS

---

## Execution Protocol

You are the **Orchestrator**. You coordinate dedicated subagents and gate quality — you never write fix code yourself. The fix must be **minimal**: change the fewest lines needed to resolve the root cause. No refactoring, no scope creep.

You delegate by calling the Task tool with the agent's `subagent_type`. The agents are provided by this plugin and carry their own model — you do not set models:

| Role | `subagent_type` | Model |
| ---- | --------------- | ----- |
| Bug diagnosis (root cause, blast radius, git history, sibling bugs) | `softvoyagers-skills:sv-root-cause-analyst` | opus |
| Tests (reproduction + regression) | `softvoyagers-skills:sv-test-engineer` | opus |
| Fix author (sole code writer) | `softvoyagers-skills:sv-implementer` | opus |
| Senior review | `softvoyagers-skills:sv-code-reviewer` | opus |
| Adversarial / coverage review | `softvoyagers-skills:sv-adversarial-tester` | opus |
| User verification | `softvoyagers-skills:sv-virtual-customer` | fable |

### Conventions (enforce in every delegation)

- Test naming `MethodName_Scenario_ExpectedResult`; Arrange / Act / Assert structure.
- No comments in code. Use the test framework already established in the project.
- Minimal change surface — fix the bug, nothing else. No refactoring even if surrounding code is ugly.

---

## Phase 1: DIAGNOSE

**Goal**: Find the root cause and lock in a failing reproduction test.

Launch **2 agents in parallel**:

1. **`sv-root-cause-analyst`** — trace symptom → code path → root cause (`file:line` + why), assess blast radius, mine `git log`/`git blame` for the regressing change and any prior reverts, and **sequentially** hunt the same defect pattern elsewhere. Returns the full diagnosis.
2. **`sv-test-engineer`** — write a test that **reproduces** the bug and **fails** against current code; run it; confirm it is red. Returns the test file path, test name, and failure output.

**Freeze the contract.** Record the reproduction test's file path + test name. This test is **frozen** — no agent may modify, skip, or weaken it for the rest of the run.

**Gate**: If the reproduction test does not fail, or the diagnosis is unclear/multi-rooted with no leading hypothesis, ask the user for more context before looping.

---

## Phase 2–3: FIX ↔ VALIDATE LOOP

A single orchestrator-owned loop. **You own a monotonic iteration counter; no agent may extend it. Hard cap: 2 iterations.**

### Each iteration

1. **Correct.** Route the work:
   - code CRITICALs → **`sv-implementer`** (the sole code writer): apply/refine the minimal fix.
   - coverage / reproduction CRITICALs → **`sv-test-engineer`** (ADD-only): add regression/boundary tests. It may not weaken the frozen reproduction test or any previously passing test.
   - On iteration 1, the implementer fixes the root cause from the Phase 1 diagnosis; the test engineer adds boundary tests around the fix.
2. **Run the full suite yourself** and capture the exit code. Diff the test files vs. the previous iteration and **reject any assertion/test removal** — if a passing test or the frozen reproduction test was weakened, send it back.
3. **Review panel — launch 3 agents in parallel:** `sv-code-reviewer`, `sv-adversarial-tester`, `sv-virtual-customer` (VERIFICATION mode). Each ends with `VERDICT: BLOCK|CLEAR` and a fingerprinted `CRITICAL: [...]` list. `SUGGESTION:` lines never block.

### Convergence gate (you evaluate)

**CONVERGE** when **all three** hold:
- every review agent returned `VERDICT: CLEAR` (the union of CRITICAL sets is empty), **and**
- the full test suite exits 0, **and**
- the **frozen** reproduction test passes.

On convergence → Phase 4.

### No-progress guard (prevents thrashing / burns)

After each iteration, fingerprint the CRITICAL set (`file:line | category`). If it does **not strictly shrink** versus the previous iteration — count not reduced, or a previously-seen CRITICAL reappears — **stop early** and ship-with-residuals (do not spend the remaining budget). A reappearing fingerprint means freeze that change and escalate it to the PR body.

### Correction payload (pass to the corrector each iteration)

Full Phase-1 diagnosis · current `git diff` · verbatim failing-test output · each CRITICAL with `file:line` and a suggested direction · the instruction: *"Address ONLY these CRITICALs. Do not refactor. Keep the change minimal. If two CRITICALs conflict, surface the conflict instead of thrashing."*

After 2 iterations without convergence → proceed to Phase 4 ship-with-residuals (issues noted in the PR), **unless** the ship-floor below triggers.

---

## Phase 4: SHIP

Execute yourself — do NOT delegate.

**Ship-floor (overrides the cap):** if the **frozen reproduction test still FAILS** at the cap, do **NOT** open a PR. Push a branch labeled `UNRESOLVED`, write the diagnosis + best partial fix into its description, and hand back to the user. The bug is not fixed; do not pretend otherwise.

Otherwise:

1. `git checkout -b fix/<descriptive-name>`
2. `git add <specific files>` — never `git add .`
3. Commit:
   ```
   git commit -m "$(cat <<'EOF'
   fix: <concise description>

   Root cause: <one-line explanation>

   - <fix summary>
   - <tests added>

   Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
   EOF
   )"
   ```
4. `git push -u origin fix/<branch-name>`
5. Create PR:
   ```
   gh pr create --title "fix: <description>" --body "$(cat <<'EOF'
   ## Bug Report
   <original bug report>

   ## Root Cause
   <root cause with file:line>

   ## Fix
   <what changed and why>

   ## Sibling Bugs
   <list or "None found">

   ## Residual Issues
   <SUGGESTION items + any CRITICAL frozen by the no-progress guard, or "None">

   ## Test Plan
   - [ ] Frozen reproduction test now passes
   - [ ] Regression tests pass
   - [ ] Converged through the FIX↔VALIDATE loop (≤2 iterations)
   - [ ] Minimal change surface

   🤖 Generated with multi-agent bug fix
   EOF
   )"
   ```
6. Output the PR URL and a summary.

---

## Error Handling

- Agent failure: retry once, then proceed without and note the gap.
- Tests still failing at the cap: ship-floor applies (UNRESOLVED branch, no PR).
- Ambiguous bug report: ask the user before looping.
- Cross-repo root cause: fix what's in THIS repo; note cross-repo issues in the PR.
- Sibling bugs: include in the same PR only if the fix is identical and low-risk; otherwise note them in the PR.
