---
name: reporting
description: "Write task completion reports — document what was done, decisions made, artifacts produced, and follow-up tasks created."
---

# Skill: Reporting

Every completed task produces a report file. No exceptions.

## Task Report File

When a subagent completes a task, it writes a report to `tasks/done/TASK-XXX.md`
(where XXX matches the task ID). This is included as a mandatory instruction in
every subagent prompt via the delegation skill.

The canonical template lives at `tasks/done/TEMPLATE.md`. Subagents copy
that file as a starting point. Do not duplicate the template here — refer
to the file.

Omit sections that don't apply — the only mandatory sections are **Summary**
and **Work Performed**.

## Board Entry

The task's compact done line on the board always links the report:

```markdown
- **TASK-XXX:** [Title] — [1-line result] | tasks/done/TASK-XXX.md
```

## Depth Guidelines

The report should match the task's complexity:

| Task size                        | Report depth                                       |
| -------------------------------- | -------------------------------------------------- |
| Small (config tweak, doc update) | Summary + Work Performed only (3-5 bullets)        |
| Medium (feature, bug fix)        | All sections filled out                            |
| Large (architecture, phase)      | All sections + detailed Decisions Made sub-section |
