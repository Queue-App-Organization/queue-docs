# Portable Agent Tooling

This document explains the project-level tool setup agents should use across Codex, Claude Code, opencode, Antigravity, or any future IDE.

## Decision

Agent tooling must be configured at the project/runtime layer, not through a single IDE.

Use `.agents/` as the portable contract:

- `.agents/mcp/mcp.template.json`
- `.agents/env/env.example`
- `.agents/permissions/agent-permissions.yaml`
- `.agents/runbooks/bootstrap.md`
- `.agents/runbooks/git-workflow.md`
- `.agents/runbooks/jira-workflow.md`

## Required Tool Categories

### Jira

Agents need Jira access through the official Atlassian Rovo MCP server where possible.

Default remote endpoint:

```text
https://mcp.atlassian.com/v1/mcp/authv2
```

Agents need Jira access for:

- Ticket search.
- Ticket read.
- Assignment.
- Comments.
- Status transitions.
- Acceptance criteria tracking.
- QA and deployment evidence.

### GitHub

Agents need GitHub access through the official GitHub MCP server where possible.

Default remote endpoint:

```text
https://api.githubcopilot.com/mcp/
```

Agents need GitHub access for:

- Repository reads.
- Branch creation.
- Commits.
- Pull requests.
- Reviews.
- CI/check status.
- Workflow/deployment evidence where applicable.

### Local Git

Engineering agents should use local git inside the checked-out repository for normal implementation work.

### CI/CD

DevOps and engineering agents need access to CI results. Production deploy access must remain gated.

## Runtime Strategy

Each IDE should load equivalent tools:

```text
Codex         -> project MCP config
Claude Code   -> project MCP config
opencode      -> project MCP config
Antigravity   -> project MCP config
```

If an IDE cannot directly consume `.agents/mcp/mcp.template.json`, create a runtime-specific adapter from this template. Do not change the logical tool names or permissions model.

## Setup Status

Current setup is a portable template, not a completed credentialed integration.

Still required before agents can execute real Jira/GitHub work:

- Authenticate to the Atlassian Rovo MCP server.
- Authenticate to the GitHub MCP server.
- Create service accounts or tokens.
- Define exact GitHub repositories.
- Confirm Jira project key `RESQ`.
- Validate permissions for each agent role.
- Test access in at least one runtime.

## Recommended Next Step

Create a small sandbox ticket and sandbox repository branch to verify:

- Hermes can assign a ticket.
- A specialist agent can read the ticket.
- The specialist agent can create a branch and commit.
- The specialist agent can open a PR.
- QA can comment with test evidence.
- Hermes can accept or reject the ticket.
