---
name: orchestration
description: "How skillwave plans and executes work autonomously — decomposition, scheduling, and the execution loop."
---

# Skill: Orchestration

How skillwave plans and executes work autonomously.

## Planning

When the user provides a goal:

1. Identify tasks (concrete work units) — phases are a mental grouping only, NOT execution checkpoints
2. For each task, identify the role needed
3. If the role doesn't exist yet → create it (see /role-creation skill)
4. Add tasks to the board with dependencies
5. Begin execution immediately — do NOT pause between phases or groups

**State storage is ONLY `tasks/BACKLOG.md`.** Never create external databases (SQLite, JSON stores, etc.) to track tasks or dependencies. The markdown board is the single source of truth.

## Execution Loop

Run continuously without stopping between phases or milestones.

**Board writes are mandatory and blocking.** Every state transition MUST be
written to `tasks/BACKLOG.md` before the loop continues. Do not defer or skip
board updates to save steps — an unwritten update did not happen.

```
LOOP:
  tasks = read_board() → filter(status=open, not blocked)
  if empty(tasks):
    if all_done → report completion to user, stop
    if all_blocked → report blockers + inbox questions, stop

  batch = tasks[0:4]  # up to 4 highest-priority unblocked tasks
  for task in batch:
    WRITE board: task.status = active        # ← mandatory before subagent call

  # ── Parallel dispatch (up to 4 concurrent subagents) ──────────────
  results = []
  for task in batch:                         # invoke in parallel
    role = read_role(task.assigned_to)
    context = gather_context(role, task)
    prompt = build_subagent_prompt(role, task, context)
    results.append( runSubagent(prompt) )    # non-blocking; up to 4 in flight
  await all(results)                         # wait for all to complete

  # ── Post-task review — run for EACH task in batch ─────────────────
  for (task, result) in zip(batch, results):
    report = read(tasks/done/{task.id}.md)
    if report missing → see /delegation skill § Error handling

    follow_ups = extract follow-up tasks from report
    decisions  = extract decisions / risks from report
    role_gaps  = note any constraints the task revealed

    WRITE board: task.status = done, result = summarize(report.summary)
    WRITE board: compact line links tasks/done/{task.id}.md
    WRITE board: create + prioritize follow_ups as new tasks
    WRITE board: unblock dependent tasks      # see /collaboration skill for rules
    if role_gaps → update role file           # see /role-creation § Revisiting a Role
    if decisions → log in .github/roles/README.md Decision Log

    # ── Git commit ────────────────────────────────────────────────────
    artifacts = report.files_created_or_modified   # from subagent response
    git add {artifacts}                            # stage ONLY this task's files
    git commit -m "feat({task.id}): {task.title}"
    # One commit per task. Never use git add -A (may capture parallel work).
    # ─────────────────────────────────────────────────────────────────
  # ───────────────────────────────────────────────────────────────────

  CONTINUE LOOP
```

## Context Gathering

See the `/delegation` skill for the full context-gathering rules and token budget.
Short summary: include role definition, task fields, dependency results, and relevant file excerpts — nothing else.

## Batching

Group related small tasks for a single subagent call when they share the
same role and context. Example: "Fix bugs #1, #2, #3" → one call, not three.

## Parallelism

The execution loop dispatches up to 4 subagents concurrently. Constraints:

1. **No write conflicts** — two parallel tasks must NOT share a Writes-to
   directory. If they do, serialize them.
2. **No dependency overlap** — a task cannot run in parallel with any task it
   `Depends on:` (or that depends on it).
3. **Board writes are sequential** — post-task review and board updates happen
   one task at a time after all parallel subagents return.
4. **Degrade gracefully** — if only 1-2 tasks are eligible, run what's
   available. Don't wait to fill a batch of 4.

## Verification

Code tasks require verification before they are marked `done`:

1. After a subagent completes a code task, create a **verification task**
   assigned to the `tester` role (or skillwave if no tester role exists)
2. The verification task checks: files exist, code is syntactically valid,
   tests pass, acceptance criteria are met
3. If verification fails → create a fix task assigned to the original role
4. If verification passes → mark the original task `done`
5. **Retry cap:** track the fix/verify cycle count per original task. After
   3 failed verification cycles, mark the task `blocked` and add a question
   to INBOX with the failure details. Do not keep looping.

Verification is **mandatory** for tasks that create or modify code.
It is optional for documentation, planning, and design tasks.

## Testing Requirements

Tests should accompany code changes when they add value. Not every task
requires new tests.

1. **When to write tests** — new business logic, algorithms, data
   transformations, and non-trivial utilities. Skip tests for config files,
   scaffolding, pure wiring, and documentation.
2. **When to skip tests** — if existing tests already cover the changed code,
   no new tests are needed. The subagent should note which existing tests
   provide coverage.
3. **Scope** — tests cover the task's acceptance criteria, not every line
4. **Passing** — all tests (new and existing) must pass before `done`
5. **Location** — follow the project's existing test conventions
6. **Verification includes tests** — the verification step runs the test
   suite and confirms all tests pass

## Git Commits

After each task is marked `done` (post-task review complete), skillwave
commits that task's deliverables:

1. Collect the file list from the subagent's "Files created or modified" response
2. Also include the task report: `tasks/done/{task.id}.md`
3. Stage only those files: `git add <file1> <file2> ...`
4. Commit with a conventional message: `feat(TASK-XXX): <task title>`
5. **Never use `git add -A`** — parallel tasks may have unstaged work in progress
6. One commit per task — do not batch commits across multiple tasks
7. If the commit fails (e.g., nothing to commit), log the issue and continue

## Scaling Guidance

When the backlog grows large (>15 open tasks):

1. **Summarize, don't re-read**: maintain a running 1-paragraph project summary
   in the first section of `tasks/BACKLOG.md` capturing current state
2. **Batch by role**: group tasks by assigned role, execute same-role tasks
   together to minimize context switches
3. **Prune stale tasks**: if a task has been open and unblocked for 10+ other
   task completions, re-evaluate whether it's still needed
4. **Dependency graph**: when reasoning about task ordering, only consider
   direct dependencies — not transitive ones

## Error Handling

See the `/delegation` skill § Error handling for the full retry-and-block protocol.
Short summary: retry once with narrowed scope; if still failing, mark blocked and add question to INBOX.

## Hierarchy

Hierarchy is capped at 2 levels: lead → worker. No deeper chains.

For complex projects, skillwave may create lead roles that produce sub-task
plans. skillwave then executes those plans by invoking worker roles directly.
Leads never invoke subagents themselves.
