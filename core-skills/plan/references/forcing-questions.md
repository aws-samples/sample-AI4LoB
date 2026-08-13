# Forcing Questions, Cold-Start + Gap-Scan Banks

<role>
This reference is consulted by the `plan` skill during Phase 0 (cold-start) and Phase 3 (Socratic open-question loop). Use ONE question at a time. Never batch. Prefer single-select options. Skip any question whose answer is already present in the PRD.
</role>

## Usage Rules

- Ask ONE question per turn. Wait for answer before next.
- Use `question` tool with options when possible (single-select default).
- Frame every question via the **4-part template**:
  1. **Re-ground**, 1-sentence summary of project/branch/task state
  2. **Simplify**, plain-English explanation (no jargon)
  3. **Recommend**, `RECOMMENDATION: [X] because [reason]. Completeness: X/10`
  4. **Options**, lettered A/B/C with effort shown as `human: ~X / CC: ~Y`
- STOP once information gain drops below threshold (after 3-4 meaningful answers in cold-start, 1-2 in gap-scan).
- If the user answers "don't know" or "you pick", record the recommendation as the working assumption and move on. Don't loop.
- Every answer should resolve a concrete plan field. If you can't name the field, don't ask the question.
- Quote the user's answer verbatim into the plan (don't paraphrase), then attribute with `(per user, Phase 0/3)`.

---

## Bank A: Cold-Start Questions (no PRD)

*Entry condition: `plan` skill invoked WITHOUT a PRD path argument. Use 6-8 questions ordered by information gain.*

### Q1, Scope anchor
**Trigger:** Always, first question in cold-start.
**What answer unblocks:** Whether to classify as Lightweight/Standard/Deep.
**Question template:** "How big is this change? (A) Single file, single function. (B) New feature touching 2-4 modules. (C) Cross-cutting / new subsystem."
**Derived field:** `depth`

### Q2, Success criteria
**Trigger:** After Q1.
**What answer unblocks:** Requirements Trace section, converts intent into R#s.
**Question template:** "When is this 'done'? List 2-5 observable outcomes. (e.g., 'CLI accepts --flag X', 'endpoint returns 200 with payload Y')."
**Derived field:** `R1..R5` seeds

### Q3, Surface area
**Trigger:** After Q2.
**What answer unblocks:** Files section per unit.
**Question template:** "Which files/modules will change? List paths you already know. If unclear, name the closest anchor (a neighbor file or directory)."
**Derived field:** Initial `Files` list per unit

### Q4, Dependency assumptions
**Trigger:** After Q3.
**What answer unblocks:** Context & Research section; `docs/solutions/` consultation decision.
**Question template:** "Are we building on existing code (Reuse / Extend) or net-new (Build)? Name the existing system if Reuse/Extend."
**Derived field:** Architectural posture

### Q5, Unknown unknowns
**Trigger:** After Q4, only if user signals uncertainty.
**What answer unblocks:** Risks & Dependencies table.
**Question template:** "What could go wrong that you haven't thought about yet? Even speculation is fine, we'll classify as 'Deferred'."
**Derived field:** `RK#` risks

### Q6, Completion gate
**Trigger:** Before Phase 5 (unit enumeration).
**What answer unblocks:** Success Criteria section.
**Question template:** "What automated or manual check proves this is done? (test run? build exit code? manual verification? external user confirmation?)"
**Derived field:** `Success Criteria` checklist

### Q7, Deferred concerns
**Trigger:** Optional, only for Deep tier.
**What answer unblocks:** Scope Boundaries's "Deferred to Follow-Up Work" subsection.
**Question template:** "What's explicitly NOT in scope but adjacent? (helps future-planning)"
**Derived field:** `Deferred Follow-Up` list

### Q8, Compounding hook
**Trigger:** Optional, only if `docs/solutions/` exists.
**What answer unblocks:** Institutional Learnings consultation.
**Question template:** "Have we solved something similar before? If yes, point me at the doc/file."
**Derived field:** Context & Research → `Institutional Learnings`

---

## Bank B: Gap-Scan Questions (PRD exists)

*Entry condition: PRD was provided. Scan for gaps. Ask ONLY questions whose answer isn't already in the PRD. Target: 0-4 questions.*

### GQ1, Unresolved R#/A#
**Trigger:** PRD has `Outstanding Questions → Resolve Before Planning` non-empty.
**What answer unblocks:** Unblocks the plan handoff (brainstorm gates this).
**Question template:** "Before I plan, the PRD flags [N] questions as blocking. Let's resolve them one by one. Starting with: [quote first question]."
**Derived field:** Resolves PRD's blocking questions

### GQ2, Ambiguous F#
**Trigger:** A feature in PRD lacks Acceptance Example.
**What answer unblocks:** Test scenarios per unit.
**Question template:** "Feature F[N] in the PRD ('[summary]') has no acceptance example. Give me one input-action-outcome triple so I can write test scenarios."
**Derived field:** `AE#` → Test scenarios

### GQ3, Missing dependency declaration
**Trigger:** PRD requirements imply cross-module touch but no dependency listed.
**What answer unblocks:** Unit `Dependencies:` field accuracy.
**Question template:** "R[N] mentions [module A] and [module B]. Does A depend on B, B on A, or independent?"
**Derived field:** Unit-level `Dependencies:` values

### GQ4, NFR ambiguity
**Trigger:** Non-functional requirements mentioned but unquantified (e.g., "fast", "scalable").
**What answer unblocks:** Test scenarios for perf/scale.
**Question template:** "PRD says '[NFR]'. What's the concrete bar? (latency target, concurrency count, error rate)"
**Derived field:** Quantified NFR in Test scenarios

### GQ5, Unsourced claim
**Trigger:** PRD makes claim without `research.md` citation.
**What answer unblocks:** Whether to trust claim or flag for research.
**Question template:** "PRD states '[claim]'. Source? If none, should I accept as axiom or run `deepresearch` first?"
**Derived field:** Research gate

### GQ6, Scope drift detector
**Trigger:** PRD's stated scope and Actor/Flow inventory disagree.
**What answer unblocks:** Scope Boundaries section fidelity.
**Question template:** "PRD scope says [X] but Actors/Flows imply [Y]. Which is authoritative?"
**Derived field:** Authoritative scope statement

---

## Question Ordering Principle

Order questions by **information gain**, each answered question should shrink the space of possible plans by the largest amount. Generally:

1. Scope first (Q1 / GQ6), answers change everything downstream
2. Requirements next (Q2 / GQ1 / GQ2)
3. Context/dependencies (Q3, Q4 / GQ3)
4. Quality bars (Q6 / GQ4)
5. Speculation last (Q5 / Q7 / GQ5)

A rough heuristic: if two questions are both unanswered, ask the one whose answer would change the **plan tier** (Lightweight vs Standard vs Deep). Tier changes cascade; detail questions don't.

## Stopping Conditions

End the question loop when ANY of these hit:

- Three consecutive answers produced no new plan field (diminishing returns).
- The user says "enough" or "just plan it" (respect the signal, note remaining gaps in `Open Questions`).
- You can draft every required section of the plan template without hedging words like "probably" or "TBD".
- Cold-start: max 8 questions (the full Bank A). Gap-scan: max 4 questions.

Anything still unresolved becomes a bullet in the plan's `Open Questions` section, not another round of questioning.

## Anti-Patterns

- ❌ Asking questions whose answer is in the PRD already
- ❌ Asking 2+ questions in one turn
- ❌ Yes/no questions when single-select-from-list is possible
- ❌ Open-ended "anything else?" questions (user doesn't know what they don't know)
- ❌ Asking for details the `execute` skill will ask later anyway
- ❌ Re-asking a question after the user declined to answer ("you pick" means you pick)
- ❌ Chaining questions mid-answer ("and also, while we're at it...")
- ❌ Asking about implementation details before scope is fixed
