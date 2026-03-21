---
name: role-creation
description: "How skillwave creates, updates, and retires dynamic roles in the role registry."
---

# Skill: Role Creation

How skillwave creates, updates, and retires roles.

## When to Create a Role

Create a role when:

- A new type of work appears that no existing role covers
- A task requires specialized expertise not in the current registry

Don't create a role when:

- An existing role can handle the work (even if imperfectly)
- It's a one-off task that skillwave can handle directly

## Role File Template

Create `.github/roles/{role-name}.md` — lowercase, hyphenated (e.g. `quant-engineer.md`).

```markdown
# {role-name}

- **Expertise:** [domain and tools — 1 line, specific enough to guide behavior]
- **Writes to:** [output directory — exactly one]
- **Reads:** [comma-separated list of context files — max 5]
- **Constraints:** [hard rules — testable, not vague. One bullet per rule.]
- **Active since:** [YYYY-MM-DD]
- **Last updated:** [YYYY-MM-DD]
- **Status:** active

## When to Use

[1-2 sentences: what tasks trigger this role, and what it should NOT do.]

## Role Instructions

[Exact text skillwave pastes into the subagent prompt. 4-8 lines max.]
```

After creating the file, insert a row **above** the `<!-- ADD ROLE ROW: ... -->`
comment in `.github/roles/README.md`:

```markdown
| {role-name} | {one-line expertise summary} | `.github/roles/{role-name}.md` |
```

If the role owns a directory, also insert a row above the
`<!-- ADD DIRECTORY ROW: ... -->` comment in the same file.

## Pre-Creation Checklist

Before writing a new role file, verify all of the following:

- [ ] **No existing role covers this work** — only slightly partially
- [ ] **Expertise is concrete** — names a domain, not a vague trait ("React + TypeScript frontend" not "coding")
- [ ] **Writes-to directory is unambiguous** — exactly one directory
- [ ] **Reads list is ≤5 entries** — if more are needed, the task scope is too broad
- [ ] **Constraints are testable** — "use CSS modules" is testable; "write clean code" is not
- [ ] **Role Instructions is ≤8 lines** — the exact text injected into the subagent prompt
- [ ] **Decision is logged** — entry added to Decision Log in `.github/roles/README.md`

Do not skip this checklist. A poorly-defined role produces low-quality output
and the only recovery is rework.

## Revisiting a Role

After completing a task, skillwave must check whether the assigned role's file
needs updating. Update the file in place — never create duplicates or versioned copies.

| Situation                           | Action                                                   |
| ----------------------------------- | -------------------------------------------------------- |
| Task produced wrong output          | Tighten Constraints + Role Instructions                  |
| New constraint discovered           | Add to Constraints + Role Instructions                   |
| Writes-to or Reads changed          | Update those fields                                      |
| Role handles work outside its scope | Expand or split into two roles                           |
| Role no longer needed               | Set `Status: inactive`, move row to Inactive Roles table |

Log the reason for every change in the Decision Log in `.github/roles/README.md`.

## Retirement

1. Set `Status: inactive` in `.github/roles/{role-name}.md`
2. Move the row from Active Roles to above the `<!-- MOVE RETIRED ROLE ROWS HERE -->` comment in `.github/roles/README.md`
3. Log the retirement in the Decision Log in `.github/roles/README.md`

## Decision Log Format

All role lifecycle decisions go in the Decision Log section of `.github/roles/README.md`.
Insert each entry **above** the `<!-- ADD ENTRY: ... -->` comment:

```
YYYY-MM-DD — [decision description]
```

Examples:

- `2025-11-10 — Created backend-dev role for REST API and database work`
- `2025-12-03 — Updated backend-dev role: added Redis caching constraint after TASK-018`
- `2025-12-15 — Retired data-migrator role after schema migration completion`
