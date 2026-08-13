# Adversarial Review — Oracle 5-Dimension PRD Stress Test

This file is loaded in **Phase 5: Adversarial Review**. It contains the full Oracle subagent prompt, the five review dimensions, the convergence guard, and the finding-classification router.

---

## When to Invoke

Fire Oracle review when the PRD draft is complete (end of Phase 4) and BEFORE writing to disk (Phase 6). The review operates on the in-memory draft, not the written file.

**Gate criterion to enter Phase 5:**
- All requirements have stable IDs
- All Outstanding Questions are categorized (blocking vs deferred)
- Approaches and Recommended Approach are present (for Standard+ scope)
- The Finalization Checklist (12 items in `prd-template.md`) has been self-run

If the draft fails the pre-gate, loop back to Phase 3 or Phase 4 first.

---

## Oracle Invocation Pattern

Dispatch ONE oracle subagent in **foreground** (blocking). This is not parallel — we want a focused, independent pass. Max 3 review rounds with convergence detection.

```typescript
task(
  subagent_type="oracle",
  load_skills=[],
  run_in_background=false,
  description="Adversarial PRD review — round {N}",
  prompt=ORACLE_REVIEW_PROMPT  // constructed from template below
)
```

---

## Oracle Review Prompt Template

Construct the full prompt using this scaffold. Variables in `{braces}` come from brainstorm's working state.

```
You are performing an adversarial review of a Product Requirements Document produced by the `brainstorm` skill. Your job is to find what is wrong, missing, or dangerously under-specified — not to validate or praise.

## Review Round
Round {N} of max 3.

## Scope Tier of this PRD
{Lightweight | Standard | Deep-feature | Deep-product}

## Source Inputs
- Research markdown: {path to research.md, or "cold-start (no research provided)"}
- User interrogation answers: {inline summary of Phase 3 answers with direct quotes}

## PRD Draft Under Review

<prd_draft>
{full PRD draft content}
</prd_draft>

## Prior Decisions (round 2+)

<prior_decisions>
{list of findings from previous rounds:
- What was flagged
- Decision: applied / rejected / deferred
- Evidence / reasoning for the decision}
</prior_decisions>

(For round 1, state: "This is round 1 — no prior decisions.")

## Your Task

Review the draft on exactly FIVE dimensions. For each dimension, produce findings in the structured format below. Do not comment on dimensions that are clean — silence is signal.

### Dimension 1: Coherence
Hunt for internal contradictions and terminology drift.
- Do any two requirements contradict each other?
- Does terminology stay consistent across sections?
- Do Acceptance Examples actually test the Requirements they claim to cover?
- Do Flows reference Actors that exist in the Actors section?

### Dimension 2: Feasibility
Hunt for requirements that cannot be built as specified.
- Are any requirements physically or computationally impossible?
- Do any requirements conflict with stated Dependencies?
- Are non-functional requirements (performance, scale) grounded or hand-waved?
- Does the Recommended Approach actually deliver all claimed Requirements?

### Dimension 3: Scope Clarity
Hunt for ambiguity about what is in versus out.
- Is the Narrowest Wedge truly narrow, or is it a platform in disguise?
- Do "In scope" and "Out of scope" items actually align with Requirements?
- Are there implicit requirements hidden in Approaches that aren't in the Requirements list?
- For Deep-product: is "Outside Product Identity" meaningful, or a dumping ground?

### Dimension 4: Assumption Surfacing
Hunt for unstated assumptions masquerading as premises.
- Are there Premises that the user did not explicitly agree to?
- Are there assumptions baked into Requirements that should be Outstanding Questions?
- Is Demand Evidence grounded in behavior, or in stated intent?
- Does the Future-Fit / durability thesis hold under obvious alternative scenarios?

### Dimension 5: Handoff Readiness
Hunt for gaps that would force `plan` to invent product behavior.
- If `plan` ran on this PRD, what product decision would it still have to make?
- Do Success Criteria include measurable, observable outcomes?
- Is the Distribution Plan concrete enough that "go-to-market" is not a black box?
- Is "The Assignment" a real-world validation step, not a platform-internal invocation?

## Output Format

Return findings in this exact XML schema:

<review_findings round="{N}">
  <finding id="F{N}.1">
    <dimension>{Coherence | Feasibility | Scope | Assumption | Handoff}</dimension>
    <severity>{critical | high | medium | low | fyi}</severity>
    <location>{section name, e.g., "Requirements R3" or "Approaches Considered → Approach B"}</location>
    <issue>{1-2 sentence description of the problem}</issue>
    <evidence>{direct quote from the PRD showing the issue}</evidence>
    <proposed_fix>{concrete, minimal change that resolves the issue}</proposed_fix>
    <confidence>{100 | 75 | 50}</confidence>
  </finding>
  ...
</review_findings>

<review_summary>
  <convergence_signal>
    {Compare this round's findings to prior rounds:
     - "New issues found" / "Same issues persisting" / "All prior issues resolved" }
  </convergence_signal>
  <overall_readiness>
    {one of: "handoff-ready" | "needs-rework" | "critical-gaps"}
  </overall_readiness>
  <top_3_concerns>
    {if "needs-rework" or "critical-gaps", list the 3 most important findings by ID}
  </top_3_concerns>
</review_summary>

## Severity Rubric

- **critical**: PRD is actively misleading. Fix before any handoff.
- **high**: Will cause `plan` to stall or invent product decisions. Fix before handoff.
- **medium**: Weakens the PRD but does not block handoff. Fix if cheap.
- **low**: Polish. Note but do not block.
- **fyi**: Observation without a recommended action.

## Confidence Rubric

- **100**: Finding is backed by direct textual evidence in the PRD and falsifiable.
- **75**: Finding is well-supported but could be resolved by content not visible in the PRD.
- **50**: Interpretive concern. Suppress if confidence would drop below 50 — do not surface speculation.

## Forbidden Behaviors

- Do NOT praise the PRD or mark dimensions as "strong" — only surface what's wrong.
- Do NOT re-litigate findings from prior rounds that were marked "rejected" in <prior_decisions>, unless NEW evidence has appeared.
- Do NOT invent requirements; only flag what is present or absent.
- Do NOT suggest nice-to-haves; limit to issues that affect correctness, feasibility, scope, assumptions, or handoff.
```

---

## Convergence Guard

After each round, compare findings to prior rounds:

| Signal | Meaning | Action |
|---|---|---|
| **Zero critical + zero high** findings in this round | Converged. | Exit review loop, proceed to Phase 6 (Write). |
| **Same findings persist** across 2 consecutive rounds | Stuck. | Exit loop. Persist unresolved findings to PRD as `Reviewer Concerns` subsection under Outstanding Questions. Do NOT attempt another fix. |
| **New issues surface** on round 2 or 3 | Still discovering. | Continue to next round, apply fixes, re-review. |
| **Round 3 completed without convergence** | Hard ceiling. | Exit loop. Persist all outstanding findings as `Reviewer Concerns`. |

**The convergence guard is mandatory** — it prevents infinite loops where fixes create new issues that create new fixes ad infinitum.

---

## Finding-Classification Router (Phase 5 → Phase 6)

After each review round, classify findings into three buckets:

### `safe_auto` — Apply silently
Conditions (ALL must hold):
- Severity: `low` or `medium`
- Confidence: 100
- Dimension: Coherence or Handoff (mechanical issues)
- Fix is unambiguous (single obvious correction)

Examples: terminology drift ("user" vs "customer" used inconsistently), a Flow referencing a non-existent Actor ID, a section missing from the Section Matrix for this tier.

### `gated_auto` — Preview then apply in bulk
Conditions:
- Severity: `high` or `medium`
- Confidence: 75 or 100
- Fix affects one section only
- No new product decisions required

Show the user: "Round {N} review found {count} fixes. Apply all?" — single yes/no. If yes, apply. If no, downgrade to `manual`.

### `manual` — Walk through one at a time
Conditions (ANY triggers manual):
- Severity: `critical`
- Dimension: Assumption Surfacing or Scope Clarity (requires product judgment)
- Fix requires a new user decision
- Finding is cross-cutting (affects multiple sections)

For each manual finding, surface via the blocking question tool: show the evidence, the proposed fix, and ask the user to approve / modify / reject.

### `fyi` — Advisory, no decision
Conditions:
- Severity: `fyi` or `low` with confidence 50

Append to the PRD's `Reviewer Concerns` subsection (if created) but do not require action.

---

## Integration With PRD

If convergence fails OR if any finding is user-rejected, add this subsection to the PRD's `Outstanding Questions` section:

```markdown
### Reviewer Concerns (persisted from adversarial review)

Findings from {N} rounds of adversarial review that were not resolved in this revision. These are NOT blockers for planning unless marked `[blocking]`.

- **F1.2** [Assumption][medium]: {issue summary}
  - **Evidence**: "{direct quote from PRD}"
  - **Proposed fix**: {fix}
  - **User decision**: {accepted | rejected | deferred}
  - **Rationale**: {why this decision was made}
```

---

## Quick Operational Summary

1. **Entry check** — is PRD draft past Finalization Checklist? If no, loop back to Phase 3/4.
2. **Round 1** — fire oracle with prompt template. No `<prior_decisions>`.
3. **Classify findings** — safe_auto / gated_auto / manual / fyi.
4. **Apply or walk through** — based on classification.
5. **Round 2** — re-run review on revised draft. Include `<prior_decisions>` block summarizing round 1 outcomes.
6. **Convergence check** — if zero critical+high OR same findings as round 1, exit.
7. **Round 3 (if needed)** — same pattern. Ceiling enforced.
8. **Persist unresolved** — any remaining findings become `Reviewer Concerns` in Outstanding Questions.
9. **Proceed to Phase 6** — write file.
