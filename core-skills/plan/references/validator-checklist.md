# Validator Checklist

<role>
Walk this checklist EVERY TIME before finalizing a plan file OR before exiting. Each box must be ticked or the plan is rejected. When a check fails, fix before proceeding. This is a hard gate.
</role>

## How to use

1. After drafting the plan (in-memory), walk sections 1-5 in order.
2. For each item, mark `[x]` if satisfied, `[ ]` if not.
3. Any unchecked item in sections 1-4 blocks file emission.
4. Section 5 items can defer to `DONE_WITH_CONCERNS` completion status.

---

## Section 1 — YAML Frontmatter Validity (skill-creator rules)

Applies to: the SKILL.md itself (authoring meta-check, not the output plan file).

- [ ] `name` is present, kebab-case, matches regex `^[a-z0-9-]+$`, ≤64 chars, matches parent directory name
- [ ] `name` contains no leading/trailing `-`, no `--` anywhere, no reserved words (`anthropic`, `claude`)
- [ ] `description` is present, non-empty, ≤1024 characters
- [ ] `description` contains NO angle brackets (`<` or `>`)
- [ ] `description` uses third-person voice (no "I", "you", "we")
- [ ] `description` includes BOTH what-it-does AND when-to-use-it clauses
- [ ] `description` lists explicit trigger phrases (EN + ZH if multilingual)
- [ ] No frontmatter keys exist outside the allowed set: `{name, description, license, allowed-tools, metadata, compatibility}`

## Section 2 — Structure & Progressive Disclosure (skill-creator rules)

Applies to: the SKILL.md body.

- [ ] SKILL.md body ≤500 lines total (count `wc -l`)
- [ ] SKILL.md body ≤~5000 tokens (rough: ~4 chars per token)
- [ ] `references/` directory is exactly ONE level deep from SKILL.md (no `references/nested/x.md`)
- [ ] Reference files >100 lines start with a table of contents
- [ ] All path references use forward slashes `/` (never backslashes `\`)
- [ ] If a `scripts/` dir exists, SKILL.md says whether to EXECUTE or READ each script

## Section 3 — Content & Voice (Anthropic best-practices)

Applies to: SKILL.md and every reference file.

- [ ] Imperative voice in instructions ("Read X", "Write Y", not "You should read X")
- [ ] Explains WHY rules exist (not just "MUST"/"ALL-CAPS")
- [ ] No time-sensitive statements like "Before August 2025" (use "Legacy" subsection instead)
- [ ] No extraneous ancillary docs inside the skill folder (no extra README.md, INSTALL.md)

## Section 4 — Plan Output Schema

Applies to: the plan file the `plan` skill emits.

- [ ] Plan file has YAML frontmatter with keys `title`, `type` (`feat|fix|refactor`), `status: active`, `date` (YYYY-MM-DD), `origin` (PRD path or `inline-spec`)
- [ ] Plan file has all 11 canonical H2 sections in order: Overview, Problem Frame, Requirements Trace, Scope Boundaries, Context & Research, Key Technical Decisions, Open Questions, Implementation Units, System-Wide Impact, Risks & Dependencies, Sources & References
- [ ] Every `U#` block has all 7 canonical sub-fields in order: Goal, Requirements, Dependencies, Files, Approach, Test scenarios, Verification
- [ ] Every `U#` block's `Dependencies:` field is non-empty (either `None`, a U-id list, or `external: ...`)
- [ ] `Implementation Units` section contains a `### Parallel Execution Plan` subsection with at least one wave declared
- [ ] No absolute paths anywhere in the plan file (grep `/Users/`, `C:\`, `~/` returns zero)
- [ ] No implementation code in the plan: no `import` statements, no function bodies, no shell recipes (code fences allowed only for ASCII diagrams or YAML frontmatter)
- [ ] `U#` IDs are stable, if editing an existing plan, no renumbering occurred (gaps are correct)
- [ ] Plan file ends with `## Completion Status` section containing one of: `PLANNED | IN_PROGRESS | DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT`
- [ ] Plan file uses repo-relative paths in all `Files:` entries (e.g., `src/foo.ts` not `/Users/.../src/foo.ts`)

## Section 5 — Soft Concerns (non-blocking, may defer)

- [ ] Description field is "pushy" (combats undertriggering, e.g., `"Use when..."`, `"Triggers:"`)
- [ ] Risks & Dependencies table has at least one RK# entry (even if Low/Low)
- [ ] Open Questions section splits into `Resolved During Planning` + `Deferred to Implementation`
- [ ] Scope Boundaries distinguishes `Out of scope` from `Deferred to Follow-Up Work`
- [ ] Plan's language matches user's input language (English input → English plan; Chinese input → Chinese plan)

---

## Known-Bad Test Samples (for skill self-test)

A correctly-implemented `plan` skill walking this checklist on these samples MUST flag the indicated violations:

### Sample 1 — Absolute path violation

```yaml
---
title: bad-plan
type: feat
status: active
date: 2026-01-01
---

## Implementation Units

- [ ] U1. **Example**
  **Files:** Create `/Users/bob/project/src/foo.ts`
```

Must fail Section 4 check "No absolute paths anywhere".

### Sample 2 — Missing Dependencies field

```
- [ ] U1. **Example**
  **Goal:** do thing
  **Requirements:** [R1]
  **Files:** Create `src/foo.ts`
```

Must fail Section 4 check "Every `U#` block's `Dependencies:` field is non-empty".

### Sample 3 — Nested reference

Directory layout:
```
plan/
  SKILL.md
  references/
    phase1/
      detail.md   ← VIOLATION
```

Must fail Section 2 check "`references/` is exactly ONE level deep".

### Sample 4 — Disallowed frontmatter key

```yaml
---
name: plan
description: ...
version: 1.0.0   ← VIOLATION
---
```

Must fail Section 1 check "No frontmatter keys exist outside the allowed set".
