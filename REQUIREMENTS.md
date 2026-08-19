# Requirements

## MVP Goal

Replace paper, manual WhatsApp, and memory-based restaurant waiting lists with:

```text
QR + Queue + Tables + WhatsApp + Basic Analytics
```

## MVP Functional Requirements

### Customer

- Customer can scan a restaurant QR code and open a mobile web check-in flow. (done — RESQ-6)
- Customer can join with name, Indian phone number, and party size. (done — RESQ-6)
- Customer can see queue position, estimated wait, and live status. (done — RESQ-7)
- Customer can cancel their queue entry. (RESQ-8)
- Customer receives WhatsApp confirmation after joining. (RESQ-15)
- Customer receives WhatsApp table-ready notification. (RESQ-16)
- Customer does not need to create an account or install an app. (done — RESQ-6)

### Staff

- Staff can log in.
- Staff can see the live waiting list.
- Staff can add walk-ins manually.
- Staff can see party size, joined time, wait duration, and status.
- Staff can call a party.
- Staff can mark a party arrived.
- Staff can mark a party seated.
- Staff can skip a party.
- Staff can cancel a party.
- Staff can mark a party as no-show.
- Staff can assign a party to a table.

### Tables

- Owner or staff can create tables.
- Each table has a capacity.
- Each table has a status: available, occupied, or cleaning.
- Staff can update table status.
- Staff can assign a queue entry to a table.

### Owner

- Owner can set up restaurant name.
- Owner can configure operating hours.
- Owner can create table inventory.
- Owner can generate a QR code.
- Owner can view basic daily analytics.

### Analytics

MVP analytics should include:

- Waiting now.
- Average wait.
- Longest wait.
- Tables occupied.
- Parties served.
- No-shows.
- Cancellations.

## V1 Requirements

Add after real usage from 5-10 restaurants:

- "I'm on my way" customer action.
- Almost-ready WhatsApp alerts.
- Seating preferences.
- Special requirements.
- Reorder queue.
- Customer notes.
- Staff accounts.
- Staff permissions.
- Search customer.
- Restaurant sections.
- Table combining.
- Better party-to-table matching.
- Historical ETA.
- Peak-hour analysis.
- Automated reminders.

## V2 Requirements

Move toward restaurant seating optimization:

- Dynamic ETA.
- Table turnover prediction.
- Intelligent seating recommendation.
- Reservations.
- Reservation and walk-in optimization.
- Customer history.
- Repeat customer recognition.
- POS integration.
- Advanced analytics.
- Multi-location management.
- API and integrations.

## Explicit Non-Requirements For MVP

Do not build initially:

- Native customer app.
- Facial recognition.
- Biometric check-in.
- Physical token printers.
- Advanced AI chatbot.
- Complex kiosk hardware.
- Staff gamification.
- 1,000+ integrations.
- Complex multi-counter architecture.
- POS integration.
- Payments.
- Ordering.
- Inventory.
- Kitchen management.
- Loyalty.

## Non-Functional Requirements

### Usability

- Customer check-in must be possible in under one minute.
- Restaurant setup should be possible in under 10 minutes.
- Staff dashboard must be usable on a busy floor.
- Core actions should be one or two taps/clicks.

### Reliability

- Queue and table state must not be lost.
- WhatsApp delivery failures must be visible to staff.
- Staff must be able to recover from missed or failed notifications.

### Performance

- Staff dashboard should update in near real time.
- Customer status page should load quickly on mobile connections.
- Core operational actions should complete quickly enough for live service.

### Auditability

Events should be recorded for important operations:

- Queue joined.
- Customer called.
- Customer arrived.
- Customer skipped.
- Customer cancelled.
- Table assigned.
- Customer seated.
- Table status changed.

### Localization

- Indian phone number handling is required.
- Indian date and time formatting is required.
- Hindi and regional-language support are V1/V2 candidates depending on target market.

## Definition Of Done

A ticket is done only when:

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
