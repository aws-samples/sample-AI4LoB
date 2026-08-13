# Plan Parser Reference

This document tells the `execute` skill how to parse a plan file produced by the `plan` skill. Every rule is load-bearing: drift here means the orchestrator mis-dispatches work or corrupts the plan it's supposed to advance.

## Table of Contents

1. Plan File Location Resolution
2. U-Block 7-Field Schema
3. Dependencies Field, Allowed Values
4. Same-Wave Files Overlap Detection
5. `[category]` Hint Extraction
6. Completion Status Field
7. Halt Condition Summary

## Plan File Location Resolution

Resolve the plan file from one of three inputs, in priority order:

1. **Explicit path parameter.** If the caller passes a path, use it verbatim.
2. **Session-context scan.** If no path is given, scan the current session's working directory and any paths mentioned in recent user messages for files matching the plan signature.
3. **User-prompted fallback.** If the scan yields nothing, ask the user for the path.

Define the **plan signature** as a file that satisfies ALL of: YAML frontmatter containing `status: active`; body contains `## Implementation Units`; body contains at least one heading matching `#### U\d+:`.

Selection: 0 candidates or >1 candidates, ASK the user; exactly 1 candidate, proceed. Silent selection across multiple active plans executes against the wrong unit set.

## U-Block 7-Field Schema

Match the canonical U-block heading with:

```
^#### U(\d+):\s+(.+)$
```

Match each of the 7 canonical fields with:

```
^-\s+\*\*(Goal|Requirements|Dependencies|Files|Approach|Test scenarios|Verification)\*\*:\s*(.*)$
```

Field order is significant. The fields MUST appear in the sequence: Goal, Requirements, Dependencies, Files, Approach, Test scenarios, Verification. Drift in order, missing fields, or renamed fields mean parse failure, HALT `NEEDS_CONTEXT`.

Why order matters: downstream tooling indexes fields positionally for schema validation and diffing.

**Valid example:**

```
#### U3: Write parser reference
- **Goal**: Document the parse contract.
- **Requirements**: Covers 7-field schema and halts.
- **Dependencies**: U1, U2
- **Files**: references/plan-parser.md
- **Approach**: Draft, verify, trim.
- **Test scenarios**: wc -l ≤ 150; 8 H2 sections present.
- **Verification**: Run the verification command block.
- **[category]**: writing
```

**Malformed example (REJECT):** a block where `- **Files**:` appears before `- **Requirements**:` violates field order and MUST be rejected.

## Dependencies Field, Allowed Values

Permit three forms for the Dependencies value:

- `None`, match `^None$`
- Comma-separated U-ID list, match `^U\d+(,\s*U\d+)*$`, e.g. `U1, U2, U4`
- External dependency, match `^external:\s*.+$`, e.g. `external: waiting on design review`

Enforce two invariants:

- **Dangling reference.** If any referenced `U\d+` does not correspond to a U-block heading present in the same plan file, HALT `NEEDS_CONTEXT`. Reason: the executor cannot dispatch a wave that depends on a unit that doesn't exist.
- **Cycle.** If the dependency graph contains a cycle, HALT `NEEDS_CONTEXT`. Example: `U1` depends on `U2`, `U2` depends on `U1`. Topological sort is impossible, so no valid wave ordering exists.

## Same-Wave Files Overlap Detection

This resolves the deferred question Q9 in the plan file: how does the executor protect itself from two parallel agents writing the same path?

Normalize every token in each U-block's `Files:` value:

1. Strip any parenthesized annotations in ASCII `(...)` or CJK `（...）` form before splitting. Example: `src/foo.ts（create，仅 header）, src/bar.ts` becomes `src/foo.ts, src/bar.ts`. Annotations frequently contain embedded commas that confuse a naive split.
2. If the resulting value (after annotation strip and whitespace trim) matches `None | N/A | 无 | 无文件 | 无新文件`, treat the Files list as EMPTY and skip this U-block in collision detection.
3. Otherwise split on comma.
4. Trim surrounding whitespace on each token.
5. Normalize path separators to `/`.
6. Drop any leading `./`.

Two files conflict when the normalized strings compare byte-equal. Comparison is case-sensitive (Linux filesystems are case-sensitive, don't silently collide on case-insensitive hosts).

Within a single wave, the UNION of normalized Files tokens across all units in that wave MUST contain no duplicates. Any overlap, HALT `NEEDS_CONTEXT`. Reason: two parallel agents writing the same file race each other and corrupt the output.

**Exception:** the plan file itself is updated by `execute` when writing Completion Status, it never appears as a U-block `Files:` entry and is exempt from this check.

## `[category]` Hint Extraction

Each U-block SHOULD declare a category hint AFTER the 7 canonical fields, on its own line:

```
- **[category]**: <name>
```

Before matching against the recognized set, NORMALIZE the value:

1. Strip any leading/trailing whitespace.
2. Strip surrounding backticks if present (human authors often write `` `quick` `` per markdown convention).
3. Strip any trailing parenthesized annotation in either ASCII `(...)` or CJK `（...）` form. Example: `` `oracle`（特殊：非 category 而是 subagent_type）`` normalizes to `oracle`.
4. Lowercase.

Recognized category values: `quick`, `writing`, `unspecified-low`, `unspecified-high`, `visual-engineering`, `ultrabrain`, `deep`, `artistry`. Recognized subagent_type overrides: `explore`, `librarian`, `oracle`.

Dispatch rules (using the `task()` tool; note the tool always uses `subagent_type` or `category` parameters, not `category=` as a string):

- If the normalized hint is a recognized category value, dispatch with `category=<name>`.
- If the normalized hint is `oracle`, dispatch with `subagent_type="oracle"` AND `run_in_background=false`. Never pass `category=` alongside the oracle override.
- If the normalized hint is `explore` or `librarian`, dispatch with the matching `subagent_type`.
- If the normalized hint is unrecognized (not in either set), HALT `NEEDS_CONTEXT` with the unknown value.
- If the `[category]` line is missing, default to `unspecified-high`. Reason: unknown work deserves the stronger generalist, not the cheap one.

## Completion Status Field

Quote the 6-value vocabulary defined in the `plan` skill's `SKILL.md` verbatim:

```
PLANNED | IN_PROGRESS | DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
```

Match the status footer with either of:

```
^`?(PLANNED|IN_PROGRESS|DONE|DONE_WITH_CONCERNS|BLOCKED|NEEDS_CONTEXT)`?$
^Completion Status:\s*(PLANNED|IN_PROGRESS|DONE|DONE_WITH_CONCERNS|BLOCKED|NEEDS_CONTEXT)$
```

Take the LAST matching line in the file as the authoritative status line.

Update the status via the `edit` tool using literal string replacement. Do NOT rewrite the whole file, full rewrites clobber concurrent edits to other sections.

If more than one candidate status line matches, HALT `NEEDS_CONTEXT`. Reason: ambiguous status means the executor can't tell which line to advance.

## Halt Condition Summary

| Condition | Resulting Status | Reason |
|---|---|---|
| Zero U-blocks parsed | NEEDS_CONTEXT | Nothing to execute; plan is malformed or empty. |
| Malformed U-block (field missing or out of order) | NEEDS_CONTEXT | Schema drift breaks positional indexing downstream. |
| Dependencies graph contains a cycle | NEEDS_CONTEXT | No valid wave ordering exists. |
| Dependencies reference a nonexistent U-ID | NEEDS_CONTEXT | Can't dispatch against a missing unit. |
| Same-wave Files overlap | NEEDS_CONTEXT | Parallel writers would corrupt the shared path. |
| Multiple Completion Status lines found | NEEDS_CONTEXT | Ambiguous footer; can't pick authoritative status. |
| Completion Status line not found | NEEDS_CONTEXT | No footer to update after execution. |
