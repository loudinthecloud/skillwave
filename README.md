# Skillwave

An autonomous AI agent scaffold for GitHub Copilot that turns a written goal into a working project — using GitHub Copilot subagents to plan, execute, and deliver.

You describe **what** you want built, preferably in user stories. Skillwave handles the **how**: it decomposes the goal into tasks, creates specialized worker roles on demand, dispatches up to 4 parallel subagents, and loops until everything is done.

## How It Works

1. **You write a goal** in `GOAL.md` — requirements, constraints, success criteria.
2. **Invoke `@skillwave`** in copilot.
3. **Skillwave takes over** — it reads your goal, creates a task board, spins up roles (e.g. backend engineer, designer), and delegates work to subagents.
4. **Work happens autonomously** — tasks execute in parallel, results feed back into the board, follow-up tasks are created as needed.
5. **Questions for the user are queued** in `tasks/INBOX.md` — you answer asynchronously; unblocked work continues in the meantime.

## Getting Started

```bash
git clone git@github.com:loudinthecloud/skillwave.git
cd skillwave
```

1. Open the folder in with github copilot.
2. Fill in `GOAL.md` — describe what you want built (use an LLM to help you if you like).
3. Open Copilot Chat and after seleting the `@skillwave` agent, just say "run" or "execute" to get started.

### CLI usage

```bash
copilot --autopilot --yolo --max-autopilot-continues 10000
```

Then select the model with `/model` and the agent with `/agent`.

## Project Structure

| Path               | Purpose                                                |
| ------------------ | ------------------------------------------------------ |
| `GOAL.md`          | Your project goal — the only file you need to fill in  |
| `tasks/BACKLOG.md` | Task board — single source of truth for all task state |
| `tasks/INBOX.md`   | Async questions for the user                           |
| `tasks/done/`      | Completed task reports                                 |
| `.github/agents/`  | Agent definition (the `skillwave` orchestrator)        |
| `.github/roles/`   | Dynamic role definitions, created on demand            |
| `.github/skills/`  | Domain knowledge powering the orchestrator             |

## Key Concepts

**Orchestration** — Skillwave runs a continuous loop: read the board → batch unblocked tasks → dispatch subagents in parallel → collect results → update the board → repeat.

**Roles** — Lightweight worker definitions (expertise, constraints, allowed files) created dynamically when a task needs a specialist. Retired when no longer needed.

**Skills** — Built-in knowledge modules (orchestration, delegation, task management, user comms, role creation, reporting, collaboration) that define how skillwave operates.

**Task lifecycle** — `open → active → done` (or `blocked → active → done`). Each completed task produces a report in `tasks/done/` and a git commit.

**Stateless subagents** — Every worker call is a one-shot invocation with a self-contained prompt. No shared memory between agents; all coordination flows through the markdown board.

## License

[MIT](LICENSE)
