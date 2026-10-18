# DEC-019: JWT staff authentication for queue-api-drf

Date: 2026-10-18
Status: Accepted (founder, 2026-10-18), with option (b): a per-request `sid` check
Relates to: DEC-018 (DRF evaluation), DEC-003 (no customer accounts), SECURITY.md (Authentication, MVP Controls), RESQ-68

## Context

`queue-api-drf` authenticates staff the same way as `queue-api`:

- An HttpOnly `queue_session` cookie holds a 256-bit random session id. Only its SHA-256 hash is stored, in `staff_sessions`.
- Revocation is immediate and server-side: logout, password reset and deactivation all take effect on the next request.
- A CSRF origin middleware protects unsafe methods.

The owner asked for JWT staff auth in `queue-api-drf` (RESQ-67 point 4, split out to RESQ-68). The change is security-sensitive and changes the auth contract, so it needs this record before any build work. The owner chose the four options marked below on 2026-10-18.

## Decision

### Scope: `queue-api-drf` only (owner choice)

- JWT ships in `queue-api-drf` as part of the DEC-018 evaluation.
- `queue-api` keeps session cookies.
- Whether `queue-api` follows is a later, separate decision, taken only if DEC-018 adopts DRF or the owner asks.
- From then on, `queue-api-drf`'s auth contract differs from `queue-api`'s. The parity contract (`contract/queue-api-openapi.json`) gets a documented auth delta, and every other operation stays identical.

### 1. Transport: `Authorization: Bearer` access token (owner choice)

- **Access token:** a JWT, returned in the login and refresh response bodies. queue-web keeps it **in memory only**, never in `localStorage` or `sessionStorage`, and sends it as `Authorization: Bearer <jwt>`. A page reload gets a new one through the refresh call below.
- **Refresh token:**
  - Opaque, 256-bit random.
  - Kept in an HttpOnly, Secure cookie, `queue_refresh`, scoped to `Path=/auth`. Same SameSite rules as today's cookie: `None` in production (cross-site SPA) and `Lax` locally.
  - It is sent only to `/auth/refresh` and `/auth/logout`.
- **Dashboard stream (SSE):** `EventSource` can't send headers, so the client gets a stream ticket first.
  - It calls `POST /auth/stream-ticket` (Bearer-authenticated, restaurant-scoped) and gets a **single-use ticket** with a 30-second TTL.
  - It then opens `GET /staff/restaurants/{id}/stream?ticket=...`.
  - Tickets are opaque, 256-bit, stored hashed, and bound to the staff member, restaurant and refresh family.
  - A ticket is consumed **atomically**: a single update that succeeds only while the ticket is still unused. Two connections can never share one ticket.
  - A leaked URL is worthless once used or expired. The stream keeps its RESQ-69 per-tick check, re-checking the ticket's refresh family and the staff row.
  - **Reconnects:** after a network blip, `EventSource` retries the same, already-used URL, gets a 401 and goes `CLOSED`. The client must then fetch a fresh ticket and reconnect with backoff (see section 6).
  - **Access logs:** the ticket appears in uvicorn and Fly access logs, because it is in the query string. That is accepted: by the time anyone reads a log, the ticket is used or past its 30-second TTL.
- **CSRF:**
  - Bearer-authenticated requests don't need CSRF protection: browsers never attach the header on their own.
  - The two cookie-bearing endpoints (`/auth/refresh`, `/auth/logout`) keep the existing CSRF origin check.
  - The middleware stays global. It is cheap, and it covers any future cookie use.
- **XSS:**
  - Script running on the page can read the in-memory access token. A copied token works elsewhere for 5 minutes at most, or until the family is revoked.
  - Script cannot read the refresh cookie, but it can still call `/auth/refresh` from the page, because the browser sends the cookie along. So while an XSS is live in the page, it can keep minting access tokens.
  - JWT doesn't fix XSS. queue-web keeps its CSP and escaping rules (SECURITY.md).
- **Caching:** login, refresh and stream-ticket responses send `Cache-Control: no-store`.

### 2. Revocation: short access tokens plus rotating refresh tokens (owner choice)

- **Access token lifetime:** 5 minutes, set by `JWT_ACCESS_TTL_SECONDS` (default 300).
- **Refresh families:**
  - Each login creates a refresh **family**.
  - The existing `staff_sessions` table is reused as the family table. It already has `staff_user_id`, `expires_at` (absolute lifetime, 12 hours by default) and `revoked_at`.
  - Reusing it keeps logout, the revoke-all in password reset, the RESQ-67 soft-delete revocation and the RESQ-69 stream check working unchanged.
  - Each refresh token is a hashed row in a new table, linked to its family. A refresh consumes the presented token and issues a new one (rotation).
- **Reuse detection, with a grace window:**
  - Several tabs share one refresh cookie, and parallel 401s can refresh at the same moment. For **30 seconds** after a rotation, presenting the previous token therefore returns the family's current pair instead of revoking it.
  - Any other reuse of a used token revokes the whole family, because it has been stolen or replayed.
  - The client also refreshes single-flight (section 6).
- **What each event does:**

  | Event | Effect |
  |---|---|
  | Logout | Revokes the family and clears the cookie. |
  | Password reset | Revokes every family of that staff member. |
  | Deactivation or soft delete (RESQ-67) | Every refresh fails. |

- **Per-request check (owner chose option (b) at acceptance):**
  - Each request loads the staff row by `sub`; DRF needs `request.user` anyway.
  - It also checks that the token's `sid` family is neither revoked nor expired, with one more indexed read.
  - So logout, password reset, deactivation and soft delete all cut access **immediately**, as they do today. The SSE stream also re-checks both on every tick.
  - The other option, keeping a window of up to 5 minutes for logout and reset to avoid that read, was declined.
- **SECURITY.md:** revocation stays immediate, so no change to that wording is needed.

### 3. Signing: HS256 with `kid` (owner choice)

- Uses PyJWT.
- **Keys:** `JWT_SIGNING_KEYS`, a JSON map of `kid` to secret, injected through the environment only. Each secret is at least 256 bits. `JWT_ACTIVE_KID` selects the key used to sign.
- **Verification:** accepts any listed `kid` and pins the algorithm to `HS256`. `alg: none` and algorithm confusion are rejected.
- **Rotation:** add a new `kid`, switch `JWT_ACTIVE_KID`, and remove the old key after one access-token lifetime.
- **Startup:** fails closed if the keys are missing outside `APP_ENV=local`, like `SECRET_KEY` does today.
- **Claims:**

  | Claim | Value |
  |---|---|
  | `iss` | `queue-api-drf` |
  | `aud` | `queue-web` |
  | `sub` | staff id |
  | `rid` | restaurant id |
  | `role` | staff role |
  | `sid` | refresh family id |
  | `iat` | issued at |
  | `exp` | expiry |
  | `jti` | token id |

  Leeway is 30 seconds. Tokens carry no PII (no name, email or phone).

- **Why not a keypair:** only `queue-api-drf` issues and verifies tokens, so EdDSA would add key handling for no gain. Revisit if another service ever needs to verify tokens without being able to mint them.

### 4. Customer tokens: unchanged

The customer scoped tokens (DEC-003, status links) stay **hashed opaque tokens**. They are long-lived links that must stay individually revocable, which JWT does not give for free. They are not staff auth.

### 5. Contract changes (`queue-api-drf` only)

| Endpoint | Change |
|---|---|
| `POST /auth/login` | Body unchanged. The response adds `access_token`, `token_type: "Bearer"` and `expires_in`, and sets the `queue_refresh` cookie instead of `queue_session`. |
| `POST /auth/refresh` (new) | Rotates the refresh cookie and returns a new access token. 401 when the token is invalid, expired, revoked or reused. |
| `POST /auth/logout` | Revokes the family and clears the cookie. Works with only the refresh cookie present. |
| `POST /auth/stream-ticket` (new) | Bearer-authenticated. Returns `{ticket, expires_in}`. |
| `GET /staff/restaurants/{id}/stream` | Authenticates with `?ticket=` instead of a cookie. |

All other staff endpoints authenticate with Bearer. The 401 and 403 semantics and the `{"detail": ...}` error contract stay the same. A missing, invalid or expired access token returns 401.

### 6. queue-web

- `queue-web` keeps talking to `queue-api` (cookies). A linked FE ticket tracks the client work needed if this design is adopted:
  - **Token:** an in-memory access token, sent as Bearer by the typed client in `src/api.ts`.
  - **Refresh:** silent refresh on a 401 or on a timer, **single-flight**. Only one refresh runs at a time, shared across tabs (for example with the Web Locks API or a BroadcastChannel).
  - **Stream:** fetch a stream ticket before opening `EventSource`. When the stream goes `CLOSED`, fetch a fresh ticket and reconnect with backoff. Redirect to the landing page only if the refresh itself fails.
- To test against the DRF backend before then, use the API directly, or a queue-web branch pointed at `queue-api-drf`.

## Consequences

- **Data:** `staff_sessions` stays and becomes the refresh family table. Two new tables are added, refresh tokens and single-use stream tickets. All of them store hashes only, and no token material is logged. Old session cookies stop working; pre-customer, no migration path is needed (DEC-012).
- **Revocation:** stays immediate for logout, password reset, deactivation and soft delete, through the per-request `sid` and staff checks. The cost is two indexed reads per request.
- **Approval gates:** the mandatory security pass applies to the build PR, and this record must be Accepted first (AGENTS.md: security-sensitive and major architecture change).

## Alternatives considered

- **Tokens in HttpOnly cookies:** the smallest client change, and no stream ticket needed. Not chosen (owner choice).
- **Fetch-based SSE with Bearer:** this would drop the stream ticket but rewrite the client's stream handling. Not chosen.
- **Per-request revocation epoch:** this would make revocation immediate everywhere, but it gives up most of what JWT is for. Not chosen. Deactivation is immediate anyway, through the per-request staff lookup.

## Related

- Jira: RESQ-68 (build), RESQ-67 (layout this builds on), RESQ-69 (stream per-tick check)
- `queue-docs/SECURITY.md`, `queue-docs/ARCHITECTURE.md`
- `queue-docs/decisions/DEC-018-evaluate-django-rest-framework-backend.md`
