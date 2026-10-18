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
- Password recovery is self-service via email OTP (RESQ-52 backend,
  RESQ-53 frontend): identity is the staff email, 6-digit code, 10-minute
  TTL, single-use, max 10 attempts, neutral responses (no account
  enumeration), per-identity and per-IP rate limits. Only OTP/reset-token
  hashes are stored. A successful reset sets a bcrypt password, revokes
  all sessions, and appends a PASSWORD_RESET audit event with no secrets
  in the payload. Email sends via Resend (key + from address env-injected;
  log provider in dev; production needs a verified domain, see RESQ-51).
  WhatsApp OTP to the phone on file exists but stays disabled behind
  `password_reset_whatsapp_enabled`.
- `queue-api-drf` staff auth is JWT (DEC-019, RESQ-68). `queue-api` keeps
  the HttpOnly session cookie.
  - **Access token:** HS256, 5 minutes, sent as `Authorization: Bearer`.
    It carries ids only (no PII) and a `kid`. Keys are env-injected
    (`JWT_SIGNING_KEYS`) and startup fails without them outside local.
  - **Refresh token:** opaque, rotating, in an HttpOnly cookie on `/auth`.
    Hashes only are stored. Reuse more than 30s after rotation revokes the
    whole login. The two cookie-bearing endpoints keep the CSRF origin check.
  - **Revocation:** every request re-checks the login and the staff row, so
    logout, password reset, deactivation and soft delete cut access
    immediately, as with the cookie.
  - **Dashboard stream:** authenticated by a single-use, 30-second, hashed
    ticket. It may appear in access logs, which is accepted because it is
    used or expired by the time anyone reads them.
  - **XSS:** script running in the page can use the in-memory access token
    and can call refresh, so CSP and escaping still matter.

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

Soft delete (queue-api-drf, RESQ-67):

- Soft delete applies only to configuration and staff records: tables, sections, staff users and restaurants. It sets `is_deleted` and `deleted_at`, and the row, with any staff name, phone and email, stays in the database.
- Customer records (customers, queue entries, access tokens, notifications) are never soft-deleted. A delete removes them, so "delete" never silently retains customer PII.
- A soft-deleted staff member, or any staff of a soft-deleted restaurant, can no longer log in. The soft delete also revokes their sessions and burns outstanding OTPs and reset tokens, so restoring the row never revives an old credential.
- Soft-deleting a restaurant soft-deletes its staff, tables and sections. Its customer records are kept and are not soft-deleted; how long they are retained is an open founder decision.
- Removing a soft-deleted row for good (`hard_delete()`), or deleting customer data, remains a human approval gate (see Human Approval Gates).

## WhatsApp Security

Requirements:

- Store provider credentials securely.
- Log delivery status without exposing full message content unless needed for support.
- Separate transactional messages from promotional messages.
- Handle opt-out or blocked delivery states.
- Do not send sensitive operational details to the wrong customer.

Sender configuration (confirmed at RESQ-15): the WhatsApp provider is
selected at runtime from environment variables only
(`WHATSAPP_PROVIDER=log|twilio`, plus `TWILIO_ACCOUNT_SID`,
`TWILIO_AUTH_TOKEN`, `TWILIO_WHATSAPP_FROM`). Credentials are injected at
deploy time and never committed, echoed or logged. `log` is the
credential-free default for local/dev: it records the send outcome without
the phone number or message content. Every transactional send is recorded
on `notification_events` (channel, template, status, provider message id,
timestamps); a customer with a BLOCKED or OPTED_OUT delivery receives no
further sends until staff resets the state.

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
