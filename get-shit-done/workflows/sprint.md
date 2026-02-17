<purpose>
Sprint orchestrator: run multiple phases end-to-end without manual intervention.

Wraps GSD's existing auto-advance machinery with:
- Phase range selection (all remaining or specified range)
- Failure recovery (gap closure retry, skip-on-failure)
- Sprint report generation

This is a thin orchestrator. It invokes existing phase workflows with `--auto`
and adds monitoring, recovery, and reporting on top.
</purpose>

<required_reading>
Read all files referenced by the invoking prompt's execution_context before starting.
</required_reading>

<process>

<step name="parse_arguments">
Parse $ARGUMENTS for:

- **Range**: e.g., `3-5` → phases 3, 4, 5. Single number `3` → just phase 3. Omit → all remaining.
- **`--dry-run`**: Show plan without executing.
- **`--skip-failures`**: Continue to next phase if current phase fails after retry.
- **`--consolidated`**: Force consolidated workflow for all phases.
- **`--prd <file>`**: Pass PRD file to skip discuss phase.

Store as: `RANGE_START`, `RANGE_END`, `DRY_RUN`, `SKIP_FAILURES`, `USE_CONSOLIDATED`, `PRD_FILE`.
</step>

<step name="determine_phases">
Read ROADMAP.md and STATE.md to determine:

1. Current milestone's phase list (number, name, goal, status)
2. Which phases are incomplete (not marked `[x]`)
3. Current phase from STATE.md

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.cjs init plan-phase "${RANGE_START:-current}" --include state,roadmap)
```

**If range specified:** Filter to phases within `RANGE_START` through `RANGE_END`.
**If no range:** Use all incomplete phases from current phase onward.

Build `SPRINT_PHASES` — an ordered list of phase numbers to execute.

**Validation:**
- Error if no incomplete phases found
- Error if range references non-existent phases
- Error if all phases in range are already complete
- Warn if any phases in range already have partial progress (plans exist but no summaries)

```bash
CONSOLIDATED_CFG=$(node ~/.claude/get-shit-done/bin/gsd-tools.cjs config-get workflow.consolidated 2>/dev/null || echo "false")
```

If `--consolidated` flag or `CONSOLIDATED_CFG` is `"true"`, set `USE_CONSOLIDATED=true`.
</step>

<step name="show_sprint_plan">
Display the sprint plan:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► SPRINT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Phases: {first} → {last} ({count} phases)
Mode: {Standard / Consolidated}
On Failure: {Stop / Skip & Continue}
PRD: {filename / None}

| # | Phase | Status |
|---|-------|--------|
| {N} | {Name} | Pending |
| {N+1} | {Name} | Pending |
| ... | ... | ... |

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**If `--dry-run`:** Display the plan and stop. Do not execute.

```
DRY RUN — no phases will be executed.
To run this sprint: /gsd:sprint {range}
```

Return early.

**If NOT dry-run:** Ask for confirmation before proceeding:

```
AskUserQuestion([
  {
    question: "Start sprint? This will run {count} phases unattended.",
    header: "Sprint",
    multiSelect: false,
    options: [
      { label: "Start Sprint", description: "Run all listed phases end-to-end" },
      { label: "Cancel", description: "Return without executing" }
    ]
  }
])
```

If cancelled, return early.
</step>

<step name="save_and_set_config">
Save current auto_advance value and enable it for the sprint:

```bash
ORIGINAL_AUTO=$(node ~/.claude/get-shit-done/bin/gsd-tools.cjs config-get workflow.auto_advance 2>/dev/null || echo "false")
node ~/.claude/get-shit-done/bin/gsd-tools.cjs config-set workflow.auto_advance true
```

Initialize sprint tracking:

```bash
SPRINT_START_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
```

Create arrays to track results:
- `COMPLETED_PHASES=()` — phases that passed
- `FAILED_PHASES=()` — phases that failed and were skipped
- `BLOCKED_PHASE=""` — phase that blocked the sprint (if stop-on-failure)
</step>

<step name="execute_sprint_loop">
For each phase in `SPRINT_PHASES`:

### Display Phase Banner

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 SPRINT ► Phase {N}/{total}: {Name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Progress: [{completed}✓ {failed}✗ {remaining}…]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Determine Phase Entry Point

Check what state this phase is in:

```bash
PHASE_DIR=$(node ~/.claude/get-shit-done/bin/gsd-tools.cjs phase dir "${PHASE_NUM}")
```

1. **Has VERIFICATION.md with `passed`** → Phase already complete, skip.
2. **Has SUMMARY.md files** → Phase partially executed, resume with execute-phase.
3. **Has PLAN.md files but no SUMMARY.md** → Plans exist, start at execute-phase.
4. **Has CONTEXT.md but no PLAN.md** → Context gathered, start at plan-phase.
5. **Nothing** → Start from beginning (discuss-phase or consolidated-phase).

### Invoke Phase Workflow

Build the invocation command based on entry point and mode:

**If `USE_CONSOLIDATED` and starting from beginning:**
```
Task(
  prompt="Execute /gsd:consolidated-phase {phase_num} --auto" + (PRD_FILE ? " --prd " + PRD_FILE : ""),
  subagent_type="general-purpose",
  description="Sprint: Consolidated Phase {phase_num}"
)
```

**If standard mode, starting from beginning:**
```
Task(
  prompt="Execute /gsd:discuss-phase {phase_num} --auto" + (PRD_FILE ? "" : ""),
  subagent_type="general-purpose",
  description="Sprint: Discuss Phase {phase_num}"
)
```

Note: If `--prd` is set in standard mode, skip discuss and start at plan-phase:
```
Task(
  prompt="Execute /gsd:plan-phase {phase_num} --auto --prd {PRD_FILE}",
  subagent_type="general-purpose",
  description="Sprint: Plan Phase {phase_num}"
)
```

**If resuming from a later entry point:** invoke the appropriate command (plan-phase, execute-phase) with `--auto`.

### Check Phase Result

After the Task returns, check the phase outcome:

```bash
# Check if phase was marked complete
PHASE_STATUS=$(node ~/.claude/get-shit-done/bin/gsd-tools.cjs phase status "${PHASE_NUM}" 2>/dev/null || echo "incomplete")
```

Also check for VERIFICATION.md:
```bash
VERIFICATION_FILE=$(find ".planning/phases/${PHASE_DIR}" -name "*-VERIFICATION.md" -type f 2>/dev/null | head -1)
```

If VERIFICATION.md exists, read its status field.

### Handle Outcome

**If phase completed (status = complete):**
- Add to `COMPLETED_PHASES`
- Continue to next phase

**If phase has gaps (VERIFICATION.md status = gaps_found):**

Attempt gap closure:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 SPRINT ► Gap Closure: Phase {N}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

```
Task(
  prompt="Execute /gsd:plan-phase {phase_num} --gaps --auto",
  subagent_type="general-purpose",
  description="Sprint: Gap Closure Plan Phase {phase_num}"
)
```

Then:
```
Task(
  prompt="Execute /gsd:execute-phase {phase_num} --gaps-only --auto",
  subagent_type="general-purpose",
  description="Sprint: Gap Closure Execute Phase {phase_num}"
)
```

Re-check phase status after gap closure.

**If gap closure succeeded:** Add to `COMPLETED_PHASES`, continue.

**If gap closure failed:**

If `SKIP_FAILURES`:
- Add to `FAILED_PHASES` with reason "gaps unresolved after retry"
- Continue to next phase

If NOT `SKIP_FAILURES`:
- Set `BLOCKED_PHASE` with details
- Break out of loop

**If phase failed for other reasons (agent error, planning failure):**

If `SKIP_FAILURES`:
- Add to `FAILED_PHASES` with reason
- Continue to next phase

If NOT `SKIP_FAILURES`:
- Set `BLOCKED_PHASE` with details
- Break out of loop
</step>

<step name="restore_config">
Restore original auto_advance setting:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.cjs config-set workflow.auto_advance ${ORIGINAL_AUTO}
SPRINT_END_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
```
</step>

<step name="write_sprint_report">
Write `.planning/SPRINT-REPORT.md`:

```markdown
# Sprint Report

**Started:** {SPRINT_START_TIME}
**Ended:** {SPRINT_END_TIME}
**Mode:** {Standard / Consolidated}
**Phases Attempted:** {total attempted}
**Phases Completed:** {completed count}
**Phases Failed:** {failed count}
**Phases Skipped:** {not attempted count, if blocked early}

## Results

| Phase | Name | Status | Notes |
|-------|------|--------|-------|
| {N} | {Name} | ✓ Passed | — |
| {N+1} | {Name} | ✗ Failed | Gaps unresolved after retry |
| {N+2} | {Name} | ⊘ Skipped | Blocked by Phase {N+1} |

## Quality Gates

{If quality gates were configured, summarize pass/fail per phase}

## Completed Phases

{For each completed phase, one-line summary from VERIFICATION.md}

## Failed Phases

{For each failed phase, details of what went wrong and what was attempted}

## Next Steps

{Based on outcome:}
- If all passed: "All phases complete. Run `/gsd:complete-milestone` to archive."
- If some failed: "Failed phases need manual attention. Run `/gsd:progress` to see current state."
- If blocked: "Sprint stopped at Phase {N}. Resolve gaps with `/gsd:plan-phase {N} --gaps` then re-run `/gsd:sprint {remaining_range}`."
```

Commit the report:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.cjs commit "docs: sprint report" --files .planning/SPRINT-REPORT.md
```
</step>

<step name="present_results">
Display final sprint summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► SPRINT COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{completed}✓  {failed}✗  {skipped}⊘

| Phase | Status |
|-------|--------|
| {N}: {Name} | ✓ |
| {N+1}: {Name} | ✗ gaps |
| ... | ... |

Report: .planning/SPRINT-REPORT.md
```

**If all phases completed:**
```
All {count} phases passed.

▶ /gsd:complete-milestone
```

**If some failed with `--skip-failures`:**
```
{completed} of {total} phases passed. {failed} need attention.

▶ /gsd:progress — see what needs fixing
```

**If blocked (no `--skip-failures`):**
```
Sprint stopped at Phase {N}: {Name}

▶ /gsd:plan-phase {N} --gaps — resolve gaps
▶ /gsd:sprint {N}-{last} — resume sprint from Phase {N}
```
</step>

</process>

<success_criteria>
- [ ] Phase range correctly determined from ROADMAP.md and STATE.md
- [ ] Dry-run shows plan without executing
- [ ] User confirms before sprint starts
- [ ] Auto-advance enabled temporarily, restored on completion
- [ ] Each phase invoked with correct entry point (discuss/plan/execute based on existing state)
- [ ] Gap closure attempted automatically on verification failures
- [ ] Failed phases either block sprint or are skipped (based on --skip-failures)
- [ ] SPRINT-REPORT.md written with per-phase results
- [ ] Original auto_advance config restored
- [ ] Sprint does not cross milestone boundaries
</success_criteria>
