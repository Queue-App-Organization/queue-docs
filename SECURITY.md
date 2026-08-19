# Security

## Security Goals

Protect customer phone numbers, restaurant operational data, staff access, WhatsApp messaging privileges, and event history.

The MVP handles personal data and live restaurant operations, so security must be designed from the start even if the feature set is small.

## Data Classification

### Personal Data

- Customer name.
- Customer phone number.
- Visit history.
- Queue and seating history.
- Special requirements or notes.
- WhatsApp message status.

### Restaurant Operational Data

- Table inventory.
- Queue activity.
- Wait-time metrics.
- No-show and cancellation metrics.
- Staff actions.

### Sensitive System Data

- Staff credentials.
- Session tokens.
- WhatsApp provider credentials.
- API keys.
- Database credentials.

## Authentication

MVP:

- Staff and owners must authenticate.
- Customers should not need accounts.
- Customer status pages should use unguessable tokens or scoped links.

Later:

- Multi-factor authentication for owners/admins.
- SSO only if enterprise customers justify it.

## Authorization

Suggested roles:

- Owner: full restaurant access.
- Manager: operational access plus analytics and staff management.
- Staff: queue and table operations.
- Read-only: analytics or monitoring only.

Authorization requirements:

- Staff can only access assigned restaurants.
- Customer links can only access that customer's queue entry status.
- Admin actions must be permission-checked server-side.

## Privacy

Principles:

- Collect the minimum data needed for waitlist operation.
- Do not require account creation for customers.
- Do not expose phone numbers unnecessarily in shared displays.
- Avoid using transactional WhatsApp flows for marketing until consent and compliance are explicitly handled.
- Retain customer data only as long as needed for operations, analytics, support, and legal requirements.

## WhatsApp Security

Requirements:

- Store provider credentials securely.
- Log delivery status without exposing full message content unless needed for support.
- Separate transactional messages from promotional messages.
- Handle opt-out or blocked delivery states.
- Do not send sensitive operational details to the wrong customer.

## Audit Logging

Record staff actions for:

- Calling a customer.
- Skipping a customer.
- Marking no-show.
- Cancelling an entry.
- Assigning a table.
- Changing table status.
- Changing restaurant configuration.
- Changing staff permissions.

Audit events should include:

- Actor.
- Restaurant.
- Entity.
- Action.
- Timestamp.
- Before/after values where appropriate.

## Abuse And Risk Cases

- Customer link guessing.
- Staff accessing another restaurant.
- Spam or accidental repeat WhatsApp messages.
- Host accidentally seating the wrong party.
- Malicious queue flooding through public QR links.
- Ex-staff retaining access.
- Data export or deletion without approval.

## MVP Controls

- Rate-limit public queue joins.
- Validate phone numbers.
- Use CSRF protection where applicable.
- Use secure, HTTP-only cookies for staff sessions.
- Use HTTPS in production.
- Encrypt secrets at rest through the hosting provider or secret manager.
- Keep database backups.
- Restrict production credentials to deployment/runtime systems.
- Log security-sensitive operations.

## Human Approval Gates

Require explicit human approval for:

- Destructive production migrations.
- Deleting customer data.
- Changing staff permission models.
- Changing WhatsApp templates that affect compliance.
- Payment or pricing changes.
- External communications.
- Major architecture changes.
