# Skill Generation Sub-Path

Reference for the `execute` orchestrator when a plan's PRD requires generating a new skill.

## Table of Contents

1. Decision Summary: Why Option (c)
2. Runtime Assets to Read
3. Writing Agent Prompt Template
4. Gating & Packaging Commands
5. Forbidden Patterns
6. Output Path Handling

## Decision Summary: Why Option (c)

Three options were considered for handling the "generate a new skill" sub-path:

- **(a) Dispatch `task(subagent_type=skill-creator, ...)`**. **REJECTED**. The skill-creator skill is built around a human-in-the-loop interview flow. Its `SKILL.md` at `~/.opencode/skills/skill-creator/SKILL.md` assumes interactive turns on lines 22, 49, 58, 143, 251, 269, and 319 (asking the user questions, sharing test cases for approval, confirming packaging). A non-interactive subagent dispatch will stall waiting for turns that never come.
- **(b) Bypass skill-creator entirely and author `SKILL.md` from scratch**. **REJECTED**. This duplicates the Skill Writing Guide and anatomy rules into `execute`, creating drift every time skill-creator updates upstream. The guidance gets stale and divergent.
- **(c) Read skill-creator's stable assets at runtime, inject them as context into parallel writing subagents, and shell out to skill-creator's non-interactive Python scripts for validation and packaging**. **CHOSEN**. Upstream updates flow through automatically, no human stall, parallel throughput stays intact.

**Rule:** skill-creator is a reference library for this sub-path, NEVER a subagent.

## Runtime Assets to Read

Read these with the `read` tool at runtime. Do NOT snapshot their contents into this file; upstream fixes must flow through.

| Asset | Path | Inject Into |
|---|---|---|
| Skill Writing Guide | `~/.opencode/skills/skill-creator/SKILL.md` lines 64-139 | writing-agent context |
| JSON schemas reference | `~/.opencode/skills/skill-creator/references/schemas.md` | writing-agent context if the plan requires evals |
| Anatomy of a Skill diagram | `~/.opencode/skills/skill-creator/SKILL.md` lines 75-84 | writing-agent context |
| Pushy description guidance | `~/.opencode/skills/skill-creator/SKILL.md` line 67 | writing-agent context |

## Writing Agent Prompt Template

Dispatch one writing agent per target file (the `SKILL.md` body and each `references/*.md` file listed in the plan's U-block). Fire them all in a single assistant turn for parallelism.

```
You are writing ONE section of a new skill.

PRD:
{PRD_CONTENT}

TARGET SKILL PATH: {TARGET_SKILL_PATH}
YOUR ASSIGNED SECTION/FILE: {SECTION}

ANATOMY (from skill-creator SKILL.md lines 75-84):
{ANATOMY_DIAGRAM}

PUSHY DESCRIPTION RULE (from skill-creator SKILL.md line 67):
{PUSHY_DESC_RULE}

SCHEMAS CONTEXT (only if your section touches evals):
{SCHEMAS_CONTEXT}

RULES:
- Produce ONLY the assigned section/file. Do not touch anything else.
- Frontmatter allowed keys: name, description, license, allowed-tools, metadata, compatibility. No others.
- `name` must be kebab-case and match the skill directory name.
- `description` must be third person, ≤1024 chars, no angle brackets, and pushy (tell the model when to trigger, even when not explicitly asked).
- SKILL.md body ≤500 lines.
- Imperative voice; explain WHY for counter-intuitive rules.

Return a `=== VERIFICATION ===` block with post-write checks (line count, frontmatter key scan, description length).
```

## Gating & Packaging Commands

**Mandatory gate** (non-interactive, shell-out). Run from inside skill-creator's
installation directory — assumed to be `~/.opencode/skills/skill-creator/`, but verify it
before running: this repository does not ship that code, and the path points at a
user-writable directory whose contents you are about to execute. If the directory is absent
or the scripts are not the ones you expect, halt the U-block with `NEEDS_CONTEXT` rather
than guessing an alternative path.

```bash
python3 -m scripts.quick_validate {TARGET_SKILL_PATH}
```

Exit 0 → pass. Nonzero → fail the U-block and halt; report validator output to the user.

**Optional packaging**, only if the plan's U-block success criteria mention a `.skill` artifact:

```bash
python3 -m scripts.package_skill {TARGET_SKILL_PATH} [OUTPUT_DIR]
```

DO NOT run `run_loop.py`, `run_eval.py`, `aggregate_benchmark.py`, or `generate_review.py`. Those are post-authoring polish that need human review or prior eval outputs. If the user wants them, suggest a follow-up plan rather than auto-running.

## Forbidden Patterns

- `task(subagent_type="skill-creator", ...)`. This will stall on human turns.
- Inlining skill-creator's Skill Writing Guide into this file as a snapshot. Drift risk.
- Running skill-creator's browser-based eval viewer. Blocks on human interaction.
- Fabricating frontmatter keys beyond `name, description, license, allowed-tools, metadata, compatibility`.
- Dispatching two writing agents against the same section/file. Race condition. One section, one agent.

## Output Path Handling

The target skill path is resolved in this order:

1. If the plan's U-block has explicit `Files:` entries, use those verbatim.
2. Otherwise, if the user passed `output=<path>` when invoking `execute`, use that.
3. If neither source specifies a path, HALT with `NEEDS_CONTEXT` and ask the user for the target skill directory. Do NOT guess.
