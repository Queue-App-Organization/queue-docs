# Architecture

Source context:
- ChatGPT shared chat: https://chatgpt.com/share/6a857d66-3bb8-83ee-95d3-550ff535328f
- ChatGPT shared chat: https://chatgpt.com/share/6a857d2c-fb18-83ee-bf45-588f96399da1

## Product Architecture

The product should be built around four primary surfaces:

- Customer mobile web check-in and status page.
- Staff live operations dashboard.
- Owner/admin setup and analytics dashboard.
- WhatsApp transactional notification channel.

## Recommended System Components

```text
Customer Web
    |
    v
Application API <-> Database
    |
    +-> Staff Dashboard Realtime Updates
    |
    +-> WhatsApp Provider
    |
    +-> Analytics/Event Pipeline
    |
    +-> Admin Dashboard
```

## Application Modules

### Restaurant Setup

Owns restaurant profile, operating hours, table inventory, QR code generation, and initial onboarding.

### Queue Management

Owns queue entries, queue status, customer-facing status, manual walk-ins, cancellation, skip, no-show, and queue ordering.

### Table Management

Owns table capacity, status, section, cleaning state, and party assignment.

### Notification Service

Owns WhatsApp messages for confirmation, position updates, almost-ready alerts, table-ready alerts, cancellation confirmations, and later automated reminders.

### ETA Service

MVP starts simple. Later it evolves into ETA based on party size, table availability, historical wait data, day/time patterns, table turnover prediction, and reservation pressure.

### Analytics

Consumes domain events and produces owner-facing metrics:

- Average wait.
- Longest wait.
- 95th percentile wait.
- No-show rate.
- Cancellation rate.
- Table utilization.
- Table turnover.
- Queue-to-seated conversion.
- Notification-to-arrival rate.

Implemented at RESQ-18 (MVP set, REQUIREMENTS.md Analytics): a daily
metrics endpoint (`GET /owner/restaurants/{id}/analytics/daily`) computes
the seven MVP metrics from the append-only event log only (DEC-006) —
waiting now, average wait, longest wait, tables occupied, parties served,
no-shows, cancellations — for any day in the restaurant's own timezone
(day boundaries converted to UTC; Indian date/time formatting NFR). Exact
event-derived definitions live in DOMAIN.md (Analytics Metrics). Owner
role required; other staff roles are denied (SECURITY.md Authorization).

### Authorization

Owns staff roles and permissions. MVP can start with simple owner/staff roles; V1 can add granular permissions.

## Data Architecture

Use relational modeling for the operational core:

- restaurants.
- staff_users.
- customers.
- queues.
- queue_entries.
- tables.
- sections.
- seating_events.
- notification_events.

Events should be append-only where practical. The current state can be stored on operational tables, while the event log provides history, analytics, and auditability.

## Realtime Architecture

The staff dashboard should update when:

- A customer joins.
- A customer cancels.
- A customer marks "I'm on my way."
- Staff calls a party.
- Staff marks arrived, seated, skipped, or no-show.
- Table status changes.

Implementation options:

- WebSockets.
- Server-sent events.
- Polling for early prototype only.

### Realtime transport (confirmed at RESQ-9)

The prototype uses **Server-Sent Events (SSE)**:

- Staff dashboard subscribes to `GET /staff/restaurants/{id}/stream`
  (authenticated, restaurant-scoped). The server re-pushes a full dashboard
  snapshot (~every 1.5s) as `event: snapshot`, so joins, cancels, calls,
  seats and table-status changes surface within ~2 seconds.
- SSE is one-way (server -> browser), which fits the dashboard's needs;
  staff actions are plain HTTP POSTs (RESQ-10).
- The stream's data source is `dashboard_event_generator` (queue-api),
  which re-reads committed state from Postgres per snapshot - correct in
  any deployment (multi-worker safe, no broker required).
- WebSockets can replace SSE later if bidirectional traffic (e.g. live
  typing) ever justifies it.

## AI Team / Delivery Architecture

Agent roles, operating rules, human approval gates, backward compatibility policy, and git commit-message requirements are defined in `queue-docs/AGENTS.md`.

## Release Gates

Agents may autonomously:

- Create tickets.
- Draft designs.
- Write code.
- Add tests.
- Refactor.
- Create PRs.
- Update documentation.
- Deploy to staging.
- Improve observability.

Require human approval for:

- Production database migrations with destructive consequences.
- Payment changes.
- Security-sensitive changes.
- Major architecture changes.
- Deleting customer data.
- Changing pricing.
- External communication.
- Production deployments until reliability is proven.

## Environment Strategy

Recommended environments:

- Local development.
- Preview per pull request.
- Staging.
- Production.

Production release should require passing CI, migrations reviewed, monitoring in place, and explicit approval while the system is young.

## Repository Layout & Tech Stack

Pre-customer baseline (DEC-012: prefer the best clean design; stack is
reversible until real usage). Decided at RESQ-2 (Foundation wave), 2026-08-19.

```text
Queue-App-Organization/
  queue-docs/            docs (product, domain, architecture, security, decisions)
  queue-api/             backend API + event log
  queue-web/             frontend SPA (customer mobile web, staff dashboard, owner dashboard)
  queue-infrastructure/  docker-compose local stack, GitHub Actions CI, deploy tooling
```

- **queue-api**: Python 3.12, FastAPI, SQLAlchemy 2 (async, asyncpg),
  Alembic migrations, Postgres 16. `/health` reports app + database status.
- **queue-web**: React 18 + Vite + TypeScript (single SPA for all three
  surfaces in MVP).
- **queue-infrastructure**: `docker-compose.yml` (db + api + web with
  healthchecks) and GitHub Actions CI (lint, typecheck, unit tests, build).
- **Realtime**: SSE for the prototype (WebSockets/SSE per this document;
  polling only as last resort).
- **Local dev**: `docker compose up` from `queue-infrastructure/`; setup
  documented in each repo README (target: under 10 minutes).

## Observability

Minimum observability:

- API request logs.
- Error logs.
- Queue state transition logs.
- WhatsApp delivery status logs.
- Realtime connection health.
- Deployment history.
- Basic business metrics.

Operational failures should be visible to staff when action is needed, but customer-facing copy should remain simple and reassuring.
