# Domain

## Core Domain

The system manages the relationship between waiting parties and physical restaurant tables.

The core domain is not "a line." It is:

> Matching arriving demand to available seating capacity over time.

## Core Entities

### Restaurant

Represents one operating location.

Key attributes:

- Name.
- Address.
- Time zone.
- Operating hours.
- WhatsApp sender configuration.
- QR code configuration.
- Table inventory.

### Customer

Represents a person joining the queue.

Key attributes:

- Name.
- Phone number.
- Optional notes.
- Optional preferences.

### Queue

Represents the active waitlist for a restaurant and service period.

Key attributes:

- Restaurant.
- Status.
- Opened at.
- Closed at.
- Current entries.

### Queue Entry

Represents one waiting party.

Key attributes:

- Customer.
- Party size.
- Status.
- Queue position.
- Joined at.
- Estimated wait.
- Called at.
- Arrived at.
- Seated at.
- Cancelled at.
- No-show at.
- Assigned table.
- Notes.

Confirmed at RESQ-6: the QR check-in creates the entry directly in `WAITING`
status via a restaurant-scoped public join (the QR encodes the restaurant,
and the open queue is resolved server-side at join time), then mints an
unguessable scoped token for the customer's status link (DEC-003).

Walk-ins (RESQ-11): staff add entries with name + party size only; the
Indian mobile number is optional (validated when provided) and the customer
row's phone is NULL for phone-less walk-ins. Walk-ins emit QUEUE_JOINED with
actor = staff and are treated identically to QR joins downstream. Notes are
deferred (V1).

### Table

Represents a physical table.

Key attributes:

- Restaurant.
- Label.
- Capacity.
- Section.
- Status.
- Current party.

### Section

Represents an area inside the restaurant, such as indoor, outdoor, family, bar, or private dining.

### Staff

Represents an employee or operator.

Key attributes:

- Restaurant.
- Name.
- Role.
- Permissions.
- Active status.

### Seating Event

Represents a state transition or operational action.

Events should be first-class data because they support analytics, debugging, audit trails, and future intelligence.

## Queue Entry State Machine

Primary path:

```text
WAITING -> ALMOST_READY -> CALLED -> ARRIVING -> ARRIVED -> SEATED
```

Staff shortcuts (RESQ-10, documented with the implementation): the staff
"call" action may advance WAITING straight to CALLED, and "mark arrived" may
advance CALLED straight to ARRIVED. ALMOST_READY remains the V1
almost-ready-alert stage and ARRIVING the V1 "I'm on my way" stage; the
matrix in queue-api (`app/domain/state_machines.py`) is the exact source of
truth and every allowed transition maps to a domain event.

Exception paths:

```text
WAITING -> CANCELLED
WAITING -> NO_SHOW
CALLED -> SKIPPED
CALLED -> NO_SHOW
ARRIVING -> ARRIVED
ARRIVING -> NO_SHOW
```

CANCELLED semantics (confirmed at RESQ-8): a customer can self-cancel only
from WAITING (never once called/seated); the transition appends
QUEUE_ENTRY_CANCELLED with actor = customer; cancelled entries drop out of
the active queue so remaining positions recalculate, and the staff
dashboard reflects the cancellation immediately.

## Table State Machine

Primary path:

```text
AVAILABLE -> OCCUPIED -> CLEANING -> AVAILABLE
```

Optional states for later:

```text
RESERVED
BLOCKED
COMBINED
```

Implementation notes (RESQ-13): AVAILABLE -> OCCUPIED is driven by seating
a party (RESQ-14) — staff cannot set OCCUPIED manually (the
occupied-iff-party invariant). Staff can manually mark OCCUPIED -> CLEANING
(party left) and CLEANING -> AVAILABLE (cleaning done); every change appends
TABLE_STATUS_CHANGED with the actor and before/after values.

## Domain Events

Important events:

- RESTAURANT_CREATED.
- QUEUE_OPENED.
- QUEUE_CLOSED.
- QUEUE_JOINED.
- QUEUE_ENTRY_CANCELLED.
- CUSTOMER_CALLED.
- CUSTOMER_ALMOST_READY_SENT.
- CUSTOMER_TABLE_READY_SENT.
- CUSTOMER_ON_THE_WAY.
- CUSTOMER_ARRIVED.
- CUSTOMER_SKIPPED.
- CUSTOMER_NO_SHOW.
- TABLE_CREATED.
- TABLE_STATUS_CHANGED.
- TABLE_ASSIGNED.
- CUSTOMER_SEATED.
- SEATING_COMPLETED.

## Matching Rules

MVP:

- Staff manually assigns parties to tables.
- System shows table capacity and status.
- Staff can override all operational choices.

Implemented at RESQ-14: the staff dashboard shows AVAILABLE tables whose
capacity fits the party (capacity >= party size) as guidance — no
recommendation engine (that is V1). Assignment is staff-confirmed only
(DEC-005) and the server re-checks capacity and availability at assign
time. Assigning a queue entry to a table sets the table OCCUPIED (T12
lifecycle) with the entry as its current party, records
`assigned_table_id` on the entry, and appends TABLE_ASSIGNED with the
acting staff member and the table state before/after. Unassign and
reassign are allowed until the party is seated; every step is logged as a
TABLE_ASSIGNED event (context action: assign / reassign / unassign), so
the audit trail reconstructs the full assignment lifecycle per table. A
terminal exit of an assigned entry (cancel / no-show / skip) releases the
hold automatically. Seated entries keep their table link and the table
stays OCCUPIED until staff mark it CLEANING (RESQ-13).

V1:

- System recommends compatible tables.
- Compatibility considers party size, table capacity, section, and table status.
- Staff confirms recommendation.

V2:

- System recommends next action based on waiting parties, table turnover, party sizes, arrival status, and historical data.

## ETA Rules

MVP:

- ETA can be based on simple queue position and average seating rate.
- Implemented at RESQ-7 as: `estimated wait = live position x average
  seating rate`, where the live position counts active entries ahead in the
  current queue and the average seating rate is the mean QUEUE_JOINED ->
  CUSTOMER_SEATED duration of the current service period (from the event
  log), falling back to 10 minutes per party before any party has been
  seated. Terminal entries (seated/cancelled/no-show/skipped) expose no
  position or ETA.

V1:

- ETA should consider party size and current table availability.

V2:

- ETA should consider historical wait data, day/time patterns, table turnover prediction, reservations, and party-to-table fit.

## Key Invariants

- A queue entry belongs to exactly one restaurant.
- A queue entry should have one active status at a time.
- A seated queue entry must have a seated timestamp.
- A table can be occupied by at most one active party unless table-combining is explicitly modeled.
- Staff overrides must be recorded as events.
- Customer-facing status should never expose internal operational confusion.

## Analytics Metrics (RESQ-18)

Exact event-derived definitions of the seven REQUIREMENTS.md MVP metrics.
All are computed from the append-only event log only (DEC-006), for a day
window `[day_start, day_end)` in the restaurant's timezone (converted to
UTC for queries):

1. **Waiting now** — parties in a non-terminal state at day end: for every
   QUEUE_ENTRY event before day end, take the latest per entry and read its
   `after.status`; count statuses in {WAITING, ALMOST_READY, CALLED,
   ARRIVING, ARRIVED} (a called-but-unseated party is still waiting).
2. **Average wait** — mean of `CUSTOMER_SEATED.occurred_at -
   QUEUE_JOINED.occurred_at` over parties seated that day (whole minutes).
3. **Longest wait** — max of the same durations.
4. **Tables occupied** — tables whose latest state event before day end
   (TABLE_CREATED / TABLE_ASSIGNED / TABLE_STATUS_CHANGED) has
   `after.status == OCCUPIED`.
5. **Parties served** — count of CUSTOMER_SEATED events in the window.
6. **No-shows** — count of CUSTOMER_NO_SHOW events in the window.
7. **Cancellations** — count of QUEUE_ENTRY_CANCELLED events in the window.

Exposed to owners via `GET /owner/restaurants/{id}/analytics/daily?date=`
(defaults to today in the restaurant timezone); owner role only.
