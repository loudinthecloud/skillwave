---
name: task-management
description: "Manage tasks on the shared task board — read, create, update, assign, and close tasks following the team's conventions."
---

# Skill: Task Management

How skillwave manages the shared task board. Roles do not edit the board
directly — skillwave handles all board operations.

## Task Format

```markdown
### TASK-XXX: [Title]

- **Status:** open | active | blocked | done
- **Assigned to:** [role-name]
- **Priority:** P0 | P1 | P2 | P3
- **Description:** What needs to be done (2-3 sentences max)
- **Acceptance:** What "done" looks like (bullet list)
- **Depends on:** TASK-YYY (optional)
- **Result:** 1-3 line summary filled when done (optional report link for milestones)
```

## Task Numbering

- Tasks are numbered sequentially: TASK-001, TASK-002, …
- Never reuse a task number. Always 3-digit zero-padding.

## Board Hygiene

The board is the only source of truth. Keep it current at all times:

- **Before starting a task:** set status to `active` in the file — do not just mentally note it
- **After completing a task:** compact it immediately — do not batch multiple completions
- **Never skip a board write** to save steps. If the file wasn't edited, the state change didn't happen.
- Update `> Last updated:` at the top of the file on every write.

## Reading the Board

1. Open `tasks/BACKLOG.md`
2. Open tasks are at the top, ordered by priority
3. Completed tasks are below the `--- DONE ---` divider
4. Check dependencies before starting a task

## Creating Tasks

1. Find the highest task number on the board and increment by 1
2. Use the Task Format above
3. Place in priority order among open tasks
4. Assign to the appropriate role (must exist in the Role Registry in `.github/roles/README.md`)

## Updating Tasks

- **Starting work:** Change status to `active`
- **Blocked:** Change status to `blocked`, note what's blocking it
- **Done:** Change status to `done`, add 1-3 line Result summary, then compact (see below)

## Compacting Completed Tasks

When a task is marked done, **two edits** must happen in `tasks/BACKLOG.md`:

1. **Delete** the full task block (header + all bullet fields) from the
   `## Open Tasks` section.
2. **Append** a single compact line to the `## Completed` section
   (below the `--- DONE ---` divider):

```markdown
- **TASK-XXX:** [Title] — [1-line result summary] | tasks/done/TASK-XXX.md
```

Both edits happen in one board write — never leave a done task in the Open
section. If the `## Completed` section or `--- DONE ---` divider does not
exist yet, create it at the bottom of the file before appending.

This keeps the board small. The full result is preserved in `tasks/done/TASK-XXX.md`
(written by the subagent) and in git history.

## Task Lifecycle

```
open → active → done (compact to one line)
         ↓
      blocked → active → done
```

## Priority Levels

| Level | Meaning             | Behavior                           |
| ----- | ------------------- | ---------------------------------- |
| P0    | Blocking the system | Execute immediately                |
| P1    | High priority       | Execute next                       |
| P2    | Normal              | Execute in order                   |
| P3    | Low                 | Execute when nothing else is ready |

## Inbox Conventions

User questions live in `tasks/INBOX.md`.

- Questions are numbered sequentially: Q-001, Q-002, … (3-digit zero-padding, never reused)
- skillwave assigns Q-numbers when queuing questions
- When the user answers, skillwave marks it answered inline and removes it from the open section
- Never block the whole system waiting for an answer — continue with unblocked tasks
