---
description: Multi-agent bug diagnosis and minimal fix with regression tests
---

# /bug-fix — Multi-Agent Bug Fix

Diagnose and fix a bug end-to-end using specialized agents. Input: a bug report or error description. Output: a PR with a minimal, tested fix ready for human merge.

**Bug report**: $ARGUMENTS

---

## Execution Protocol

You are the **Orchestrator**. You coordinate specialized agents across 4 phases. You never write fix code yourself — you delegate, synthesize, and gate quality. The fix must be **minimal** — change the fewest lines possible to resolve the root cause. No refactoring, no scope creep.

### Conventions (enforce in all agent prompts)

- Test naming: `MethodName_Scenario_ExpectedResult`
- Test structure: Arrange / Act / Assert
- No comments in code
- Minimal change surface — fix the bug, nothing else
- No refactoring — even if surrounding code is ugly
- Use the test framework already established in the project

---

## Phase 1: DIAGNOSE

**Goal**: Find the root cause, write a failing reproduction test, check for sibling bugs.

Launch **3 agents in parallel**:

### Agent 1 — Root Cause Analyst (Explore)

Trace the code path that triggers the bug. Use Grep, Read, git log, and git blame to identify the exact root cause (file, line, what's wrong). Map symptoms to code path to root cause to blast radius. Return: SYMPTOMS, CODE PATH, ROOT CAUSE (file:line + explanation), RECENT CHANGES, BLAST RADIUS.

### Agent 2 — Reproduction Test Writer (general-purpose)

Write a test that REPRODUCES the bug — it should FAIL with current code. Read existing tests first to match conventions. Run the test to confirm failure. Return: TEST FILE, TEST NAME, FAILURE OUTPUT, EXPECTED BEHAVIOR.

### Agent 3 — Sibling Bug Hunter (Explore)

Once you understand the bug pattern, search the entire codebase for the same defect pattern in other locations. Return: BUG PATTERN, SIBLING LOCATIONS (file:line list), SEVERITY per location, RECOMMENDATION (same PR or separate).

**Gate**: Synthesize diagnosis. If reproduction test failed or diagnosis is unclear, ask the user for more context.

---

## Phase 2: FIX

**Goal**: Apply a minimal fix and write regression tests.

Launch **2 agents in parallel**:

### Agent 1 — Fixer (general-purpose)

Apply a minimal fix for the root cause. Provide the full Phase 1 diagnosis. Rules: fix ROOT CAUSE not symptoms, change FEWEST LINES possible, no refactoring, no improvements. Fix sibling locations only if flagged for same-PR. List every file:line changed with justification.

### Agent 2 — Test Writer (general-purpose)

Write regression tests for the fix. Verify the Phase 1 reproduction test, write boundary tests around the fix. Run the full test suite — all tests should pass. Report: test count, test names, runner results.

**Gate**: Run the full test suite yourself to verify.

---

## Phase 3: VALIDATE

**Goal**: Two review rounds to verify correctness and minimality.

### Review Round (repeat up to 2 times)

Launch **3 agents in parallel**:

#### Agent 1 — Senior Reviewer (general-purpose)

Review ALL changes via `git diff`. Check: CORRECTNESS (root cause addressed?), MINIMALITY (unnecessary changes?), REGRESSION RISK, TEST QUALITY. Flag issues as CRITICAL or SUGGESTION. A bug fix PR should have a tiny diff.

#### Agent 2 — User Verification (Explore)

Trace the code path from bug trigger through fix. Verify the exact scenario from the bug report is resolved. Check for uncovered edge cases. Return: FIXED yes/no with explanation.

#### Agent 3 — Adversarial Tester (Explore)

Try to break the fix: slightly different inputs, null/empty/boundary values, regression in related functionality, partial fixes. Return: attack scenarios and results.

**After reviews**: Collate CRITICAL issues, run test suite. If zero critical issues AND tests pass, proceed to Phase 4. Otherwise launch a fix agent, then re-review. After 2 rounds, proceed with remaining issues noted.

---

## Phase 4: SHIP

**Goal**: Create a branch, commit, push, and open a PR.

Execute yourself — do NOT delegate:

1. `git checkout -b fix/<descriptive-name>`
2. `git add <specific files>` — never use `git add .`
3. Commit:
   ```
   git commit -m "$(cat <<'EOF'
   fix: <concise description>

   Root cause: <one-line explanation>

   - <fix summary>
   - <tests added>

   Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
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

   ## Test Plan
   - [ ] Reproduction test confirms fix
   - [ ] Regression tests pass
   - [ ] Code reviewed through 2 automated rounds
   - [ ] Minimal change surface

   🤖 Generated with multi-agent bug fix
   EOF
   )"
   ```
6. Output the PR URL and summary.

---

## Error Handling

- Agent failure: retry once, then proceed without and note the gap.
- Tests fail after fix: fix agent in Phase 3 addresses this. If still failing after 2 rounds, list in PR.
- Ambiguous bug report: ask user before proceeding past Phase 1.
- Cross-repo root cause: fix what's in THIS repo, note cross-repo issues in PR.
- Sibling bugs: include in same PR only if fix is identical and low-risk, otherwise note in PR.
