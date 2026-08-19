# DEC-014: IDE-Agnostic Agent Tool Access

Status: Accepted

## Decision

Agent access to Jira, GitHub, git, CI/CD, documentation, and deployment evidence must be IDE-agnostic.

Codex plugins, Claude Code integrations, opencode helpers, and Antigravity integrations may be used as clients, but they must not be the primary source of project capability.

## Reason

Agents need to work regardless of whether the active client is Codex, Claude Code, opencode, Antigravity, or another runtime. If tool access is tied to one IDE, the agent organization becomes locked to that client and cannot reliably execute work elsewhere.

## Required Tooling

The agent runtime should expose:

- Official Atlassian Rovo MCP or equivalent Jira API access.
- Official GitHub MCP or equivalent GitHub API/CLI access.
- Local git.
- CI/CD access.
- Filesystem access to project documentation.
- Secrets through environment injection or a secret manager.

The portable project contract is defined under `.agents/` and summarized in `queue-docs/agent-tools/PORTABLE-TOOLS.md`.

## Operating Rule

Configure tools once for the agent runtime, then let each IDE act as a replaceable client.

Hermes coordinates tool usage and assignment, but specialist agents must still be able to use Jira and GitHub directly within their permission scope.

## Implication

Do not consider Codex plugin installation sufficient project setup. Before writing final agent files, define portable MCP/tool configuration, required environment variables, authentication model, and per-agent permission scopes.

## Related

See `queue-docs/AGENTS.md`.
