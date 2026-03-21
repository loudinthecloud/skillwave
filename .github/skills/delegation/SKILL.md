---
name: delegation
description: "How skillwave constructs prompts for subagent calls — prompt templates, context rules, and token budget."
---

# Skill: Delegation

How skillwave constructs prompts for subagent calls.

## Subagent Prompt Template

```
You are a {role.expertise} specialist working on a software project.

RULES:
- Write output to: {role.writesTo}
- {role.constraints}
- If this task creates or modifies testable code (business logic, algorithms,
  utilities), include tests covering the acceptance criteria. If existing tests
  already cover the change, note which tests provide coverage instead of writing
  new ones. Skip tests for config, scaffolding, and documentation tasks.
- On completion, write a task report to tasks/done/{task.id}.md
  using the template at tasks/done/TEMPLATE.md.
- Do NOT read or write tasks/BACKLOG.md — board management is not your concern.
- Be concise in your response. No preamble. Just do the work and report back.

{role.roleInstructions}

TASK: {task.title}
{task.description}

ACCEPTANCE CRITERIA:
{task.acceptance}

CONTEXT:
{gathered context — file contents, previous task results}

RESPOND WITH:
1. What you did (1-3 sentences)
2. Files created or modified (list)
3. Any follow-up tasks needed (list, or "none")
4. Any questions for the user (list, or "none")
5. Confirm: tasks/done/{task.id}.md written
```

## Prompt Construction Rules

1. **Minimal context**: Only include what the subagent needs. If a task is
   "add a button to the header", don't include the full design system — just
   the relevant section.
2. **Excerpt, don't dump**: When including file context, extract the relevant
   section (e.g., lines 50-80) rather than the whole file.
3. **Reference previous results**: If task B depends on task A, include A's
   1-3 line result summary and the paths to artifacts A created.
4. **Structured response format**: Always specify what the subagent should
   return. This prevents verbose, unstructured responses.
5. **No meta-instructions**: Don't tell the subagent about the agent system,
   other roles, the backlog, or conventions. It doesn't need to know.

## Invoking Subagents

Use the `runSubagent` tool to invoke a subagent. This is the VS Code Copilot
agent tool that spawns a stateless child agent. Each invocation:

- **Is stateless**: the subagent has no memory of prior calls. All necessary
  context must be in the prompt.
- **Returns a single message**: the subagent's entire output comes back as one
  text response. There is no interactive back-and-forth.
- **Has no access to your conversation**: the subagent cannot see the user's
  chat history, other tasks, or the broader system context.

### Parallel invocation

skillwave may invoke up to 4 subagents concurrently (see /orchestration skill
§ Parallelism). Each parallel call must still be fully self-contained — no two
parallel subagents may write to the same directory or depend on each other's
output. Parse all responses after all parallel calls return.

### Invocation parameters

| Parameter     | Required | Description                                                  |
| ------------- | -------- | ------------------------------------------------------------ |
| `prompt`      | yes      | The fully constructed prompt (see template above)            |
| `description` | yes      | Short label (3-5 words) shown in the UI                      |
| `agentName`   | no       | Named agent to invoke; omit to use the default general agent |

### Error handling

If the subagent response is empty, malformed, or indicates failure:

1. **Diagnose first.** Before retrying, check which failure mode occurred:
   - Report file missing → subagent didn't write `tasks/done/{task.id}.md`
   - Output format wrong → response didn't match the RESPOND WITH structure
   - File paths ambiguous → subagent couldn't find or create the right files
   - Scope too broad → subagent produced partial or confused output
   - Acceptance criteria unclear → subagent did work but missed the point
2. Log the failure _and the diagnosed cause_ in the task's `Result` field
3. Retry once with a targeted fix for the diagnosed cause (e.g., add explicit
   file paths, simplify acceptance criteria, reduce scope to one deliverable)
4. If the retry fails, mark the task `blocked` and add a question to INBOX
   including both failure summaries so the user has context

### Parsing the response

After invoking, parse the response for:

- **What you did** → use as the task `Result` summary
- **Files created or modified** → record artifact paths
- **Follow-up tasks needed** → create on the board
- **Questions for the user** → add to INBOX

## Context Gathering

Before building a prompt, decide what context the subagent actually needs:

1. **Role definition** — always include Expertise, Writes-to, Constraints, and Role Instructions from the role file
2. **Task fields** — title, description, acceptance criteria
3. **Dependency results** — for each `Depends on:` task, include its 1-3 line
   Result summary and the paths of artifacts it created
4. **File content** — read only the files listed in `role.reads`, and only the
   relevant sections:
   - If you need a function → read ±20 lines around it
   - If you need a config value → read ±5 lines around it
   - If you need overall file structure → read the first 50 lines
5. **Summarize vs. include raw**: include raw content when the subagent needs
   to modify or extend it; summarize (1-2 sentences) when it only needs to
   know what exists

Decision rule: if in doubt, include less. The subagent can ask for more via
"questions for the user."

## Token Budget Awareness

- A typical subagent prompt should be 50-200 lines of context (not 1000+)
- If context exceeds ~300 lines, split the task or summarize the context
- Subagent responses should be <50 lines (enforced by response format)
