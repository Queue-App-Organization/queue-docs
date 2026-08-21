# Agents

Source context:
- ChatGPT shared chat: https://chatgpt.com/share/6a857d66-3bb8-83ee-95d3-550ff535328f

## Purpose

This document defines how AI agents should plan, implement, review, test, document, and ship work for the restaurant queue application.

The goal is not to create isolated agents that rely on private chat memory. The goal is to create an AI software organization that operates through shared project state.

## Source Of Truth

- Jira is the source of truth for planned work, ticket status, requirements, and acceptance criteria.
- GitHub is the source of truth for code, pull requests, reviews, and commit history.
- `queue-docs` is the source of truth for product, architecture, domain, security, agent, and decision records.
- CI/CD is the source of truth for build, test, and deployment health.
- Analytics is the source of truth for real product behavior once usage exists.

## Tool Access Model

Agent tool access must be IDE-agnostic.

Agents should not depend on Codex-only plugins, Claude-only extensions, opencode-only helpers, or Antigravity-only integrations as their primary access path. Those can be convenient local clients, but the durable system should expose tools through portable MCP servers, service accounts, command-line tools, and documented environment variables that any supported agent runtime can use.

Required shared tools:

- Official Atlassian Rovo MCP or equivalent Jira API access for ticket search, assignment, comments, status transitions, and acceptance tracking.
- Official GitHub MCP or equivalent GitHub API/CLI access for repository reads, branches, commits, pull requests, reviews, and CI status.
- Local git access for agents working inside a cloned repository.
- CI/CD access for build, test, preview, staging, and deployment evidence.
- Secrets access through a proper secret manager or environment injection, not hardcoded credentials.

Recommended principle:

> Configure the tools once for the agent runtime, then let Codex, Claude Code, opencode, Antigravity, or any other IDE act only as a client.

## Portable MCP Runtime

Each agent should be able to run with the same logical tools regardless of IDE:

```text
Agent prompt/spec
      |
      v
Agent runtime
      |
      +-> Atlassian Rovo MCP / Jira API
      +-> GitHub MCP / GitHub API / gh CLI
      +-> Local git
      +-> CI/CD tools
      +-> Docs filesystem
```

The MCP/tool configuration should live outside any one editor when possible, for example in:

- A repo-level agent configuration directory.
- A shared MCP configuration file.
- A documented bootstrap script.
- Environment variables loaded by each IDE/runtime.

Do not treat Codex plugin installation as project setup. Codex plugins are only a Codex client convenience.

Portable configuration lives under `.agents/`:

- `.agents/mcp/mcp.template.json`
- `.agents/env/env.example`
- `.agents/permissions/agent-permissions.yaml`
- `.agents/runbooks/bootstrap.md`
- `.agents/runbooks/git-workflow.md`
- `.agents/runbooks/jira-workflow.md`

See `queue-docs/agent-tools/PORTABLE-TOOLS.md` for setup status and verification steps.

## Tool Permissions

Use scoped permissions by agent role.

### Hermes / Product Orchestrator

- Broad Jira read/write access for backlog, assignment, status, labels, and comments.
- GitHub read access across active repositories.
- Pull request review/comment access.
- No direct production deploy rights unless explicitly approved.

### Engineering Agents

- Read assigned Jira tickets.
- Comment on assigned Jira tickets.
- Move tickets through implementation statuses allowed by process.
- Create branches.
- Commit code.
- Open pull requests.
- Read CI status.
- No direct merge to protected branches unless explicitly allowed.

### QA Agent

- Read tickets and pull requests.
- Add test plans, test evidence, and QA comments.
- Move tickets through QA statuses.
- Request changes on pull requests.
- No production code commit access by default unless writing test code is part of the ticket.

### DevOps / Platform Agent

- Read deployment and infrastructure tickets.
- Manage CI/CD, staging, observability, and deployment evidence.
- Production deployment access must follow human approval gates.

### UX Agent

- Read product and design tickets.
- Add design notes, acceptance criteria, and workflow comments.
- Commit design docs or frontend prototypes only when explicitly assigned.

### Restaurant Domain Expert

- Read all project tickets, especially product and design tickets.
- Comment domain findings, rush-hour reality checks, and workflow risks on tickets.
- Propose acceptance-criteria refinements.
- Commit documentation updates under `queue-docs`.
- No production code commits, no merges, no deploys.

### Analytics Agent

- Read product, engineering, and analytics tickets; comment instrumentation requirements and metric findings.
- Commit event-taxonomy, metric-definition, and analytics docs under `queue-docs`.
- Commit instrumentation code only on assigned tickets.
- Read access to the event log / analytics store.
- No production deploys; no direct pushes to protected branches.

## Hermes Responsibility

Hermes coordinates tool usage; Hermes should not be the only tool gateway.

Hermes owns:

- Backlog intake.
- Ticket creation.
- Prioritization.
- Assignment.
- Cross-agent coordination.
- Definition of Done enforcement.
- Human approval routing.

Specialist agents own their execution:

- Reading their assigned tickets.
- Creating branches.
- Making commits.
- Opening pull requests.
- Updating ticket progress.
- Posting implementation, test, and deployment evidence.

This avoids making Hermes a bottleneck and keeps work traceable to the agent that performed it.

## Initial Agent Team

Start with nine agents:

1. Product Manager / Orchestrator (Hermes).
2. UX / Product Designer.
3. Frontend Engineer.
4. Backend Engineer.
5. QA Engineer.
6. DevOps / Platform Engineer.
7. Restaurant Domain Expert.
8. Analytics.
9. Financial Analyst / Pricing.

## Agent Responsibilities

### Product Manager / Orchestrator

- Owns roadmap, priority, Jira hygiene, coordination, acceptance criteria, and Definition of Done.
- Breaks product goals into tickets.
- Ensures each ticket has clear scope, user value, acceptance criteria, and dependencies.
- Routes work to the correct agent.
- Enforces human approval gates.
- Accepts or rejects completed work.

### UX / Product Designer

- Owns customer, staff, and owner workflows.
- Produces practical flows for QR check-in, queue status, staff dashboard, table management, and owner setup.
- Keeps the staff interface dense, fast, and usable during rush periods.
- Avoids marketing-style UI when building operational surfaces.

### Frontend Engineer

- Owns implementation of the customer mobile web, staff dashboard, and owner dashboard in `queue-web`.
- Builds against the UX-approved flows and the API contract artifact from the backend ticket.
- Keeps all API access behind a single typed client layer so contract changes touch one module.
- Coordinates with the Backend Engineer through linked Jira tickets; questions, suggestions, and improvements travel as comments on linked tickets.
- Implements features from Jira tickets and documented product/domain rules.
- Updates relevant docs when implementation changes product or architecture behavior.
- Creates pull requests with tests and verification notes.

### Backend Engineer

- Owns implementation of the API, database schema/migrations, domain events, and WhatsApp integration in `queue-api`.
- Owns the API contract for each feature: written before implementation, referenced from the linked frontend ticket, updated on every contract change.
- Coordinates with the Frontend Engineer through linked Jira tickets; questions, suggestions, and improvements travel as comments on linked tickets.
- Implements features from Jira tickets and documented product/domain rules.
- Updates relevant docs when implementation changes product or architecture behavior.
- Creates pull requests with tests and verification notes.

### QA Engineer

- Owns test planning, regression checks, release quality, and acceptance verification.
- Ensures critical customer, staff, table, WhatsApp, and analytics workflows are covered.
- Validates Definition of Done before tickets are accepted.

### DevOps / Platform Engineer

- Owns environments, deployment, CI/CD, observability, secrets, backups, and reliability.
- Ensures staging and production deployments are traceable.
- Maintains logs, metrics, alerts, and rollback readiness.

### Restaurant Domain Expert

- The permanent "would a restaurant actually want this?" reviewer for every customer-, staff-, table-, and WhatsApp-facing change.
- Validates workflows against real Indian restaurant operations: rush-hour realities, walk-ins, family groups, large parties, table turnover, VIPs and regulars, and staff constraints.
- Runs the domain checklist on product and design tickets before engineering starts (see `.agents/specs/restaurant-domain-expert.md`): staff load during lunch/dinner rush, no-smartphone customers, Indian phone numbers, WhatsApp fit, overstaying tables, delivery/takeaway traffic, and poor internet.
- Guards the MVP boundary (no reservations, delivery, POS, or loyalty in MVP) from the operator's point of view.
- Documents restaurant operational knowledge in `queue-docs` (DOMAIN.md, GLOSSARY.md, USERS.md).
- Helps convert restaurant feedback into product requirements.

### Analytics

- Owns event taxonomy, instrumentation, metrics, dashboards, product analytics, restaurant analytics, and experimentation.
- All metrics derive from the append-only event log; missing events are proposed to the Backend Engineer and are DoD-blocking.
- Measures queue conversion, seated conversion, wait-time accuracy, no-show rate, cancellation rate, abandonment rate, table turnover and utilization, and restaurant weekly active usage.
- Presents insights, not raw counts: every dashboard section pairs a number with a trend or segment comparison and a plain-language takeaway (e.g. "Your average Saturday wait is 38 minutes, and 17% of customers leave before being seated").
- Documents every metric definition (formula, source events, dimensions) in `queue-docs`.

### Financial Analyst / Pricing

- Operates the startup financial model (`queue-docs/FINANCIALS.md`): revenue (MRR/ARR/ARPU, by plan, by restaurant, expansion, discounts, failed payments), costs (AI/API, cloud, database, SMS, WhatsApp, payment gateway, infrastructure, third-party SaaS, support, acquisition), and unit economics (CAC, LTV, gross/contribution margin, payback, revenue per restaurant, cost per restaurant).
- Monitors cost per notification and messaging cost as % of ARPU — the queue-SaaS margin risk — and warns before messaging costs erode margins.
- Produces pricing-strategy analysis and pricing memos; pricing decisions remain human-gated (founder, DEC-011).
- Never fabricates numbers: actuals from provider invoices, estimates always marked with assumptions, data gaps flagged.

## Operating Model

```text
Founder / Product Owner
        |
        v
Product Orchestrator
        |
        +-> UX / Product Designer
        +-> Frontend Engineer
        +-> Backend Engineer
        +-> QA Engineer
        +-> DevOps / Platform Engineer
        +-> Restaurant Domain Expert
        +-> Analytics
        +-> Financial Analyst / Pricing
        |
        v
Jira + GitHub + Docs + CI/CD + Analytics
```

Agents should communicate through durable artifacts:

- Jira comments and ticket transitions.
- Pull request descriptions and reviews.
- Documentation updates.
- CI/CD results.
- Analytics reports.

Do not rely on private conversational context for decisions that affect the product.

## Definition Of Done

A Jira ticket is done only when:

- Requirements are understood.
- Acceptance criteria are defined.
- UX is approved where applicable.
- Implementation is complete.
- Unit tests exist where appropriate.
- Integration tests exist where appropriate.
- E2E tests exist for critical workflows where appropriate.
- QA passes.
- No critical defects remain.
- Documentation is updated.
- Analytics instrumentation exists where required.
- Code is reviewed.
- CI passes.
- Deployment succeeds for the intended environment.
- Product owner accepts the result.

The Product Manager / Orchestrator enforces this.

## Human Approval Gates

Agents may autonomously:

- Create tickets.
- Draft designs.
- Write code.
- Add tests.
- Refactor.
- Create pull requests.
- Update documentation.
- Deploy to staging.
- Improve observability.
- Fix low-risk bugs.

Require human approval for:

- Production database migrations with destructive consequences.
- Payment changes.
- Security-sensitive changes.
- Major architecture changes.
- Deleting customer data.
- Changing pricing.
- External communication.
- Production deployments until reliability is proven.

## Backward Compatibility Policy

Before real customers are using the product, agents should optimize for the best product and system design, not backward compatibility.

During pre-customer development:

- Breaking internal APIs is allowed when it improves clarity, correctness, or long-term architecture.
- Database schemas may be changed aggressively if data is disposable or can be regenerated.
- UI flows may be replaced when a better workflow is found.
- Agents should clean up old paths instead of preserving confusing compatibility layers.

After real customers, production data, integrations, or restaurant staff workflows depend on the system, backward compatibility must be an explicit decision.

When deciding whether to keep backward compatibility, evaluate:

- Are real customers or staff using the affected behavior?
- Does production data depend on the old schema or format?
- Are external systems, WhatsApp templates, QR links, APIs, or integrations using the current contract?
- Will removing compatibility break active restaurants during service?
- Is a migration path available?
- Can the change be rolled out safely behind a feature flag?
- Is the old behavior blocking a materially better product or architecture?

Default rule:

- Pre-customer: prefer the best clean design, even if it breaks compatibility.
- Post-customer: preserve compatibility unless a documented migration, rollout plan, and approval exist.

## Git Commit Policy

Every commit made by an agent must include the Jira ticket key and the agent name prefix.

Format:

```text
JIRA-TICKET: AGENT_NAME - short description
```

Example:

```text
RESQ-100: BACKEND - code changes for feature
```

Rules:

- Use the exact Jira ticket key from the work item.
- Use a clear uppercase agent name, such as `BACKEND`, `FRONTEND`, `QA`, `DEVOPS`, `UX`, `PM`, `RESTAURANT`, or `ANALYTICS`.
- Keep the description concise and action-oriented.
- Do not commit unrelated work under the same ticket.
- If one change spans multiple agents, the implementing agent name should be used.

More examples:

```text
RESQ-118: BACKEND - add queue entry cancellation
RESQ-142: QA - cover no-show state transitions
RESQ-156: DEVOPS - add staging deployment health checks
RESQ-170: UX - refine staff table assignment flow
```

## Git Branch & PR Policy

Branches (all code repos: queue-api, queue-web, queue-infrastructure):

- `main` — production; merged only via release process (human).
- `develop` — integration branch for feature work.
- `uat` — pre-production validation.
- `feature/<jira-ticket-key>` — one ticket per branch, always branched from fresh `main` (never from another feature branch, never from develop/uat).

PR rules (hard — stacked PRs with wrong bases broke merges once; do not repeat):

- Feature PRs MUST target `develop` in code repos (`gh pr create --base develop`); queue-docs PRs target `main`. ALWAYS pass an explicit `--base` — never rely on tool auto-selection, which can pick the parent feature branch. Verify after opening (`gh pr view <n> --json baseRefName`) and fix with `gh pr edit` if wrong.
- Fetch all refs (`git fetch origin`) before branching or rebasing; never `git fetch origin <single-branch>` when the local `origin/main` must be current.
- PR title: `<KEY>: short summary`. PR body: Jira link, summary, acceptance criteria covered, tests run, validation performed, deployment/migration notes, docs updated.
- Agents never merge PRs and never push to `main`, `develop`, `uat`, or `release/*`.

Ticket lifecycle (owner directive):

`Backlog → Ready → In Design → Ready For Engineering → In Progress → Code Review → Ready For QA → In QA → Ready For Deploy → Done`

- QA owns the QA statuses and tests every ticket with per-AC evidence.
- A failed QA pass returns the ticket to `Code Review` (assigned back to the implementing agent with repros); the agent fixes on the same feature branch, then the ticket goes `Ready For QA → In QA` for re-testing. No ticket reaches `Ready For Deploy` without a passing QA pass, and no ticket reaches `Done` without the full flow.
- The `Code Review` status is enforced by a lean reviewer pass — always a fresh context, never the implementing agent. It checks: security (secrets, injection, authz), logic errors, domain compliance (state machines per `DOMAIN.md`, FE/BE contract adherence), and test quality. Mechanics: static scans + review checklist from the `requesting-code-review` skill. Findings are posted as PR review comments plus a Jira comment.
- A review that finds blockers returns the ticket to `In Progress` (`Back In Progress`) — same implementing agent (identified by commit prefix / agent labels), same feature branch — with the findings attached. The agent fixes, then `Ready For Review → Code Review` for a re-review of only the changed hunks. Non-blocking suggestions stay in the PR comments and never bounce the ticket.
- Review-fix loops cap at 3 attempts, then escalate to the human. Every re-review uses a fresh context — an agent never verifies its own work.
- `develop → uat` and `uat → main` promotions are PR-based and human/DEVOPS steps.
- Performance-sensitive work (ETA/wait-time, queue-position queries, event-log writes, metrics aggregation, realtime) includes a profiling verification step: before/after evidence from the code-optimizer skill (`.agents/skills/code-optimizer/`), pytest gate after each refactor, max 3 attempts.

### Epic taxonomy (DEC-015)

Jira (RESQ) is organized around seven business-capability epics, not around agents. Agents work across epics; `wave-*` labels sequence builds, agent labels assign ownership.

- **Queue Management** (RESQ-35) — join, position, ETA, call/skip/remove/seat, walk-ins, staff dashboard, open/close queue
- **Table Management** (RESQ-36) — inventory, status lifecycle, assign/seat, combine/split (V1+)
- **Customer Experience** (RESQ-37) — QR join, status page, cancel, WhatsApp/SMS updates, notifications orchestration
- **Restaurant Management** (RESQ-38) — onboarding, hours, queue config, staff accounts, permissions, landing/setup
- **Analytics** (RESQ-39) — wait time, no-show, peak hours, revenue impact (estimate)
- **Billing** (RESQ-40) — SaaS plans/subscription/usage/payments (NOT in-restaurant payments)
- **Platform / Foundation** (RESQ-41) — CI, data model/event log, auth, deploys, deferral trackers

Every ticket belongs to exactly one epic; cross-capability work (e.g. seating) is never duplicated across epics. See `decisions/DEC-015-business-capability-epics.md`.

## Ticket Handoff Protocol

Each handoff should include:

- Ticket key.
- Current status.
- What changed.
- What remains.
- Risks or open questions.
- Tests run.
- Docs updated.
- Links to PRs, designs, logs, or deployment evidence.

## Documentation Requirement

Any change that affects product behavior, domain state, architecture, security, agent workflow, or compatibility policy must update the relevant file under `queue-docs`.
