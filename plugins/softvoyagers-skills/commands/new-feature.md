# /new-feature — Multi-Agent Feature Implementation

Implement a new feature end-to-end using specialized agents. Input: a feature description. Output: a PR with tested, reviewed code ready for human merge.

**Feature request**: $ARGUMENTS

---

## Execution Protocol

You are the **Orchestrator**. You coordinate specialized agents across 5 phases. You never write feature code yourself — you delegate, synthesize, and gate quality.

### Conventions (enforce in all agent prompts)

- Test naming: `MethodName_Scenario_ExpectedResult`
- Test structure: Arrange / Act / Assert
- No comments in code
- Minimal change surface — only touch what's necessary
- Use the test framework already established in the project
- Follow existing codebase patterns and conventions

---

## Phase 1: DISCOVERY

**Goal**: Understand the codebase, define user-focused requirements, identify risks.

Launch **3 agents in parallel**:

### Agent 1 — Codebase Analyst (Explore)

Map the codebase structure relevant to this feature. Find similar existing features or patterns to follow. Identify files that need changes, integration points, dependencies, and technical constraints. Return: CODEBASE MAP, SIMILAR PATTERNS (with file paths), AFFECTED FILES, DEPENDENCIES, CONSTRAINTS.

### Agent 2 — User Advocate (Explore)

Think as a real end-user of this feature. Explore the codebase to understand the current user experience, then define:
1. **User problem**: What pain point does this solve? What's the user doing today without it?
2. **User workflow**: Step-by-step how a real user interacts with this feature (before/after)
3. **Minimum viable scope**: Simplest version that delivers value. What can be deferred?
4. **Acceptance criteria**: 3-7 testable requirements grounded in real user scenarios — describe actual user actions and outcomes, not abstract Given/When/Then
5. **Done definition**: What must be true for a real user to benefit?

### Agent 3 — UX Risk Assessor (Explore)

Think as a real user who might be confused or frustrated. Identify:
1. **Workflow breaks**: Could this break existing user workflows? Which ones?
2. **Confusion risks**: Inconsistent behavior, surprising defaults, unclear errors?
3. **Edge cases**: Empty states, first-time use, concurrent usage, slow connections
4. **Regression risks**: Which existing features could behave differently?
5. **Data integrity**: Could this corrupt, lose, or expose user data?

**Gate**: Synthesize into a requirements document. Ensure acceptance criteria reflect real user scenarios.

---

## Phase 2: ARCHITECTURE

**Goal**: Detailed implementation plan before any code is written.

Launch **1 agent** (needs Phase 1 output):

### Agent — Tech Lead (general-purpose)

Design the architecture based on discovery findings:
1. **File-by-file change plan**: path, changes, design decisions, dependencies
2. **Test strategy**: unit, integration, and edge case tests from user requirements
3. **API contracts**: exact signatures for new interfaces
4. **Implementation order**: dependency graph
5. **Risk mitigation**: how identified risks are addressed

Do NOT write code. Read existing code to match patterns. Reference exact file paths, function names, and line ranges.

**Gate**: Verify plan addresses all acceptance criteria. Revise if gaps exist.

---

## Phase 3: IMPLEMENT

**Goal**: Write the feature code and tests in parallel.

Launch **3 agents in parallel**:

### Agent 1 — Core Implementer (general-purpose)

Implement the feature following the architecture plan. Follow existing patterns, no comments, minimal surface. List every file created or modified.

### Agent 2 — Test Writer (general-purpose)

Write unit and integration tests per the test strategy and acceptance criteria. Cover happy path, edge cases, error scenarios, boundary conditions. Run the full test suite and report results.

### Agent 3 — Edge Case Tester (general-purpose)

Write additional tests focused on UX risks from Phase 1: workflow breaks, confusion scenarios, regression tests for existing behavior. Run tests after writing.

**Gate**: Run the full test suite yourself to verify everything compiles and passes.

---

## Phase 4: REVIEW & ITERATE

**Goal**: Two review rounds to catch issues before shipping.

### Review Round (repeat up to 2 times)

Launch **3 agents in parallel**:

#### Agent 1 — Senior Reviewer (general-purpose)

Review ALL changes via `git diff`. Check: CORRECTNESS, PATTERNS (codebase conventions), TESTS (coverage, meaningful assertions), MINIMAL SURFACE, NAMING. Flag issues as CRITICAL or SUGGESTION.

#### Agent 2 — Test Coverage Reviewer (Explore)

Review test coverage: untested code paths, edge cases, error scenarios. Verify test naming and structure conventions. Check tests are meaningful, not padding.

#### Agent 3 — Acceptance Validator (Explore)

Check each acceptance criterion from Phase 1 — PASS/FAIL with explanation. Verify edge cases handled and done definition met.

**After reviews**: Collate CRITICAL issues, run test suite. If zero critical AND tests pass, proceed to Phase 5. Otherwise launch fix agents, then re-review. After 2 rounds, proceed with issues noted in PR.

---

## Phase 5: SHIP

**Goal**: Branch, commit, push, PR.

Execute yourself — do NOT delegate:

1. `git checkout -b feature/<descriptive-name>`
2. `git add <specific files>` — never use `git add .`
3. Commit:
   ```
   git commit -m "$(cat <<'EOF'
   feat: <concise description>

   - <key change 1>
   - <key change 2>
   - <key change 3>

   Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
   EOF
   )"
   ```
4. `git push -u origin feature/<branch-name>`
5. Create PR:
   ```
   gh pr create --title "feat: <description>" --body "$(cat <<'EOF'
   ## Summary
   <2-3 sentences>

   ## Changes
   - <file-level summary>

   ## Acceptance Criteria
   <criteria from Phase 1 with PASS/FAIL>

   ## Test Plan
   - [ ] All tests pass
   - [ ] Edge cases and error scenarios tested
   - [ ] Acceptance criteria validated
   - [ ] Code reviewed through 2 automated rounds

   🤖 Generated with multi-agent feature implementation
   EOF
   )"
   ```
6. Output the PR URL and summary.

---

## Error Handling

- Agent failure: retry once, then proceed without and note the gap.
- Tests fail: fix agents in Phase 4 address this. If still failing after 2 rounds, list in PR.
- Ambiguous feature request: ask user for clarification before Phase 1.
- No obvious location for feature: Tech Lead decides in architecture phase with rationale.
