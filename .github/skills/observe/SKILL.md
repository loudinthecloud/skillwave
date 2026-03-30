---
name: observe
description: "Periodic goal-alignment check — re-read GOAL.md every 5 completed tasks and verify remaining tasks still address the requirements."
---

# Skill: Observe

Periodic goal-alignment checkpoint that prevents drift between the project
goal and the work being executed.

## Trigger

Run the observe step **every 5 tasks marked `done`** (i.e. after task 5, 10,
15, 20, …). Count only tasks completed in the current session. The counter
resets at session start.

## Procedure

1. **Re-read `GOAL.md`** — treat it as the authoritative requirements list.
2. **Read `tasks/BACKLOG.md`** — collect all tasks with status `open` or
   `active`.
3. **Gap analysis** — for each requirement or objective in `GOAL.md`, check
   whether at least one remaining task addresses it. Flag:
   - **Uncovered requirements** — goals with no matching open/active task.
   - **Orphan tasks** — open tasks that don't map to any current goal
     (scope creep or stale work).
   - **Priority misalignment** — high-priority goals whose tasks are low on
     the backlog or blocked without a clear path to unblocking.
4. **Decide and act:**
   - If gaps are found → create new tasks, reprioritize, or retire orphan
     tasks on the board. Log the adjustment in the Decision Log
     (`.github/roles/README.md`).
   - If alignment is good → note the checkpoint in the session log and
     continue.
5. **Resume the execution loop** — do not pause for user confirmation unless
   the gap analysis reveals a fundamental contradiction between `GOAL.md` and
   the current plan (e.g. a major requirement is completely unaddressed). In
   that case, add a question to `tasks/INBOX.md` before continuing.

## Output

After each observe step, append a brief entry to the board or session context:

```
> [Observe @ TASK-XXX] Goal alignment ✓ — no gaps found.
```

or

```
> [Observe @ TASK-XXX] Goal alignment — created TASK-YYY (uncovered: <requirement>), retired TASK-ZZZ (orphan).
```

## Rules

- The observe step is **read-only analysis + board writes**. It does NOT
  invoke subagents.
- Keep it lightweight — spend no more than a few minutes on the check.
- Do not skip or defer the observe step. If the 5-task boundary falls in the
  middle of a parallel batch, run observe after the batch completes.
- If `GOAL.md` has changed since the last observe step, treat the new version
  as authoritative and realign the backlog accordingly.
