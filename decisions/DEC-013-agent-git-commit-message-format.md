# DEC-013: Agent Git Commit Message Format

Status: Accepted

## Decision

Every git commit made by an agent must use the Jira ticket key followed by the agent name.

Format:

```text
JIRA-TICKET: AGENT_NAME - short description
```

Example:

```text
RESQ-100: BACKEND - code changes for feature
```

## Reason

The project will be managed through Jira and agent-owned work. Commit messages must make it easy to trace code changes back to the ticket and responsible agent.

## Rules

- Use the exact Jira ticket key from the work item.
- Use a clear uppercase agent name, such as `BACKEND`, `FRONTEND`, `FULLSTACK`, `QA`, `DEVOPS`, `UX`, `PM`, or `ANALYTICS`.
- Keep the description concise and action-oriented.
- Do not commit unrelated work under the same ticket.
- If one change spans multiple agents, the implementing agent name should be used.

## Implication

Agents must not create generic commit messages such as `fix bug`, `updates`, or `WIP`. Commits should remain traceable and reviewable.

## Related

See `queue-docs/AGENTS.md`.
