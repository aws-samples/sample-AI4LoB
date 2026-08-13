# AI-PLC Practices

**English** ｜ [🇨🇳 中文](README.zh-CN.md)

A skill collection for **Amazon Quick desktop** that turns the full product
lifecycle — research → product definition → engineering planning → delivery —
into a reusable pipeline.

> **This is sample code, for non-production usage.** You should work with your
> security and legal teams to meet your organizational security, regulatory and
> compliance requirements before deployment. Artifacts produced by these skills
> (research reports, PRDs, plans, HTML prototypes) are drafts for review, not
> deliverables to ship as-is.

> **⚠️ Your input leaves your machine.** `DeepResearch` and `AIPLC` send the
> research topic — and the search queries derived from it — to third-party
> search and inference providers. **Do not use confidential information as a
> research topic**: unreleased product or launch plans, internal metrics,
> customer names or other customer-identifying details, credentials, or anything
> your organization classifies as confidential. Restate the topic in
> non-confidential terms first. Paraphrasing does not help — a paraphrase is
> still transmitted.

---

## Contents

| Directory | Description |
| --- | --- |
| [`core-skills/`](core-skills/README.md) | The five core skills forming the AI-PLC pipeline |
| [`extend-skills/`](extend-skills/README.md) | Home for extension skills |

---

## 1. What this is

Five skills form an **AI Product Lifecycle (AI-PLC) pipeline**. The first four
are functional skills; the fifth (AIPLC) is the orchestrator that chains them.

```
topic
    │
    ▼
┌─────────────────┐  research.md
│  DeepResearch   │───────────────┐
└─────────────────┘               ▼
                       ┌─────────────────┐  PRD (R#/A#/F#/AE#)
                       │   brainstorm    │───────────────┐
                       └─────────────────┘               ▼
                                            ┌─────────────────┐  plan.md (U# + DAG + waves)
                                            │      plan       │───────────────┐
                                            └─────────────────┘               ▼
                                                                   ┌─────────────────┐
                                                                   │     execute     │──▶ artifacts
                                                                   └─────────────────┘    (code/docs/skills)
     ▲                        ▲                      ▲                      ▲
     └────────────────────────┴──────────────────────┴──────────────────────┘
                  AIPLC (orchestration, user confirmation gate between steps)
```

### 1.1 DeepResearch — multi-agent deep research orchestrator

Decomposes a research topic into multiple angles, dispatches 5-8 research agents
in parallel (web search, doc lookup, GitHub code search), cross-validates the
findings into a consensus matrix, then applies Socratic questioning to challenge
assumptions and expose gaps. Produces a markdown report with confidence-graded
findings, open questions, and a source appendix.

This is a **read-only** skill — it edits no code. Its core discipline is "NEVER
guess. NEVER fabricate.": every factual claim must trace to a source URL; where
evidence is absent it says "No evidence found" rather than inventing one.

- **Input** — a research topic in natural language
- **Output** — `deepresearch-{topic-slug}-{YYYY-MM-DD}.md`
- **Triggers** — `deep research`, `research [topic]`, `deepresearch`

### 1.2 brainstorm — research report to PRD converter

Acts as a Product-Lead-in-residence. Interrogates before it synthesizes: one
question at a time, options before recommendations, then a pressure test and
adversarial review, producing a PRD with stable IDs (`R#` requirements /
`A#` actors / `F#` flows / `AE#` acceptance examples).

It **never writes code**. In its own words: "I am the user's thinking partner,
not their yes-person. Flattery is forbidden. Friction is the service." Every
requirement must trace back to a specific passage of the research report or to
the user's verbatim answer during interrogation.

- **Input** — path to a `deepresearch*.md` (optional; without it, cold-start mode
  treats your idea description as the research input)
- **Output** — `{slug}-prd-{YYYY-MM-DD}.md` plus a spoken handoff menu
- **Triggers** — `brainstorm`, `turn research into PRD`, `write PRD`

### 1.3 plan — PRD to DAG implementation planner

Acts as a Staff-Engineer-in-residence, converting a PRD into a DAG-shaped
implementation plan: `U#` units with dependencies, parallel execution waves,
touched files, acceptance criteria, and category hints. The better the PRD, the
fewer clarifying questions it asks (0-8).

**Planning only** — no code, no builds, no invoking execute. Every `R#` reference
in the plan must correspond to an ID that actually exists in the PRD; a missing
ID blocks handoff.

- **Input** — path to a PRD carrying stable IDs (optional; cold-start otherwise)
- **Output** — `docs/plans/YYYY-MM-DD-NNN-<type>-<slug>-plan.md` when that
  directory exists, else `./{slug}-plan-{YYYY-MM-DD}.md`
- **Triggers** — `plan`, `implementation plan`, and Chinese equivalents
  (`计划`, `做计划`, `实施计划`, `根据 PRD 做计划`)

### 1.4 execute — wave-parallel plan executor

A pure delegator. Dispatches category-specialist subagents in parallel following
the plan's DAG waves, collects structured verification results, retries a failure
once, transitively skips dependents on hard failure while other branches
continue, then runs a single consolidated Oracle adversarial review and writes
completion status plus an execution log back into the plan file.

It **never authors artifact code itself** — the sole exception is editing the
plan's status line and execution log. When the plan is malformed (cyclic DAG,
dangling dependency, same-wave file conflict, unknown category), it aborts with
`NEEDS_CONTEXT` and dispatches nothing; fabricating a substitute is forbidden.

Largely hands-off while running ("No mid-run questions").

- **Input** — path to the plan file produced by `plan`
- **Output** — the same plan file with status written back, plus the artifacts
  produced by subagents
- **Triggers** — `execute`, `run plan`, and Chinese equivalents
  (`执行计划`, `施工`, `实施`, `开干`, `按计划执行`)

### 1.5 AIPLC — four-step pipeline orchestrator

Chains the four skills into a `deepresearch → brainstorm → plan → execute`
pipeline that **pauses for user confirmation after every step**. You can request
changes; the agent reviews and responds, iterating until you are satisfied before
moving on.

Compared with using deepresearch alone, AIPLC adds the Amazon PR/FAQ and
One-way / Two-way Door decision frameworks, and can optionally pull in the AWS
Documentation MCP server when the topic concerns a specific AWS service.

- **Input** — one required parameter, `topic`
- **Declared dependency** — `depends-on: [deepresearch, brainstorm, plan, execute]`
- **Triggers** — `aiplc <topic>`, `AI-PLC <topic>`

---

## 2. Design principles

What separates these skills from ordinary prompt templates:

| Principle | Description |
| --- | --- |
| **Hard gate** | Each skill produces exactly one kind of thing. DeepResearch is read-only, brainstorm only emits a PRD, plan only emits a plan, execute only delegates. This makes outputs predictable and each stage independently replaceable. |
| **No fabrication** | Every claim traces to a source: a research passage, a verbatim user answer, or an ID that genuinely exists upstream. |
| **Stable IDs forever** | `R#/A#/F#/AE#/U#` are never recycled once assigned. Gaps left by deleted entries are correct. Splitting `R3` yields `R3a`/`R3b` or the next unused integer — never a reused ID. |
| **Six-value status** | `PLANNED` / `IN_PROGRESS` / `DONE` / `DONE_WITH_CONCERNS` / `BLOCKED` / `NEEDS_CONTEXT`. No other value is legal. Escalations use a fixed four-line format: STATUS / REASON / ATTEMPTED / RECOMMENDATION. |
| **Bidirectional gates** | Data does not only flow forward. Unresolved blocking questions in the PRD stop planning; a failing plan checklist stops execution. |
| **Lazy-load references** | Files under `references/` load only at the phase that needs them — "context budget is finite". Preloading them all at session start is an explicit anti-pattern. |
| **Adversarial review** | An oracle subagent attacks the work at key checkpoints, actively hunting for holes instead of rubber-stamping conclusions. |
| **Anti-sycophancy** | Flattery is explicitly forbidden. Friction is part of the service. |
| **Parallel first** | Research agents, writing agents, and execution waves run in parallel wherever safe. Two agents writing the same file is forbidden — that is a race condition. |

### Traceability chain

Auditability rests on an unbroken chain of ID references:

```
research finding & citation
  → R# in the PRD
    → Requirements Trace in the plan (every R# must appear; a missing ID blocks)
      → Requirements field of each U-block
        → subagent VERIFICATION block (command, exit code, output)
          → Artifacts list and Oracle verdict in the Execution Log
```

This is why "never renumber" is a CRITICAL rule in all four skills: renumbering
any link in the chain invalidates every downstream reference.

---

## 3. Installing on Amazon Quick desktop

> Official docs:
> <https://docs.aws.amazon.com/quick/latest/userguide/skills-and-agents-desktop.html>

### 3.1 Prerequisites

- Amazon Quick desktop installed and signed in
- Skill files obtained from `core-skills/` in this repository

### 3.2 Method A — add the folder, let the agent install (recommended)

This is the field-tested path used in the workshop. It installs all five
skills in one go and avoids attaching reference files by hand.

1. Get the skills by cloning this repository:

   ```bash
   git clone https://github.com/aws-samples/sample-ai-plc-practices.git
   ```

   The extracted skill folders are already under `core-skills/` — no unzipping
   needed. If you took the `.zip` files instead, unzip them into one folder first.

2. In Amazon Quick, open **settings** → **My computer**.

3. Choose **Add Folder** and add the folder holding the skills (e.g.
   `sample-ai-plc-practices/core-skills`). Confirm the folder shows as enabled.

4. Choose **New Chat** and prompt the agent to install them, for example:

   ```
   请帮我安装：<目录>下的 5 个 skill
   Please install the 5 skills under <directory>
   ```

5. Wait for confirmation, then check the Skills tab — the installed skills appear
   there.

Because the agent performs the installation, it picks up `references/`
subdirectories on its own and tolerates either instruction-file naming convention
(`SKILL.md` or `<skill-name>.md`).

### 3.3 Method B — Import from file

The path described in the official documentation. Use it to install a single skill.

1. Open Amazon Quick desktop, choose **Agents & skills** in the left navigation,
   then open the **Skills** tab.

2. Choose **+ Create**, then **Import from file**.

3. Select that skill's instruction file (`SKILL.md`, or `<skill-name>.md` for the
   skills under `extend-skills/`).

4. Review and edit as needed, then save. The skill appears under **MY SKILLS**.

5. **Attach reference files — required for brainstorm / plan / execute.** Each of
   these three ships 3 files under `references/`. A Quick skill is fundamentally
   a folder and supports attached reference files: after importing the
   instruction file, open the skill's detail view and attach the files from
   `references/`. Skip this and the skill will fail at the phase that lazy-loads
   them. Method A does this for you.

### 3.4 Install order and verification

**Install in dependency order.** AIPLC declares
`depends-on: [deepresearch, brainstorm, plan, execute]`. Install the four
functional skills first, otherwise AIPLC cannot hand off at the corresponding
step:

`DeepResearch` → `brainstorm` → `plan` → `execute` → `AIPLC`

**Verify.** Find the skill in the Skills tab and choose **Run** to open a
conversation with it preloaded, or simply say "use the DeepResearch skill" in chat.

### 3.5 Kiro CLI

The skills also run under Kiro CLI, where a skill is a directory beneath
`.kiro/skills/`. Place the instruction file there and it loads automatically:

```bash
mkdir -p .kiro/skills/deepresearch
cp core-skills/DeepResearch/SKILL.md .kiro/skills/deepresearch/
```

Confirm it is loaded with `/context` inside a chat session. `.kiro/skills/` is
workspace-scoped, so the skill is available in that directory; use
`~/.kiro/skills/` to make it available everywhere.

`DeepResearch` has been exercised this way; the other four skills have only been
run on Amazon Quick desktop.

### 3.6 Updating an installed skill

Quick desktop has no re-import-and-overwrite action. To update, select the skill,
choose **Edit** in its detail view, and replace the content with the new
`SKILL.md` from this repository; or delete the old skill and re-import.
Recording the commit SHA you installed from makes later comparison easier.

---

## 4. Usage

### 4.1 Full pipeline (recommended entry point)

```
aiplc <topic>
```

Chains all four steps with a confirmation gate between each — the full journey
from an idea to a deliverable artifact.

### 4.2 Using skills individually

| Scenario | Invocation | Notes |
| --- | --- | --- |
| Just want a rigorous research report | `deep research: <topic>` | Report with consensus matrix and Socratic analysis |
| Have research, need a PRD | `brainstorm <path to research.md>` | Interrogate → approaches → adversarial review → PRD |
| No research, starting from an idea | `brainstorm <idea description>` | Cold start: no phase skipped, full question set |
| Have a PRD, need a plan | `plan <path to PRD>` | 0-4 gap questions, then the DAG plan |
| No PRD, plan anyway | `plan` (cold start) | Up to 8 questions, then the plan |
| Have a plan, execute it | `execute <path to plan>` | Wave-parallel dispatch, status written back |

### 4.3 How much you are involved

| Skill | Involvement |
| --- | --- |
| **DeepResearch** | Give the topic; the rest is automatic. DISPUTED items and open questions at the end of the report need your judgement. |
| **brainstorm** | **Highest.** Answer 2-6 mandatory questions and expect pushback. Decide on approach selection, review findings, and the handoff menu. There is an escape hatch if you are impatient, but approach exploration and adversarial review cannot be waived. |
| **plan** | 0-8 questions — the better the PRD, the fewer. Only close-call, one-way-door technical decisions come to you. |
| **execute** | Largely hands-off. Surfaces only on a `NEEDS_CONTEXT` abort or the final report. |
| **AIPLC** | Every gate between steps needs your sign-off; the research report can be iterated indefinitely. |

### 4.4 When an intermediate artifact breaks

- **Cyclic DAG / dangling dependency / same-wave file conflict** — execute aborts
  with `NEEDS_CONTEXT` and names the problem; re-run `plan` to fix it.
- **Unresolved blocking questions in the PRD** — brainstorm closes the gate;
  answering them triggers an automatic re-review loop.
- **A unit fails** — automatic retry once; if it still fails, dependents are
  transitively skipped while other branches continue. Final status is
  `DONE_WITH_CONCERNS` or `BLOCKED` with the four-line escalation format, and you
  decide whether to re-run that part.

---

## 5. Why both zip and extracted sources

Each skill under `core-skills/` ships in **two** forms:

```
core-skills/
├── brainstorm.zip        ← for installing: download, unzip, import into Quick desktop
└── brainstorm/           ← for collaborating: diffable, reviewable, searchable in GitHub
    ├── SKILL.md
    └── references/
```

| Purpose | Which to use |
| --- | --- |
| Installing into Quick desktop | the `.zip`, or the `SKILL.md` inside the extracted directory |
| Reading, reviewing, improving | the extracted directory |

**Why.** A zip is an opaque binary to Git: change one word and `git diff` only
reports that the file changed. No code review, no full-text search. The extracted
sources fix that, while the zip keeps the download-and-go convenience for
colleagues who just want to install.

### ⚠️ Keep the two in sync

Edit the extracted sources first — they are authoritative — then repackage:

```bash
cd core-skills

# 1. edit the sources
vim brainstorm/SKILL.md

# 2. repackage; -r to include references/
rm brainstorm.zip
cd brainstorm && zip -r ../brainstorm.zip . -x '.*' && cd ..

# 3. verify contents
unzip -l brainstorm.zip
```

Commit both sides together and say what changed in the commit message. Updating
only one side ships a stale version to whoever downloads the zip.

---

## 6. Runtime dependencies

These skills rely on capabilities provided by the runtime; porting to another
agent platform requires adaptation.

1. **Subagent dispatch** — the whole system depends on a `task()` primitive
   supporting both `subagent_type` addressing (`oracle` / `explore` /
   `librarian` / `skill-creator`) and `category` addressing (`quick` / `writing`
   / `unspecified-low` / `unspecified-high` / `visual-engineering` /
   `ultrabrain` / `deep` / `artistry`), plus background execution, session
   resumption, and completion notification.
2. **Research tools** — DeepResearch names web search, web fetch, library doc
   lookup, and GitHub code search by tool.
3. **AWS Documentation MCP server** (optional) — AIPLC can consult the AWS
   Documentation MCP server when the topic concerns a specific AWS service. If
   the connector is absent, the skill skips that source and notes the omission
   in the report.
4. **Session interaction** — AIPLC uses session tabs to display files and a
   decision-card syntax for choices.
5. **skill-creator validation scripts** (only when a plan generates a skill) —
   `execute` shells out to `python3 -m scripts.quick_validate` from a
   skill-creator installation, assumed at `~/.opencode/skills/skill-creator/`.
   That path is an assumption about the host environment, not something this
   repository ships; adjust it to wherever skill-creator lives on your machine,
   and be aware you are executing code from a user-writable directory. If
   skill-creator is absent, skip skill-generation units.

---

## 7. Further reading

| File | Content |
| --- | --- |
| [`extend-skills/`](extend-skills/) | Extension skills, including `protoforge` (natural language → interactive HTML prototypes) and `research-to-prd` |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | How to add skills |

---

## 8. In one sentence

This system splits the product lifecycle — research, product definition,
engineering planning, delivery — into four skills with strict boundaries, each
producing a single artifact and referencing the others through stable IDs.
Parallel subagents provide speed, Socratic interrogation and Oracle adversarial
review provide quality, gates and a six-value state machine stop half-finished
work from leaking downstream, and AIPLC orchestrates the whole thing into an
automated pipeline running from a topic phrase to a shippable artifact.

---

## Maintenance

Please open an issue or pull request for questions and suggestions.

---

## License

This project is licensed under the MIT-0 License. See [`LICENSE`](LICENSE) for details.
