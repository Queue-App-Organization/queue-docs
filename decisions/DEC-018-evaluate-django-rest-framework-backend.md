# DEC-018: Evaluate Django REST Framework as an alternative backend stack

Date: 2026-10-03
Status: Proposed (decision owner: founder)
Relates to: RESQ-2 backend stack choice (ARCHITECTURE.md, Repository Layout & Tech Stack), DEC-012 (backward compatibility before customers)

## Context

The backend stack was chosen at RESQ-2 (2026-08-19): FastAPI, async
SQLAlchemy 2, asyncpg and Alembic, all in `queue-api`. ARCHITECTURE.md
records that choice as reversible until real usage, and DEC-012 favours
the best clean design while the product is pre-customer.

The founder asked for a Django REST Framework (DRF) backend so the two
stacks can be compared. So far that has produced:

- **`queue-api-drf`** (RESQ-58): Django 6.1, DRF 3.18, psycopg 3,
  gunicorn and WhiteNoise. It has `/health` (same response shape as
  `queue-api`), the Django admin, fail-closed settings, and CI covering
  lint, tests and image build. It contains no domain logic.
- **The `drf` compose profile** in `queue-infrastructure` (RESQ-59):
  `api-drf` on host port 8001, with its own Postgres (`db-drf`) on host
  port 5431.
- **A full port of `queue-api`** (RESQ-63, at the owner's direction): all
  37 API operations, all 12 models with an equivalent schema (the
  append-only trigger and partial unique indexes included), every service
  and all 25 test files (401 tests). It runs under uvicorn (ASGI) so the
  SSE dashboard stream is an async view. The contract is `queue-api`'s
  OpenAPI spec at `develop` 38c61d1, pinned in
  `queue-api-drf/contract/` and checked by a parity test.

`queue-api` is not a blank slate. It implements the MVP wedge: queue and
table state machines, the append-only event log with a DB-level
immutability trigger, staff auth and customer scoped tokens, WhatsApp
notifications, SSE realtime and analytics. That is roughly 280 tests and
8 Alembic migrations.

## Decision (proposed)

Run DRF as a **time-boxed, side-by-side evaluation**. Do not switch yet.

While the evaluation runs:

- **`queue-api` remains the system of record.** `queue-web` keeps calling
  it, and all product tickets keep landing there.
- **Product work lands in `queue-api` only.** RESQ-63 replaced the
  original "evaluation slice" rule with a one-off full port, frozen at
  `queue-api` 38c61d1. New features are not built twice. Re-syncing the
  port with later `queue-api` changes is a separate, ticketed decision,
  taken only if the evaluation continues.
- **Separate databases, always.** The two backends must never share a
  schema, because Alembic autogenerate would propose dropping Django's
  tables (RESQ-59).
- **Everything needs a ticket.** Evaluation work is tracked under the
  Platform / Foundation epic (RESQ-41) and recorded through tickets and
  PRs, as AGENTS.md requires.

Adopting DRF, which means replacing `queue-api`, is a major architecture
change. It needs founder approval (AGENTS.md approval gates, DEC-011) and
would supersede this record with an Accepted decision and a migration
plan.

## Evaluation criteria (proposed)

With the full port in place (RESQ-63), the criteria can be judged on the
whole API rather than on a slice, using the same test suite in both stacks.

1. **Domain invariants.** Can the state machines and the append-only
   event log, including the DB-level immutability trigger, be expressed
   cleanly? This covers Django migrations, raw SQL, and a single path for
   every state change.
2. **Realtime.** The staff dashboard depends on SSE (confirmed at RESQ-9).
   Under gunicorn sync workers, each open stream would hold a worker, so
   DRF would need an ASGI server and async views. The criterion is whether
   that stays simple and multi-worker safe.
3. **Auth and security parity.** Staff session cookie, CSRF origin checks,
   customer scoped tokens and public-join rate limiting must all meet
   SECURITY.md MVP controls.
4. **Delivery speed.** How quickly can an agent ship a typical ticket
   (endpoint, validation, migration, tests) in each stack?
5. **Operational fit.** Image size, deploy target, cold start and
   observability, compared on the same hosting (currently Fly for
   `queue-api`).
6. **Migration cost.** Porting about 280 tests, 8 migrations and the
   existing API contract `queue-web` depends on. Pre-customer this is
   allowed (DEC-012), but the cost should be estimated explicitly.
7. **Admin and back-office value.** How much the Django admin
   realistically saves on owner and founder tooling.

## Options at the end of the evaluation

- **Adopt DRF.** Migrate `queue-api` to DRF under a migration plan,
  repoint `queue-web`, then archive `queue-api`. Requires founder
  approval.
- **Keep FastAPI.** Archive `queue-api-drf`, remove the `drf` compose
  profile, and record the outcome here as Rejected with the reasons.
- **Extend the evaluation.** Port one more slice, with a new time box.

## Consequences

- While the evaluation runs, two backend repos exist. CLAUDE.md and
  ARCHITECTURE.md label `queue-api-drf` as experimental so agents don't
  build product features there by mistake.
- The compose `drf` profile is opt-in, so the default local stack and
  its under-10-minute setup target are unaffected.
- The time box and the criteria weights are for the founder to set when
  accepting this proposal.

## Related

- `queue-docs/ARCHITECTURE.md`: Repository Layout & Tech Stack, and
  Realtime transport (RESQ-9)
- `queue-docs/SECURITY.md`: MVP controls the DRF stack must match
- `queue-docs/decisions/DEC-012-backward-compatibility-before-customers.md`
- `queue-docs/decisions/DEC-011-product-owner-owns-strategic-direction.md`
- Jira: RESQ-58 (`queue-api-drf` scaffold), RESQ-59 (compose `drf` profile), RESQ-60 (this record)
