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
