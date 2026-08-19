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

Exception paths:

```text
WAITING -> CANCELLED
WAITING -> NO_SHOW
CALLED -> SKIPPED
CALLED -> NO_SHOW
ARRIVING -> ARRIVED
ARRIVING -> NO_SHOW
```

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

V1:

- System recommends compatible tables.
- Compatibility considers party size, table capacity, section, and table status.
- Staff confirms recommendation.

V2:

- System recommends next action based on waiting parties, table turnover, party sizes, arrival status, and historical data.

## ETA Rules

MVP:

- ETA can be based on simple queue position and average seating rate.

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
