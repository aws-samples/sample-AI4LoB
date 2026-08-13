# Pressure Test — Forcing Questions, Pushback Patterns, Anti-Sycophancy

This file is loaded in **Phase 2: Internal Pressure Test** and **Phase 3: Collaborative Interrogation**. It contains:

1. The 6 Forcing Questions (startup-style interrogation)
2. 5 Pushback Patterns with BAD-vs-GOOD examples
3. Anti-Sycophancy banned phrases
4. Question routing by scope tier
5. Escape hatch rules

---

## Part 1 — The Six Forcing Questions

Each question has four parts: **Ask** (literal prompt to surface to user), **Push Until** (success signal that validates a real answer), **Red Flags** (failure signals that demand a follow-up push), and **Bonus Pushes** (reframings if stuck).

Ask one at a time via the blocking question tool. Never batch.

### Q1 — Demand Reality

**Ask:**
> What's the strongest evidence you have that someone actually wants this — not "is interested," not "signed up for a waitlist," but would be genuinely upset if it disappeared tomorrow?

**Push Until You Hear:**
- A specific person or named segment
- A behavior already happening (workaround, paid alternative, manual process)
- An observed cost of the problem (time, money, missed outcome)

**Red Flags:**
- "People have asked about it"
- "We did a survey"
- Abstract market-sizing language
- "Users would love this"

**Bonus Pushes:**
- "Who is the one person you'd show this to first, and what would you expect them to do in the next 5 minutes?"
- "What have they already tried to solve this?"
- "How much would they pay this month to have it solved?"

### Q2 — Status Quo

**Ask:**
> What are your users doing right now to solve this problem — even badly? What does that workaround cost them?

**Push Until You Hear:**
- A concrete current workflow (tool names, steps, time)
- The cost of the workaround (measurable)
- Why they tolerate it

**Red Flags:**
- "Nothing, that's why we're building this"
- "The problem is unsolved in the market"
- Vague "users are frustrated"

**Bonus Pushes:**
- "Walk me through the last time a user hit this. What did they do in the next hour?"
- "If no tool existed, what would they use — a spreadsheet, Slack, email, memory?"

### Q3 — Desperate Specificity

**Ask:**
> Name the actual human who needs this most. What's their title? What gets them promoted? What gets them fired? What keeps them up at night?

**Push Until You Hear:**
- A role title specific enough that their calendar looks identifiable
- A professional incentive (promotion, retention, bonus)
- A specific failure mode they fear

**Red Flags:**
- "Anyone who does X" — that's not a person
- "Small businesses" — that's a census bureau category
- Persona names without behavioral specificity

**Bonus Pushes:**
- "Is this person the buyer, the user, or both?"
- "What did they do yesterday afternoon between 2pm and 4pm?"

### Q4 — Narrowest Wedge

**Ask:**
> What's the smallest possible version of this that someone would pay real money for — this week, not after you build the platform?

**Push Until You Hear:**
- One screen, one workflow, one outcome
- A price point (even if hypothetical)
- A user who would pay it without further development

**Red Flags:**
- "We need the full platform first"
- "All three features work together"
- A wedge that requires network effects to function

**Bonus Pushes:**
- "If you had to ship something by Friday, what single capability would you ship?"
- "What's the demo video that makes one person pull out a credit card?"

### Q5 — Observation & Surprise

**Ask:**
> Have you actually sat down and watched someone use this (or its closest analog) without helping them? What did they do that surprised you?

**Push Until You Hear:**
- A specific observation (user clicked X, ignored Y, asked Z)
- A surprise that contradicted an assumption
- Evidence of a reframe post-observation

**Red Flags:**
- "We haven't run sessions yet"
- "We showed it to the team and they liked it"
- Only quantitative data, no behavioral observation

**Bonus Pushes:**
- "What's the feature users kept trying to use that doesn't exist yet?"
- "What did they skip that you thought was essential?"

### Q6 — Future-Fit

**Ask:**
> If the world looks meaningfully different in 3 years — AI capability jumps, regulation shifts, the primary platform changes — does your product become more essential or less?

**Push Until You Hear:**
- A specific change vector (not "things will change")
- A thesis on how this product absorbs or resists that change
- A concrete scenario where this becomes more valuable

**Red Flags:**
- "AI won't affect us"
- "Our moat is execution"
- Generic "we'll adapt"

**Bonus Pushes:**
- "If Anthropic/OpenAI/Google shipped a 1-prompt version of this tomorrow, what remains yours?"
- "What does your product look like when compute is 10x cheaper?"

---

## Part 2 — Research-Grounded Question Adaptations

When `research.md` is provided, questions gain a **grounding clause** that references the research directly:

### Grounded Q1 (Demand Reality, research-grounded)
> The research cites {evidence source} as demand signal. How confident are you that this translates to someone being upset if the product disappeared tomorrow? What's the strongest non-survey evidence?

### Grounded Q2 (Status Quo, research-grounded)
> The research mentions {current-state observation}. Can you name a specific person and their current workaround, including what it costs them?

### Grounded Q3 (Specificity, research-grounded)
> The research identifies {user segment}. Zoom in: who is the one human in that segment you'd ship to first? Title, incentives, daily rhythm.

### Grounded Q4 (Wedge, research-grounded)
> The research proposes {scope}. What's the ≤20% of that scope someone would pay for this week?

### Grounded Q5 (Observation, research-grounded)
> The research cites {user-behavior data}. Have you directly observed — not surveyed — a user in this scenario? What surprised you?

### Grounded Q6 (Future-Fit, research-grounded)
> The research assumes {stable condition}. If that condition shifts in 3 years, does this product become more essential or less?

---

## Part 3 — Scope-Tier Question Routing

| Scope | Questions to ask (min → max) | Skip rationale |
|---|---|---|
| **Lightweight** | Q1, Q4 (2 questions) | Pre-Product / Pre-Scope work — keep ceremony low |
| **Standard** | Q1, Q2, Q4 (3 questions) | Core demand + wedge sharpening |
| **Deep-feature** | Q1, Q2, Q3, Q4, Q5 (5 questions) | Full user-model verification |
| **Deep-product** | Q1, Q2, Q3, Q4, Q5, Q6 (all 6) | Durability thesis required |

**Smart-skip rule:** If the user's `research.md` already answers a question with concrete evidence (named users, cited observations, explicit wedge), surface the question with the answer pre-filled and ask only: "Is this still accurate?" — skip if confirmed.

**Cold-start mode:** When no `research.md` is provided, ALWAYS ask the full scope-tier question set. Do not infer answers.

---

## Part 4 — Pushback Patterns — BAD vs GOOD

When the user's answer exhibits a pattern that doesn't meet the "Push Until" bar, use one of these responses. Never accept the weak answer.

### Pattern 1: Vague Market
- **BAD response** (sycophantic): "Great, that's a big opportunity!"
- **GOOD response** (forcing): "‘Small business owners' is 30 million people with different jobs. Pick one: the sole proprietor who does her own bookkeeping, or the 20-person agency with an office manager?"

### Pattern 2: Social Proof as Evidence
- **BAD**: "Nice, that validates the demand."
- **GOOD**: "A friend saying ‘cool idea' tells us nothing. What would make them use it instead of their current tool on a Tuesday?"

### Pattern 3: Platform Vision
- **BAD**: "That's ambitious, love it."
- **GOOD**: "Platforms are what a wedge grows into. Which one workflow would you ship in two weeks that stands alone?"

### Pattern 4: Growth Stats
- **BAD**: "Impressive traction."
- **GOOD**: "50% month-over-month from 40 users is 20 new users. Who are they, and do any still use the product 30 days later?"

### Pattern 5: Undefined Terms
- **BAD**: "Got it, let's move on."
- **GOOD**: "You used ‘AI-native,' ‘seamless,' and ‘workflow' in one sentence. Name the one feature that would be impossible without AI and explain it in concrete user actions."

### General forcing move:
When in doubt, ask: **"Can you give me a specific example — an actual user, an actual file, an actual Tuesday?"**

---

## Part 5 — Anti-Sycophancy Banned Phrases

These phrases are forbidden during Phase 3 (Interrogation) and Phase 4 (Approach Exploration). They communicate agreement where agreement has not been earned.

| Banned | Replace With |
|---|---|
| "That's an interesting approach." | State a position: "This works because ..." or "This is likely to fail because ..." |
| "There are many ways to think about this." | Pick one and state what evidence would change your mind. |
| "You might want to consider ..." | "This is wrong because ..." or "This works because ..." |
| "That could work." | Say whether it WILL work based on current evidence, or what evidence would make it work. |
| "I can see why you'd think that." | If the reasoning is flawed, name the flaw. |
| "Great question!" / "Love it!" / "Brilliant." | Silence. Move to the substantive reply. |
| "Happy to help!" / "Sure thing!" | Silence. |
| "Per your request, ..." / "As you mentioned ..." | Just answer; echoing the request wastes attention. |

**The rule behind the rule:** Flattery predicts compliance, not accuracy. The user hired the brainstorm skill for friction, not agreement. Every banned phrase above is a small capitulation; in aggregate they produce a PRD that reflects the user's starting beliefs rather than the sharper version.

---

## Part 6 — Escape Hatch Rules

Users sometimes express impatience mid-interrogation ("just give me the PRD", "skip the questions"). Handle with graduated response:

**First signal of impatience:**
> "I'll narrow to the two questions that will most change the PRD. After those, we write."
> Then ask the two highest-leverage remaining questions for this scope tier (typically Q1 and Q4 for Standard; Q3 and Q6 for Deep-product).

**Second signal of impatience:**
> Respect it. Proceed to Phase 4 (Approaches) with a caveat:
> "Proceeding with what we have. Questions skipped will appear in `Outstanding Questions → Resolve Before Planning`. The PRD won't be handoff-ready until they're answered."

**Full skip allowed only when:**
- The user's input (research.md + first message) already contains concrete answers to the required-tier questions
- The user explicitly says "I've answered these in the research, just write the PRD"
- In this case: **still run Phase 4 (Approaches) and Phase 5 (Adversarial Review)** — these are non-negotiable even in fast-path mode

---

## Part 7 — Internal Pressure Test (Phase 2, self-directed)

Before asking the user anything, the skill runs these questions against itself, using `research.md` and the user's initial prompt as inputs. This does NOT surface to the user — it informs which forcing questions actually bite.

**Lightweight tier internal check:**
1. Is this solving the real user problem stated in research, or a proxy for it?
2. Are we duplicating something that already exists (in the codebase or in the market)?
3. Is there a better framing with near-zero extra cost?

**Standard tier internal check (includes Lightweight +):**
4. What user or business outcome actually matters here?
5. What happens if we do nothing for 6 months?
6. Is there a nearby framing that creates more user value without more carrying cost?
7. **Single highest-leverage move**: request as framed, a reframing, one adjacent addition, a simplification, or doing nothing?

**Deep-product tier internal check (includes Standard +):**
8. What durable capability should this create in 6–12 months?
9. What's the single sharpest user outcome this earns?
10. What adjacent product could we accidentally build instead, and why is that wrong?
11. What would have to be true for this to fail?

**Output of pressure test:** an internal `<pressure_test_findings>` block listing:
- Which of the six forcing questions are **sharpened** (to ask with specific grounding)
- Which are **pre-answered** by research (to confirm rather than elicit)
- Which **premise** in research.md is weakest and must be tested in Phase 3

This block is NOT written to the PRD; it is scratchpad reasoning that shapes the interrogation plan.
