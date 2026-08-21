# DEC-015: Jira Organized by Business Capability Epics

Status: Accepted

## Decision

Jira (project RESQ) is organized around seven business-capability epics rather than agent roles or build waves. Agents work across epics; `wave-*` labels remain for build sequencing and agent labels remain for ownership.

## Epics

- Queue Management (RESQ-35)
- Table Management (RESQ-36)
- Customer Experience (RESQ-37)
- Restaurant Management (RESQ-38)
- Analytics (RESQ-39)
- Billing (RESQ-40)
- Platform / Foundation (RESQ-41)

## Reason

- Capabilities map 1:1 to the domain entities and state machines in `DOMAIN.md` (queue entry, table, restaurant, customer).
- A ticket belongs to the product capability it changes, independent of which agent implements it — agent-centric organization breaks down the moment two agents touch one feature.
- Build sequencing (`wave-*` labels) and ownership (agent labels) stay orthogonal to the capability taxonomy.
- Infrastructure and meta work (CI, deploys, auth foundations, deferral trackers) previously had no home; Platform / Foundation covers it.

## Rules

- Every story/task/bug belongs to exactly one epic. Cross-capability operations (e.g. seating: queue-entry transition + table occupancy) are NOT duplicated across epics — the canonical story lives in one epic and is cross-referenced from the other.
- Billing means SaaS monetization of Queue App (charging restaurants for the service). It is distinct from in-restaurant payments, which are explicitly out of MVP scope — do not scope-creep.
- "Revenue impact" analytics is an estimate (seated covers x configured average spend) unless a POS integration exists.
- New tickets get an epic assignment at creation (backlog intake, PM/Hermes).
- V1/V2 deferral trackers (RESQ-20/21) live under Platform / Foundation until work starts, then split into per-epic lists.

## Implication

Epics are containers, not owners. Agent labels still determine who implements; `wave-*` labels still sequence work. Board filters combine epic, wave, and agent.

## Related

- `queue-docs/AGENTS.md` — Ticket Lifecycle
- `queue-docs/DOMAIN.md` — core entities and state machines
- `queue-docs/decisions/DEC-007-mvp-scope-is-narrow.md` — MVP scope (payments/POS excluded)
