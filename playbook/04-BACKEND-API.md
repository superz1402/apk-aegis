# 04 — Backend/API playbook (the mobile-hostile-network edition)

Mobile networks drop, linger, and replay. Every rule below exists because its absence produced a real incident class. Claim classes: `FACT` = general distributed-systems standard; `SCAR-CLASS` = recurring documented incident pattern; `CANDIDATE` = AEGIS position.

## Writes: idempotency or regret

- Every non-idempotent write (create/submit/pay) carries a client-generated idempotency key. Retries without keys produce duplicate submissions — the duplicate-send incident class is documented repeatedly in practitioner threads (e.g. at-least-once send APIs producing 21 duplicate pairs in a day before reconciliation was added — merktop, r/tooling thread `82efda41`, 2026-10-06). `SCAR-CLASS`.
- Retry policy: exponential backoff + jitter, bounded attempts, **retry only on retryable signals** (timeouts, 5xx, 429-with-Retry-After). Retrying a 4xx that failed validation is not resilience, it is load.
- The client never decides success from a timeout alone. A timed-out request that *may have committed* must be resolved by a read (query by idempotency key) before retry. This is the "verify instead of assume" rule that generalizes to all agent integrations.

## Reads: timeouts, expiry, and the honest cache

- Every network call has a timeout. No defaults left implicit. `FACT`.
- Every cached datum answers: when does it expire, what invalidates it, what shows when it is stale and the network is gone (stale-while-revalidate is the honest default for read-heavy screens).
- Pagination enforced at the protocol level (cursor or page tokens); "load everything" endpoints become outages when the table grows.

## State machines across the wire (the zombie-killer section)

- Every server-driven state (submission, sync, job) has a terminal state **and a path to it from every intermediate state**. The scar: an event that stays "processing" forever because the finalization step died and nothing re-drives it — the state looks published client-side but resolves never. Walk the write path end to end: interruption before row N, after row N-1, during finalization (`SCAR-CLASS`, documented in agent ops threads 2026-10; the "verification zombie" pattern).
- Background sync (WorkManager) must be idempotent and re-drivable: on app start, re-derive incomplete work from durable state, never from memory.
- Push and pull reconciliation: the client re-queries on reconnect (network callback) instead of trusting its own model of what the server must have received.

## API contract hygiene

- Versioned API from day one (`/v1/`); the app tolerates unknown fields (forward-compatible parsing) — `CANDIDATE` default but cheap insurance.
- Error responses have a machine-readable code + human-safe message; the app maps codes to UX, not string-matching English.
- Auth: token refresh single-flight (one refresh at a time; parallel 401s must not stampede the refresh endpoint — the parallel-refresh race is a classic double-logout/double-refresh bug). `SCAR-CLASS`.
- Certificate pinning interacts with rotation — see 01-SECURITY pinning discipline before shipping pins.

## The acceptance list

1. Kill the server mid-write: client recovers to a defined state.
2. Airplane-mode the app mid-flow: UI communicates, no crash, no fake success.
3. Replay the last 3 requests blind: no duplicates created.
4. Expire the auth token mid-flow: single refresh, flow resumes.
5. Return garbage on one endpoint: parse error is caught, surfaced, and the rest of the app lives.
