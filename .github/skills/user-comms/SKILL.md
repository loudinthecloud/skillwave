---
name: user-comms
description: "How skillwave interacts with the user — inbox management, briefings, and async question handling."
---

# Skill: User Communication

How skillwave interacts with the user and manages async questions.

## Briefing Format

When the user asks for status/briefing:

```markdown
## Briefing — [date]

**Progress since last check-in:**

- [completed task summaries, 1 line each]

**Currently working on:**

- [active tasks, 1 line each]

**Blocked — needs your input:**

- Q-XXX: [question summary] (blocking TASK-YYY)

**Up next:**

- [next 3-5 tasks in priority order]
```

## Inbox Management

File: `tasks/INBOX.md`

When a subagent or skillwave needs user input:

1. Add an entry to INBOX.md with: id, question, context, blocking task, priority
2. Do NOT stop execution — continue with unblocked tasks
3. When user answers → mark `Status: resolved`, fill in Resolution, then move
   the entire entry to the **## Resolved** section at the bottom of INBOX.md.
   This keeps the open section clean.
4. Prune the Resolved section when it exceeds 20 entries or entries are >30 days old.

## When to Ask the User

ASK for:

- High-level design direction when multiple valid paths exist
- Actions agents can't perform (deploying to prod, DNS changes, etc.)
- Confirmation before destructive/irreversible operations

DON'T ASK for:

- Implementation details (make a decision, document rationale)
- Which framework/library to use (pick the best fit, document why)
- Task prioritization (skillwave owns this)
- Code style choices (follow existing patterns)

## User Feedback Processing

When user provides unsolicited feedback or instructions:

1. Acknowledge concisely
2. Translate into task board changes (new tasks, priority changes, etc.)
3. Resume execution with updated priorities
