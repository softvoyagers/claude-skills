# /discover-feature-and-deliver-most-wanted — Discover & Deliver

Discover what customers need, then implement the highest-impact feature. Input: a product area, user segment, or pain point. Output: a PR with the most wanted feature, grounded in customer research.

**Discovery focus**: $ARGUMENTS

---

## Execution Protocol

You are the **Orchestrator**. You coordinate specialized agents across 6 phases: discovery (Phases 1-2), then implementation (Phases 3-6). The discovery team identifies the most impactful feature, then the engineering team builds it.

### Conventions (enforce in all agent prompts)

- Test naming: `MethodName_Scenario_ExpectedResult`
- Test structure: Arrange / Act / Assert
- No comments in code
- Minimal change surface — only touch what's necessary
- Follow existing codebase patterns and conventions
- Every feature must trace back to a customer need from discovery

---

## Phase 1: CUSTOMER RESEARCH

**Goal**: Understand customer needs from multiple perspectives.

Launch **5 agents in parallel**:

### Agent 1 — Virtual Customer (Explore)

You are a real end-user. Explore the codebase to understand the current experience in: [paste $ARGUMENTS]

Walk through user workflows — what's painful, slow, confusing? Find friction points, hidden features, cryptic errors, dead ends. Return: TOP PAIN POINTS (ranked), WORKFLOW GAPS, USER FRICTION MAP.

### Agent 2 — UX Designer (Explore)

You are a UX designer auditing the product. Explore the area of: [paste $ARGUMENTS]

Analyze UI/UX patterns, consistency, navigation flow. Find missing states (empty, loading, error, first-time). Evaluate feedback loops. Return: UX AUDIT (ranked by impact), MISSING STATES, DESIGN DEBT.

### Agent 3 — Product Owner (general-purpose)

You are a product owner evaluating opportunities in: [paste $ARGUMENTS]

Map current features vs. gaps. Search for TODO/FIXME/HACK. Check git log for product direction. Search Jira (via ToolSearch) for feature requests and complaints. Return: FEATURE GAP ANALYSIS, PRODUCT DEBT, CUSTOMER REQUEST PATTERNS.

### Agent 4 — Senior Engineer (general-purpose)

You are a senior engineer assessing feasibility in: [paste $ARGUMENTS]

Map the architecture — easy to extend vs. needs refactoring. Find quick wins (high impact, low effort). Identify technical blockers and extension points. Return: ARCHITECTURE MAP, QUICK WINS (with effort estimates), TECHNICAL BLOCKERS.

### Agent 5 — QA Analyst (Explore)

You are a QA analyst finding quality gaps in: [paste $ARGUMENTS]

Find untested paths customers use, edge cases, error handling gaps, data validation issues. Return: QUALITY GAPS (ranked by customer impact), UNTESTED SCENARIOS.

**Gate**: Wait for all 5. Proceed with at least 3 substantive results.

---

## Phase 2: FEATURE SELECTION

**Goal**: Pick the single most impactful feature to build.

Execute yourself — do NOT delegate:

1. Group Phase 1 findings by theme — identify where multiple agents flagged the same issue
2. Score each on: **Customer Impact** (HIGH/MEDIUM/LOW) x **Frequency** x **Feasibility**
3. Select the **#1 feature** — the one with the best impact-to-effort ratio that can be built in this session
4. Define clear scope: what's IN (minimum viable), what's OUT (future work)
5. Write acceptance criteria grounded in the customer problem it solves

Present the selected feature to the conversation:
```
SELECTED FEATURE: [name]
CUSTOMER PROBLEM: [what users struggle with]
SCOPE: [what will be built]
ACCEPTANCE CRITERIA: [3-5 testable criteria]
ESTIMATED EFFORT: [LOW/MEDIUM/HIGH]
```

**Gate**: If no feature has both HIGH customer impact and LOW/MEDIUM effort, present the top 3 to the user and ask which to build. Do NOT build HIGH effort features without user confirmation.

---

## Phase 3: ARCHITECTURE

**Goal**: Plan the implementation.

Launch **1 agent**:

### Agent — Tech Lead (general-purpose)

Design the architecture for the selected feature:
1. **File-by-file change plan**: path, changes, design decisions, dependencies
2. **Test strategy**: unit, integration, edge case tests from acceptance criteria
3. **Implementation order**: dependency graph

Read existing code to match patterns. Reference exact file paths and function names. Do NOT write code.

**Gate**: Verify plan addresses all acceptance criteria. Revise if gaps exist.

---

## Phase 4: IMPLEMENT

**Goal**: Write the feature code and tests.

Launch **2 agents in parallel**:

### Agent 1 — Core Implementer (general-purpose)

Implement the feature following the architecture plan. Follow existing patterns, no comments, minimal surface. List every file created or modified.

### Agent 2 — Test Writer (general-purpose)

Write tests per the test strategy and acceptance criteria. Cover happy path, edge cases, the customer pain points from Phase 1. Run the full test suite and report results.

**Gate**: Run the full test suite yourself to verify everything passes.

---

## Phase 5: REVIEW

**Goal**: One review round focused on customer value and correctness.

Launch **3 agents in parallel**:

### Agent 1 — Senior Reviewer (general-purpose)

Review ALL changes via `git diff`. Check: CORRECTNESS, PATTERNS (codebase conventions), TESTS (meaningful assertions), MINIMAL SURFACE. Flag CRITICAL or SUGGESTION.

### Agent 2 — Virtual Customer Validator (Explore)

As a real user, trace through the new feature. Does it solve the customer problem identified in Phase 1? Is the workflow intuitive? Any confusion or friction? Return: CUSTOMER PROBLEM SOLVED (yes/no), UX ASSESSMENT, REMAINING GAPS.

### Agent 3 — QA Validator (Explore)

Review test coverage against acceptance criteria. Check edge cases from Phase 1 are handled. Verify no existing workflows are broken. Return: ACCEPTANCE CRITERIA (PASS/FAIL each), COVERAGE GAPS.

**After reviews**: If CRITICAL issues, launch a fix agent. One fix round maximum. If Virtual Customer says problem not solved, flag for user attention in PR.

---

## Phase 6: SHIP

**Goal**: Branch, commit, push, PR.

Execute yourself — do NOT delegate:

1. `git checkout -b feature/<descriptive-name>`
2. `git add <specific files>` — never use `git add .`
3. Commit:
   ```
   git commit -m "$(cat <<'EOF'
   feat: <concise description>

   Customer problem: <one-line description of the pain point this solves>

   - <key change 1>
   - <key change 2>

   Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
   EOF
   )"
   ```
4. `git push -u origin feature/<branch-name>`
5. Create PR:
   ```
   gh pr create --title "feat: <description>" --body "$(cat <<'EOF'
   ## Customer Problem
   <what real users struggle with — from discovery research>

   ## Solution
   <what was built and why this approach>

   ## Discovery Evidence
   <which agents identified this need, key findings>

   ## Acceptance Criteria
   <criteria with PASS/FAIL>

   ## Test Plan
   - [ ] All tests pass
   - [ ] Customer problem validated by Virtual Customer agent
   - [ ] Edge cases from QA covered
   - [ ] Code reviewed

   ## What's NOT Included (future work)
   <features deliberately deferred — from Phase 2 scoping>

   🤖 Discovered and delivered with multi-agent orchestration
   EOF
   )"
   ```
6. Output the PR URL, the customer problem solved, and a summary.

---

## Error Handling

- Agent failure: retry once, then proceed without and note the gap.
- Tests fail: one fix round in Phase 5. If still failing, list in PR.
- No HIGH-impact feature found: present findings as a discovery report (like /discover-feature) instead of forcing implementation.
- Ambiguous input: ask user for clarification before Phase 1.
