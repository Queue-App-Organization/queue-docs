# DEC-017: Counter-service mode widens the MVP

Date: 2026-09-21
Status: Accepted (founder)
Amends: DEC-007 (MVP scope is narrow), DEC-004 (table management is MVP scope)

## Context

DEC-007 kept the MVP narrow around seated restaurants; DEC-004 made table
management MVP scope. Real demand (e.g. Naturals Ice Cream outlets: pay,
take a token, weekend 20-minute queues) and the founder's generic-queue
brief (ice-cream counters, doctor OPDs, passport and government offices)
need table-optional operation.

## Decision

Add an explicit per-shop `service_mode` (`SEATED` | `COUNTER`) to the MVP:

- One mode per shop, chosen at onboarding; COUNTER shops skip table setup.
- Counter flow reuses the queue-entry lifecycle plus a `SERVICE_STARTED`
  audit event — no parallel state machine, no new states.
- Token number (phone screen + voice announcement) is the customer
  identity; no printed tokens.
- Table UI, copy, validation, and table analytics are gated behind
  `SEATED`; counter analytics (throughput, service time, abandonment)
  apply in `COUNTER`.
- Verticals beyond restaurants are configuration (labels, call copy,
  resource model), not separate builds.

## Consequences

- Seating remains the default mental model (`SEATED`); counter is a
  first-class equal, not a hack on nullable tables.
- Analytics, ETA basis, onboarding, and WhatsApp templates must all be
  mode-aware from the start.
- Amends DEC-007: the MVP now covers seated restaurants AND counter
  service. Reservations, delivery, POS, and loyalty stay out.
