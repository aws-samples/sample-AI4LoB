# PRD Template — Right-Sized, Stable-ID, Handoff-Ready

This file is loaded in **Phase 6: Write PRD**. Do not load it earlier — it pollutes context.

## Scoping Preface — READ FIRST

Match the document's weight to the problem's ambiguity. There are three scope tiers, established in Phase 0.

| Tier | When | Signals |
|---|---|---|
| **Lightweight** | Scoped enhancement, small surface area | One subsystem touched · Clear user problem · Few unknowns |
| **Standard** | Normal feature work | 2-4 requirements · Some ambiguity · Decision needed on approach |
| **Deep-feature** | Substantial feature crossing surfaces | 5+ requirements · Architectural choice · Non-obvious risks |
| **Deep-product** | Establishes new product shape / identity | New user category · Strategic bet · Carrying cost is durable |

## Section Matrix — What to Include by Tier

`✓` = required · `triggered` = include only when activated by a signal · `—` = skip

| Section | Lightweight | Standard | Deep-feature | Deep-product |
|---|---|---|---|---|
| Problem Frame | ✓ | ✓ | ✓ | ✓ |
| Demand Evidence | ✓ | ✓ | ✓ | ✓ |
| Status Quo | — | ✓ | ✓ | ✓ |
| Target Users & Narrowest Wedge | ✓ | ✓ | ✓ | ✓ |
| Actors (A#) | triggered | triggered | ✓ | ✓ |
| Requirements (R#) | ✓ | ✓ | ✓ | ✓ |
| Key Flows (F#) | triggered | triggered | ✓ | ✓ |
| Acceptance Examples (AE#) | triggered | triggered | ✓ | ✓ |
| Premises | triggered | ✓ | ✓ | ✓ |
| Approaches Considered | — | ✓ (≥2) | ✓ (≥2) | ✓ (≥3) |
| Recommended Approach | — | ✓ | ✓ | ✓ |
| Success Criteria | ✓ | ✓ | ✓ | ✓ |
| Scope Boundaries | single list | single list | single list | **split: Deferred / Outside Identity** |
| Distribution Plan | triggered | triggered | ✓ | ✓ |
| Dependencies & Assumptions | triggered | triggered | ✓ | ✓ |
| Outstanding Questions (split) | ✓ | ✓ | ✓ | ✓ |
| The Assignment | ✓ | ✓ | ✓ | ✓ |
| What I noticed about how you think | triggered | ✓ | ✓ | ✓ |

**Triggered sections fire when:**
- **Actors**: more than one distinct role interacts
- **Key Flows**: sequence/timing matters for correctness
- **Acceptance Examples**: edge cases or numeric thresholds exist
- **Premises**: a non-obvious assumption underlies the approach
- **Distribution Plan**: users must find/adopt this (anything user-facing)
- **Dependencies**: external systems, libraries, or teams are required
- **What I noticed**: user revealed a repeatable thinking pattern worth reflecting

## Stable ID Rules — NON-NEGOTIABLE

- `R1`, `R2`, ... — Requirements
- `A1`, `A2`, ... — Actors
- `F1`, `F2`, ... — Flows
- `AE1`, `AE2`, ... — Acceptance Examples

**IDs are permanent.** When an item is deleted, split, merged, or reordered, the original IDs DO NOT renumber. Gaps are correct. Downstream skills (planning, implementation) reference these IDs; renumbering breaks traceability.

When splitting `R3` into two requirements, the result is `R3a` and `R3b` (or `R3` becomes a parent and new leaf gets next unused integer like `R7`). Never recycle a deleted ID.

## Outstanding Questions — The Handoff Gate

This section MUST be split into two subsections, even if one is empty:

```markdown
## Outstanding Questions

### Resolve Before Planning
- [Affects R1][User decision] Which tier of support are paid users entitled to?
- [Affects R3][Product scope] Should offline mode be a launch requirement?

### Deferred to Planning
- [Affects R2][Technical] Which caching layer fits the read pattern?
- [Affects R4][Needs research] What does the existing auth middleware expose?
```

**Format:** `- [Affects R#][category] Question text`

**Categories:**
- `User decision` — only the requester can answer; product judgment
- `Product scope` — affects what we build, not how
- `Technical` — implementation detail, safe to defer
- `Needs research` — requires code exploration or external investigation

**Gate rule:** If `Resolve Before Planning` is non-empty, the PRD is **NOT handoff-ready**. Phase 7 must hide the "Proceed to Planning" option.

---

## PRD Document Template

Fill this out verbatim in the language of the user's input. Omit sections per the Section Matrix. Do not invent content — if a section has no substance, mark it `_(not applicable for this scope)_` and move on.

```markdown
# PRD: {title}

**Generated**: {YYYY-MM-DD HH:MM} by `brainstorm` skill
**Source Research**: `{path to research.md}` (or `cold-start` if none)
**Scope Tier**: {Lightweight | Standard | Deep-feature | Deep-product}
**Status**: DRAFT
**Supersedes**: {prior PRD filename, if this is a revision; else `—`}

---

## Problem Frame

{2-4 sentences. What problem, for whom, why now. Grounded in research.md evidence or user's answers. No solution language yet.}

## Demand Evidence

{From Forcing Question Q1. What proves someone actually wants this — not "is interested," not "signed a waitlist," but would be upset if it disappeared? Cite research.md sections by heading where possible.}

- **Strongest signal**: {...}
- **Weakest signal (for honesty)**: {...}
- **What we don't yet know**: {...}

## Status Quo

{From Q2. What are users doing RIGHT NOW to solve this — even badly? What does the workaround cost them?}

## Target Users & Narrowest Wedge

{From Q3 + Q4. Name the actual human. Title, what gets them promoted, what keeps them up at night. Then: the smallest possible version of this that someone would pay real money for this week.}

- **Primary user**: {role, context}
- **Narrowest wedge**: {smallest buyable slice}
- **Out of scope for wedge**: {what we explicitly do NOT target first}

## Actors

- **A1**: {role} — {their goal in this system}
- **A2**: {role} — {their goal in this system}

## Requirements

- **R1** — {single-sentence behavior statement in active voice}
  - **Rationale**: {why this requirement exists; tie to Problem Frame}
  - **Type**: {functional | non-functional | constraint}
- **R2** — ...

## Key Flows

- **F1: {flow name}** — {1-2 sentence description}
  1. {step}
  2. {step}
  3. {step}

## Acceptance Examples

- **AE1** (covers R1, R3): Given {setup}, when {action}, then {observable outcome}.
- **AE2** (covers R2): ...

## Premises

Numbered assumptions the user has explicitly agreed to (Phase 3 output). If unagreed, they belong in Outstanding Questions, not here.

1. {premise}
2. {premise}

## Approaches Considered

### Approach A: {name} — {minimal | ideal | lateral}
- **Description**: {2-3 sentences}
- **Pros**: {...}
- **Cons**: {...}
- **Risks**: {...}
- **When best**: {conditions under which this is the right call}

### Approach B: {name} — {minimal | ideal | lateral}
{same structure}

### Approach C: {name} — {lateral — non-obvious angle: inversion / constraint-removal / cross-domain analogy}
{same structure}

## Recommended Approach

**Selection**: Approach {A | B | C}
**Label**: {Reuse | Extend | Build new}
**Reasoning**: {why this one wins given the premises, demand evidence, and scope tier}
**What would flip the recommendation**: {concrete condition}

## Success Criteria

Outcome-level, measurable, time-bounded where possible. Cover BOTH human outcome AND downstream-agent handoff quality.

- {criterion — e.g., "90% of target users complete wedge flow in under 2 minutes within first session"}
- {criterion}
- **Handoff quality**: A `plan` invocation on this PRD produces an implementation plan without requiring net-new product decisions.

## Scope Boundaries

### In scope (this increment)
- {explicit inclusion}

### Out of scope (deferred)
- {item} — {why deferred, not rejected}

### _(Deep-product only)_ Outside Product Identity
- {item} — {why this never belongs, even later}

## Distribution Plan

How does a user discover, receive, and adopt this? Code without distribution is code nobody uses.

- **Discovery**: {how users learn this exists}
- **Onboarding**: {first-run experience, if any}
- **Adoption signal**: {what "picked up" looks like}

## Dependencies & Assumptions

- **Depends on**: {system / team / external API / library}
- **Assumes**: {environmental condition that must hold}

## Outstanding Questions

### Resolve Before Planning
- [Affects R#][category] {question}

### Deferred to Planning
- [Affects R#][category] {question}

## The Assignment

**One concrete, real-world next action** — not "go plan this." Something the user does in the next 24-72 hours that validates or refines the PRD itself.

Examples of good assignments:
- "Show this PRD to three target users and record which requirements they push back on."
- "Run `ce-plan` on this PRD and see which Outstanding Questions surface as blockers."
- "Sketch the wedge UI by hand, send to {specific person}, ask what's missing."

## What I Noticed About How You Think

2-4 quoted callbacks to things the user said during the interrogation. Use their actual words in quotes. This is not flattery — it's pattern reflection that helps the user see their own reasoning shape.

- "{direct quote}" — this suggests you {pattern observed}.
- "{direct quote}" — worth noticing: {pattern observed}.
```

---

## Finalization Checklist — RUN BEFORE WRITING THE FILE

Self-audit the draft against these 12 questions. If ANY answer is "no" or "unclear," fix before writing to disk.

1. Does every requirement trace to either research.md or an explicit user answer?
2. Are requirement IDs stable — no renumbering after edits?
3. Is every premise one the user explicitly agreed to (not the skill's invention)?
4. Are there at least 2 approaches (3 for Deep-product), with at least one non-obvious angle?
5. Is the recommendation stated AFTER the options, not woven into them?
6. Does Outstanding Questions split blocking vs deferred correctly?
7. If `Resolve Before Planning` is non-empty, is the handoff gate respected in Phase 7?
8. Do Success Criteria include both user outcome AND downstream-agent handoff quality?
9. Is the Distribution Plan concrete (not "TBD" or "marketing handles it")?
10. Is "The Assignment" a real-world action, not a skill invocation?
11. Is the language matching the user's input language?
12. **The key question**: If `plan` ran on this PRD right now, what product decision would it still have to invent? — If anything, the PRD is not done.

If #12 surfaces gaps, loop back to Phase 3 and ask the user; do not paper over with guesses.
