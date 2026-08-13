---
title: "{concise plan title, imperative mood}"
type: "feat"
status: active
date: "YYYY-MM-DD"
origin: "docs/prd/{prd-filename}.md"
depth: "Standard"
---

## Overview

<!-- placeholder: 2-3 sentences naming what this plan delivers and the PRD it implements. No solution detail yet, no code. -->

## Problem Frame

<!-- placeholder: Restate the user/system problem in plan terms. Why this work, why now, what breaks if we don't ship it. Mirror the PRD's Problem Frame in one paragraph. -->

## Requirements Trace

<!-- placeholder: Map each PRD requirement ID to its implementation intent. Use a bullet per requirement: `- **R1**: {how this plan satisfies it}`. Every R# from the PRD must appear; missing IDs block handoff. -->

## Scope Boundaries

<!-- placeholder: Two subsections, `### In Scope` and `### Out of Scope`, each a bullet list. Out-of-scope items cite why deferred, not rejected. -->

## Context & Research

<!-- placeholder: Summarize prior art, existing code paths, docs consulted, and constraints discovered during planning. Link repo-relative paths (e.g., `src/auth/session.ts`) and cite external sources by name; full URLs go in Sources & References. -->

## Key Technical Decisions

<!-- placeholder: Numbered decisions with a one-line rationale each. Format: `1. **{Decision}**: {why; alternative considered; tradeoff accepted}`. These are binding for the U-blocks below. -->

## Open Questions

<!-- placeholder: Split into two subsections below. Resolved items are frozen; deferred items route to implementation and must carry a `[U#]` tag pointing to the unit that will answer them. -->

### Resolved During Planning

<!-- placeholder: `- Q1. {question}`. **Resolved**: {answer, citing decision # or source}. -->

### Deferred to Implementation

<!-- placeholder: `- Q2. [U#] {question}`. One-liner, owning unit, no invented answers. -->

## Implementation Units

<!-- placeholder: One U-block per atomic, testable unit of work. Unit IDs are stable (U1, U2, ...) and never renumber. Follow the 7-field schema below exactly; downstream skills regex against it. -->

- [ ] U1. **[Name]**
  **Goal:** {one sentence; the observable change this unit produces}
  **Requirements:** [R#, R#]
  **Dependencies:** [None | U-id | external prerequisite]
  **Files:** Create/Modify/Test (repo-relative paths only)
  **Approach:** {2-4 sentences; the technical shape, no code}
  **Test scenarios:** Happy / Edge / Error / Integration
  **Verification:** {exact command or observable signal that proves this unit is done}

### Parallel Execution Plan

```
Wave 1 (parallel, zero deps):
  ├── U1  [category]
  └── U2  [category]

Wave 2 (after U1):
  └── U3  [category]

Critical path:  U1 → U3
Max parallelism: 2 (Wave 1)
```

## System-Wide Impact

<!-- placeholder: Cross-cutting effects: migrations, API surface changes, config keys, feature flags, telemetry, docs to update. List files touched by more than one unit here. -->

## Risks & Dependencies

<!-- placeholder: Risks table below. Dependencies (teams, services, libs, env) follow as a bullet list. -->

| # | Risk | Likelihood | Impact | Mitigation |
|---|------|------------|--------|------------|
| 1 | {risk} | Low/Med/High | Low/Med/High | {concrete mitigation, owner if known} |

## Sources & References

<!-- placeholder: Numbered list of authoritative sources consulted: PRD path, ADRs, RFCs, external docs, tickets. Repo-relative paths for internal, full URLs for external. -->

## Completion Status

`PLANNED`
