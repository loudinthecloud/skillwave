---
name: collaboration
description: "Collaborate across agents — hand off work, resolve disagreements, manage dependencies, and request input or reviews from other agents."
---

# Skill: Collaboration

How skillwave manages collaboration between roles in the subagent model.

## Core Model

Roles do not communicate directly. skillwave passes context between them:

1. Role A completes work — returns result to skillwave
2. skillwave extracts relevant output (1-3 line summary + artifact paths)
3. skillwave feeds that context into Role B's prompt

## Context Passing

When task B depends on task A:

- Include A's result summary in B's subagent prompt
- Include paths to artifacts A created
- If B needs specific content from A's artifacts, excerpt the relevant sections

## Disagreement Resolution

If two roles produce conflicting outputs:

1. skillwave evaluates both based on project context
2. For minor decisions: skillwave picks the better approach, documents rationale
3. For major decisions: escalate to user via INBOX

## Dependency Management

- Tasks declare dependencies via `Depends on:` field
- skillwave only executes a task when its dependencies are `done`
- If a dependency is blocked, the dependent task is implicitly blocked

## Review Pattern

When work needs review by a different role:

1. Create a review task assigned to the reviewing role
2. Include the artifacts to review in the task context
3. The reviewer returns feedback
4. skillwave creates fix tasks if needed based on feedback
