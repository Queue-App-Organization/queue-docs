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

**Verified working (2026-08-19) in the Hermes runtime:**

- Jira: project `RESQ` exists at https://restoq.atlassian.net (company-managed,
  software). Full agent workflow installed: `Backlog → Ready → In Design →
  Ready For Engineering → In Progress → Code Review → Ready For QA → In QA →
  Ready For Deploy → Done` (+ `Blocked`). Story/Task/Sub-task issue types.
  Board: "RESQ Board". Sandbox verified: create, assign, comment, transition,
  accept (see sandbox ticket RESQ-1, closed).
- GitHub: org `Queue-App-Organization` with repos `queue-docs` (public) and
  `queue-api`, `queue-web`, `queue-infrastructure` (private, bootstrapped).
  `gh` authenticated; sandbox verified: branch, commit, PR, review comment,
  close, delete-branch.
- Local: workspace + local git verified.

Notes:

- The docs historically referenced a `RESQ` project that did not exist (the
  site's only project was `SCRUM`). The `RESQ` project has been created and is
  now canonical; `.agents/env/local.env` `JIRA_PROJECT_KEY=RESQ`.
- Jira REST access uses the API token in `.agents/env/local.env` (never commit
  it). The Atlassian Rovo MCP OAuth flow is the long-term path.
- Remaining: branch protection on the code repos (Foundation CI ticket),
  GitHub MCP OAuth for non-CLI runtimes, and per-role permission validation in
  at least one more runtime.

## Recommended Next Step

The sandbox loop (below) has been executed and passed; it is now a regression
check to re-run when the tool contract changes (new runtime, new permissions).

- Hermes can assign a ticket.
- A specialist agent can read the ticket.
- The specialist agent can create a branch and commit.
- The specialist agent can open a PR.
- QA can comment with test evidence.
- Hermes can accept or reject the ticket.
