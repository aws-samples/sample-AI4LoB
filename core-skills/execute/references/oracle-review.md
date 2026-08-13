# Oracle Review

## Purpose

Oracle is the single quality gate between "all U-blocks reported success" and "write final Completion Status to plan file". U-block `Verification:` commands catch byte-count and exit-code regressions but miss semantic failures (right size, wrong content). Oracle spot-checks artifacts against plan Success Criteria and judges whether the Proposed Completion Status is honest.

## Invocation

Dispatch exactly once, in the foreground, at end of the execute run:

```
task(
  subagent_type="oracle",
  run_in_background=false,
  description="Final review of execute run",
  prompt=<filled Oracle prompt template below>
)
```

Rule: exactly ONE Oracle call per execute invocation. NEVER call Oracle per U-block; per-block Oracle defeats the single-gate design and burns tokens.

## Oracle Prompt Template

Fill and send verbatim:

```
# Context
Plan: {plan_file_path}
Invocation reason: {one-line reason execute was run}
## Plan Success Criteria
{verbatim paste of plan's "Success Criteria" / "System-Wide Impact" checklist}
## Artifact Manifest
- {path}: {bytes} bytes; preview: {first 3 lines}
## U-Block Verification Summary
U1: exit_code=0, summary="<agent's verification summary>"
U2: exit_code=0, summary="..."
## Proposed Completion Status
{DONE | DONE_WITH_CONCERNS | BLOCKED}
## Review Instructions
- Does the artifact actually satisfy the plan's Success Criteria? Spot-check at least one nontrivial file's content.
- Is the Proposed Completion Status justified by the verification summary? Flag any U-block that reported success but produced output inconsistent with its Goal.
- Identify silent failures: type errors suppressed with `as any`, empty catch blocks, tests deleted to pass, etc.
- Return structured verdict below.
```

## Required Oracle Output Schema

```xml
<review_verdict>
  <verdict>PASS | FAIL</verdict>
  <proposed_status_accepted>true | false</proposed_status_accepted>
  <recommended_status>DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT</recommended_status>
  <reasons>
    <reason severity="HIGH | MEDIUM | LOW">concrete concern, one per tag</reason>
  </reasons>
  <concerns_to_record>
    <concern>human-readable note to paste into plan's Execution Log</concern>
  </concerns_to_record>
</review_verdict>
```

## Execute's Parsing Rules

- Malformed or missing `<verdict>`: treat as FAIL; write `DONE_WITH_CONCERNS` with concern "Oracle verdict unparseable".
- `PASS` + `proposed_status_accepted=true`: write Proposed Completion Status as-is.
- `PASS` + `proposed_status_accepted=false`: use `<recommended_status>`; record reasons in Execution Log.
- `FAIL`: downgrade to `<recommended_status>` (typically `DONE_WITH_CONCERNS`); paste all `<reason severity="HIGH">` items into Execution Log.
- `<recommended_status>` is `BLOCKED` or `NEEDS_CONTEXT`: do NOT mark artifact shipped. Surface escalation in plan skill's canonical 4-line format (STATUS / REASON / ATTEMPTED / RECOMMENDATION).

## Worked Example, PASS Case

U1-U3 reported `exit_code=0`; Proposed Status: `DONE`. Oracle spot-checks `SKILL.md` (320 lines, within 400-line cap), confirms reference files exist. Returns:

```xml
<review_verdict>
  <verdict>PASS</verdict>
  <proposed_status_accepted>true</proposed_status_accepted>
  <recommended_status>DONE</recommended_status>
  <reasons></reasons><concerns_to_record></concerns_to_record>
</review_verdict>
```
Execute writes `DONE`.

## Worked Example, FAIL Case

U1-U3 reported success. Oracle opens `SKILL.md`, counts 523 lines; plan declared a 400-line cap. Returns:

```xml
<review_verdict>
  <verdict>FAIL</verdict>
  <proposed_status_accepted>false</proposed_status_accepted>
  <recommended_status>DONE_WITH_CONCERNS</recommended_status>
  <reasons><reason severity="HIGH">SKILL.md exceeds declared 400-line cap (523 lines)</reason></reasons>
  <concerns_to_record><concern>SKILL.md is 523 lines; plan required ≤400. Trim before next release.</concern></concerns_to_record>
</review_verdict>
```
Execute writes `DONE_WITH_CONCERNS` and appends the concern to the Execution Log.
