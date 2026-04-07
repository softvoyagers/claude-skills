---
description: Read-only customer needs analysis from multiple perspectives
---

# /discover-feature — Multi-Agent Feature Discovery

Discover what customers actually need by analyzing the product from multiple perspectives. Input: a product area, user segment, or pain point to investigate. Output: a prioritized list of feature recommendations grounded in real customer needs.

**Discovery focus**: $ARGUMENTS

---

## Execution Protocol

You are the **Orchestrator**. You coordinate 5 specialized agents across 3 phases. This is a **research-only** command — no code is written. You synthesize customer insights into actionable feature recommendations.

### Conventions (enforce in all agent prompts)

- **READ-ONLY**: No file writes, no code edits, no branch creation
- **Customer-first**: Every recommendation must trace back to a real user problem
- **Evidence-based**: Back claims with data from codebase analysis, usage patterns, or industry research
- **Prioritize by impact**: Rank by customer value, not technical coolness

---

## Phase 1: CUSTOMER RESEARCH

**Goal**: Understand the customer landscape from multiple perspectives.

Launch **5 agents in parallel**:

### Agent 1 — Virtual Customer (Explore)

You are a real end-user of this product. Explore the codebase to understand the current user experience in the area of: [paste $ARGUMENTS]

1. Walk through the current user workflows step by step — what's painful, slow, or confusing?
2. Identify friction points: too many clicks, unclear labels, missing feedback, dead ends
3. Find features that exist but are hard to discover or use
4. Look for error messages and error handling — are they helpful or cryptic?
5. Check for accessibility and usability gaps

Return: TOP PAIN POINTS (ranked by frustration severity), WORKFLOW GAPS, HIDDEN FEATURES, USER FRICTION MAP.

### Agent 2 — UX Designer (Explore)

You are a UX designer auditing the product experience. Explore the codebase in the area of: [paste $ARGUMENTS]

1. Analyze the current UI/UX patterns — consistency, information hierarchy, navigation flow
2. Identify where the experience violates user expectations or common design patterns
3. Look for missing states: empty states, loading states, error states, first-time user experience
4. Evaluate the feedback loop — does the user know what happened after each action?
5. Compare with industry-standard UX patterns for this type of product

Return: UX AUDIT FINDINGS (ranked by impact), MISSING STATES, INCONSISTENCIES, DESIGN DEBT, IMPROVEMENT OPPORTUNITIES.

### Agent 3 — Product Owner (general-purpose)

You are a product owner evaluating feature opportunities. Analyze the codebase and product area of: [paste $ARGUMENTS]

1. Map the current feature set — what exists, what's half-built, what's missing
2. Identify the biggest gaps between what users likely need and what the product provides
3. Search for TODO/FIXME/HACK comments that indicate known product debt
4. Look at git log for recently added features — what direction is the product heading?
5. Search Jira (via ToolSearch for MCP tools) for feature requests, customer complaints, and enhancement tickets

Return: FEATURE GAP ANALYSIS, PRODUCT DEBT LIST, CUSTOMER REQUEST PATTERNS (from Jira if available), MARKET OPPORTUNITIES.

### Agent 4 — Senior Engineer (general-purpose)

You are a senior engineer evaluating technical feasibility. Analyze the codebase in the area of: [paste $ARGUMENTS]

1. Map the architecture — what's easy to extend vs. what requires significant refactoring
2. Identify quick wins: features that would be high-impact but low-effort given the current architecture
3. Identify bottlenecks: what technical debt blocks the most valuable improvements
4. Check for performance issues, scalability limits, or architectural constraints
5. Assess testability — which areas have good test coverage vs. none

Return: ARCHITECTURE MAP, QUICK WINS (with effort estimates), TECHNICAL BLOCKERS, EXTENSION POINTS, TESTABILITY ASSESSMENT.

### Agent 5 — QA Analyst (Explore)

You are a QA analyst looking for quality gaps that affect customers. Explore the codebase in the area of: [paste $ARGUMENTS]

1. Find untested or undertested code paths that customers likely use
2. Identify edge cases that could cause failures in real-world usage
3. Look for error handling gaps — what happens when things go wrong?
4. Check for data validation issues at user input boundaries
5. Review existing tests — do they test real user scenarios or just implementation details?

Return: QUALITY GAPS (ranked by customer impact), UNTESTED SCENARIOS, ERROR HANDLING GAPS, DATA VALIDATION ISSUES.

**Gate**: Wait for all 5 agents. If fewer than 3 return substantive findings, note the gaps and proceed with available data.

---

## Phase 2: SYNTHESIS

**Goal**: Cross-reference all findings into a unified prioritized feature list.

Execute yourself — do NOT delegate. Synthesize Phase 1 outputs:

1. Group findings by theme — identify where multiple agents flagged the same issue
2. For each theme, extract the underlying customer need (not the technical symptom)
3. Score each need on: **Customer Impact** (HIGH/MEDIUM/LOW) x **Frequency** (how many users affected) x **Feasibility** (from Senior Engineer assessment)
4. Rank the top 5-10 feature opportunities
5. For each feature opportunity, define: the customer problem it solves, the proposed solution (brief), effort estimate, dependencies

---

## Phase 3: REPORT

**Goal**: Deliver the prioritized discovery report.

Output the report in this format:

```
# Feature Discovery Report: [Area/Segment]

## Executive Summary
<3-5 sentences: key findings, top opportunities, recommended next steps>

## Top Feature Opportunities (ranked by customer impact)

### 1. [Feature Name]
- **Customer problem**: <what real users struggle with today>
- **Proposed solution**: <brief description>
- **Impact**: HIGH/MEDIUM/LOW — <why>
- **Effort**: LOW/MEDIUM/HIGH — <why>
- **Evidence**: <which agents identified this, what data supports it>

### 2. [Feature Name]
...

### 3-10. ...

## Customer Pain Points Summary
<consolidated list from Virtual Customer and UX Designer>

## Technical Landscape
<key architecture insights from Senior Engineer — what's easy vs. hard to change>

## Quality Gaps
<top quality issues from QA that affect customer experience>

## Product Debt
<existing TODOs, half-built features, known gaps from Product Owner>

## Recommended Next Steps
1. <immediate action>
2. <short-term action>
3. <longer-term action>
```

**Gate**: Every recommendation traces back to a customer need identified by at least one agent. No feature is recommended purely for technical reasons.
