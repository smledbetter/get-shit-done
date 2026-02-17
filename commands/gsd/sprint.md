---
name: gsd:sprint
description: Run all remaining phases (or a range) unattended — chained discuss → plan → execute → verify
argument-hint: "[range] [--dry-run] [--skip-failures] [--consolidated] [--prd <file>]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - TodoWrite
  - AskUserQuestion
---
<objective>
Sprint: run multiple phases end-to-end without manual intervention.

Wraps GSD's existing auto-advance machinery in a milestone-level orchestrator that:
1. Determines which phases to run (all remaining or a specified range)
2. Chains phases via auto-advance (discuss → plan → execute → transition)
3. Handles failures — retry gap closure or skip and continue
4. Produces a SPRINT-REPORT.md at the end

**This is not a new workflow.** It's an orchestrator that invokes the existing phase workflows with `--auto` and adds failure recovery + reporting on top.
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/sprint.md
@~/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
Arguments: $ARGUMENTS

**Flags:**
- `[range]` — Phase range like `3-5` or single phase `3`. Omit for all remaining phases.
- `--dry-run` — Show what would run without executing
- `--skip-failures` — Log blocked phases and continue to next (default: stop on first failure)
- `--consolidated` — Use consolidated workflow for each phase (overrides config)
- `--prd <file>` — Pass PRD to skip discuss phase for all phases in the sprint

@.planning/ROADMAP.md
@.planning/STATE.md
</context>

<process>
Execute the sprint workflow from @~/.claude/get-shit-done/workflows/sprint.md end-to-end.
</process>
