# Concurrent Payment Processing Review

A worked walkthrough demonstrating how to apply the expanded SPEC-005 reasoning model to a scenario involving concurrent execution, shared mutable state, credential rotation, and distributed retries.

This example shows the reasoning process for temporal trust boundaries, non-deterministic evidence, and the `[REQUIRES_EXTERNAL_EVIDENCE]` mechanism. → `specs/SPEC-005-agent-operational-behavior.md`

---

## Scenario

A FastAPI application processes payments. The system uses:

- A shared in-memory dictionary as a request cache
- JWT access tokens with refresh token rotation
- An external payment gateway called via HTTP with automatic retry on timeout
- SQLAlchemy async sessions for database access

The code under review:

```python
request_cache = {}  # shared mutable state

async def process_payment(user_id: str, amount: float, token: str):
    # Check cache
    if user_id in request_cache:
        return request_cache[user_id]

    # Refresh JWT if expired
    access_token = await refresh_jwt(token)

    # Call payment gateway with retry
    result = await call_payment_gateway(access_token, amount, retries=3)

    # Store in cache
    request_cache[user_id] = result

    # Record in database
    session = get_session()
    session.add(PaymentRecord(user_id=user_id, amount=amount))
    await session.commit()

    return result
```

## Step 1 — Perceive Input

The reviewer identifies three categories of signal:

**Spatial signals:**
- User input (user_id, amount) crossing into the application → `knowledge/input-validation.md`
- JWT token crossing into auth service → `knowledge/secrets-management.md`
- HTTP call crossing to external payment gateway → `knowledge/trust-boundaries.md`
- Database session crossing to persistent storage → `knowledge/error-handling-secure-failure.md`

**Temporal signals (SPEC-005 expanded):**
- `request_cache` is a shared mutable dictionary accessed by concurrent coroutines
- `get_session()` returns a session from a shared pool — multiple coroutines may receive the same session
- `refresh_jwt()` may be called concurrently with the same refresh token
- `call_payment_gateway()` with retries=3 may produce non-idempotent side effects

The reviewer flags temporal signals immediately. This system uses async execution with shared state, which creates temporal trust boundaries that do not exist in the spatial model alone.

## Step 2 — Construct Context

**Execution model classification:** The reviewer classifies the system's execution model before identifying boundaries. The system uses async FastAPI handlers (concurrent execution within a single process), a shared in-memory dictionary (shared mutable state), and an external payment gateway (distributed coordination). The execution model is concurrent-with-distributed. This classification determines that concurrency hypotheses (SPEC-005 expanded) are mandatory, not optional.

**Spatial boundaries:**
- Client → FastAPI handler (external input)
- Handler → JWT service (credential validation)
- Handler → Payment gateway (external service call)
- Handler → Database (persistent storage)

**Temporal boundaries (SPEC-005 expanded):**
- `request_cache`: shared mutable state. Concurrent coroutines read and write to the same dictionary without synchronization. This creates a temporal trust boundary where one coroutine's state may leak into another.
- `get_session()`: session pool. If sessions are not properly isolated, one coroutine's uncommitted transaction may be visible to another.
- `refresh_jwt()`: credential lifecycle. Two concurrent calls with the same refresh token create a race condition during rotation.

The reviewer notes: this system appears to isolate requests at the spatial level (each request has its own handler), but leaks state at the temporal level (shared cache, shared session pool, shared refresh token). → `knowledge/trust-boundaries.md`

## Step 3 — Generate Hypotheses

Before generating specific hypotheses, the reviewer models how state evolves during execution (SPEC-005 expanded). Four pieces of shared mutable state must be traced across execution time:

- `request_cache`: written by any coroutine, read by any coroutine. State is inconsistent between write and next read if interleaving occurs.
- `access_token` from `refresh_jwt()`: created at time T1, may be invalidated at time T2 by a concurrent rotation. State is time-dependent.
- `call_payment_gateway()` result: retried up to 3 times. Each retry may produce a different side effect. State accumulates across retries.
- `session` from `get_session()`: borrowed from a pool. State from a previous coroutine may persist if the session is not properly reset.

The reviewer generates hypotheses across all SPEC-005 categories:

**Spatial hypotheses (standard):**
- H1: SQL injection via user_id or amount parameters
- H2: JWT token stored insecurely or transmitted without TLS
- H3: Payment gateway response not validated (trust boundary crossing)
- H4: Error handling exposes internal state → `knowledge/error-handling-secure-failure.md`

**Temporal hypotheses (SPEC-005 expanded):**

- H5: **Context leakage via request_cache.** Concurrent coroutines may read stale or incorrect cached results from other users. If coroutine A writes `request_cache["user_A"]` and coroutine B reads before A finishes, B may receive A's payment result.

- H6: **Credential lifecycle race during JWT rotation.** If two concurrent requests present the same refresh token, one succeeds and rotates the token, the other receives an error because the old token is already invalidated. This creates a denial of service for the legitimate client. → `knowledge/secrets-management.md`

- H7: **Non-idempotent retry on payment gateway.** The `retries=3` parameter means the payment gateway may be called up to 3 times for the same payment. If the gateway does not enforce idempotency, the user may be charged multiple times. This extends to the orchestration level: if the caller retries the entire `process_payment` flow, the cache lookup may return a stale result while a fresh payment is processed, producing inconsistent state across the full flow.

- H10: **Security control undermined by retry behavior.** If the payment gateway implements rate limiting or fraud detection, the `retries=3` mechanism consumes the rate limit budget on each retry. A legitimate retry-after-failure pattern may exhaust the rate limit, blocking subsequent legitimate requests. The rate limiter is a security control that exists but is insufficient for the threat model of concurrent retries. → `specs/SPEC-005-agent-operational-behavior.md`

- H8: **Session pool state leakage.** If `get_session()` returns a session with uncommitted state from a previous coroutine, the current coroutine may see data from another user's transaction.

- H9: **Cache inconsistency on partial failure.** If the payment gateway call succeeds but the database commit fails, the cache stores a successful result while the database has no record. Subsequent requests for the same user_id return the cached success without re-processing.

## Step 4 — Evaluate Evidence

**H1 (SQL injection):** No evidence of parameterized queries in the visible code. → `knowledge/input-validation.md`
- Confidence: **Inferred.** The code does not show the database query implementation, so this cannot be confirmed from the snippet alone.

**H5 (Context leakage via request_cache):**
- The code shows `request_cache` as a module-level dictionary with no synchronization primitives (no locks, no atomic operations).
- Multiple coroutines access it concurrently without isolation.
- **Confirmed by absence of control.** The absence of synchronization primitives is itself evidence that context leakage is possible. → `specs/SPEC-005-agent-operational-behavior.md`

**H6 (Credential lifecycle race):**
- The code calls `refresh_jwt(token)` without any deduplication mechanism.
- If two concurrent requests carry the same refresh token, both call `refresh_jwt()` simultaneously.
- The reviewer cannot determine from static code whether the JWT service handles concurrent rotation gracefully.
- `[REQUIRES_EXTERNAL_EVIDENCE: RUNTIME_CONCURRENCY_LOGS]` — need to observe whether the JWT service serializes refresh requests or allows concurrent rotation of the same token.

**H7 (Non-idempotent retry):**
- The code calls `call_payment_gateway()` with `retries=3`.
- No idempotency key is visible in the function signature or the call.
- **Confirmed by absence of control.** The absence of an idempotency key means retries may produce duplicate charges. → `specs/SPEC-005-agent-operational-behavior.md`
- At the orchestration level: if the entire `process_payment` function is retried by the caller, the `request_cache` may return stale data while a fresh payment is processed concurrently. The full orchestration (check cache → refresh JWT → call gateway → cache result → commit to DB) is not idempotent.

**H10 (Security control undermined by retries):**
- The code includes a retry mechanism (`retries=3`).
- If the payment gateway enforces rate limiting, each retry consumes rate limit budget.
- The retry mechanism is a defensive control for transient failures, but it is insufficient for the threat model where concurrent retries exhaust rate limits or fraud detection budgets.
- **Confirmed by insufficient control.** The control (retry) exists, but it undermines another security control (rate limiting) under concurrent execution. → `specs/SPEC-005-agent-operational-behavior.md`

**H8 (Session pool state leakage):**
- The code calls `get_session()` without specifying isolation level.
- The reviewer cannot determine from static code whether the session pool enforces transaction isolation.
- `[REQUIRES_EXTERNAL_EVIDENCE: DATABASE_SESSION_ISOLATION_CONFIG]` — need to observe the session pool configuration to confirm whether sessions are properly isolated.

**H9 (Cache inconsistency on partial failure):**
- The code stores the result in `request_cache` before the database commit.
- If the commit fails, the cache retains the successful result.
- **Observed.** The ordering is visible in the code: cache write precedes database commit.

## Step 5 — Synthesize Decisions

The reviewer synthesizes findings with explicit confidence levels. Each finding is qualified as structurally guaranteed or time-dependent (SPEC-005 expanded).

**Finding 1 (Observed — H9): Cache inconsistency on partial failure.** Structurally guaranteed — the ordering is deterministic: cache write always precedes database commit. This finding holds regardless of execution interleaving.
The cache is written before the database commit. If the commit fails, the cache retains a successful result that was never persisted. Subsequent requests for the same user_id return the cached success without re-processing.
- **Impact:** Payment appears successful to the user but is not recorded in the database.
- **Recommendation:** Move the cache write after the database commit, or remove the cache entirely for payment operations. → `principles/minimal-change.md`

**Finding 2 (Confirmed by absence of control — H5): Context leakage via shared cache.** Structurally guaranteed vulnerability (no synchronization primitives), time-dependent manifestation (leakage occurs only under specific interleavings).
The `request_cache` dictionary has no synchronization primitives. Concurrent coroutines can read and write to it without isolation.
- **Impact:** One user's payment result may be returned to another user.
- **Recommendation:** Replace the shared dictionary with per-request state, or add explicit synchronization. → `principles/minimal-change.md`

**Finding 3 (Confirmed by absence of control — H7): Non-idempotent payment retry.** Structurally guaranteed vulnerability (no idempotency key), time-dependent manifestation (duplicate charges occur only when retries are triggered).
The payment gateway is called with `retries=3` and no idempotency key. Each retry may produce a separate charge.
- **Impact:** User may be charged up to 3 times for a single payment.
- **Recommendation:** Generate a unique idempotency key per payment and pass it to the gateway. → `principles/minimal-change.md`

**Finding 5 (Requires external evidence — H6): Credential lifecycle race.** Time-dependent — this finding depends on execution ordering. Two concurrent requests with the same refresh token may race during rotation. The static code does not reveal whether the JWT service handles this.
Two concurrent requests with the same refresh token may race during rotation. The static code does not reveal whether the JWT service handles this.
- **Impact:** Legitimate client may be locked out of their session.
- `[REQUIRES_EXTERNAL_EVIDENCE: RUNTIME_CONCURRENCY_LOGS]`

**Finding 6 (Requires external evidence — H8): Session pool state leakage.** Time-dependent — this finding depends on whether sessions retain state from previous coroutines. The session pool configuration is not visible.
The session pool configuration is not visible. Sessions may or may not enforce transaction isolation.
- **Impact:** One user's uncommitted transaction may be visible to another user.
- `[REQUIRES_EXTERNAL_EVIDENCE: DATABASE_SESSION_ISOLATION_CONFIG]`

## Step 6 — Construct Output

The reviewer structures the output in three tiers: → `specs/SPEC-005-agent-operational-behavior.md`

**Observations:**
- Cache is written before database commit (ordering visible in code).
- `request_cache` is a shared mutable dictionary with no synchronization.
- Payment gateway is called with retries=3 and no idempotency key.
- `refresh_jwt()` is called without deduplication.
- Session pool configuration is not visible in the reviewed code.

**Inferences:**
- Cache inconsistency exists: successful payments may be cached without database persistence.
- Context leakage is possible: concurrent coroutines share the cache without isolation.
- Duplicate charges are possible: retries without idempotency keys may produce multiple charges.
- Credential lifecycle race is possible but not confirmed: requires runtime observation.
- Session pool leakage is possible but not confirmed: requires configuration review.

**Risks:**
- Users may see payment results that were never persisted.
- Users may receive other users' payment results from the shared cache.
- Users may be charged multiple times for a single payment.
- Legitimate users may be locked out during concurrent JWT rotation.
- Users may see other users' uncommitted database state.

All findings are presented for human review before any action is taken. → `principles/human-in-the-loop.md`

---

## What This Example Demonstrates

- **Execution model classification** is performed before identifying boundaries. The reviewer classifies the system as concurrent-with-distributed, which makes concurrency hypotheses mandatory rather than optional. → `specs/SPEC-005-agent-operational-behavior.md`
- **State evolution modeling** traces each piece of shared mutable state across execution time, not just at a single snapshot. The reviewer identifies that request_cache, access_token, payment result, and session are state that evolves during execution. → `specs/SPEC-005-agent-operational-behavior.md`
- **Temporal trust boundaries** are identified alongside spatial ones. The shared cache, session pool, and refresh token create temporal boundaries that do not exist in a purely spatial analysis. → `specs/SPEC-005-agent-operational-behavior.md`
- **Concurrency hypotheses** are generated as a mandatory category. The reviewer does not wait for evidence of a race condition — the architecture itself generates hypotheses about context leakage, credential lifecycle races, and session pool leakage.
- **Idempotency** is evaluated as a security concern at both the operation level and the orchestration level. Retries without idempotency keys are flagged as a risk, and the full orchestration (check cache → refresh JWT → call gateway → cache result → commit) is evaluated for idempotency, not just individual operations.
- **Confirmed by absence of control** is applied when defensive mechanisms are missing. The absence of synchronization primitives, idempotency keys, and deduplication is treated as evidence, not as uncertainty.
- **Confirmed by insufficient control** is applied when a defensive mechanism exists but is inadequate for the specific threat model. The retry mechanism exists for transient failures, but undermines the rate limiting security control under concurrent execution. → `specs/SPEC-005-agent-operational-behavior.md`
- **Security controls undermined by retries** are evaluated as a distinct category. A control that exists can still fail under retry conditions — rate limiters, fraud detection, and account lockout counters may be consumed by retry storms, transforming a legitimate safety mechanism into a vulnerability. → `specs/SPEC-005-agent-operational-behavior.md`
- **`[REQUIRES_EXTERNAL_EVIDENCE]`** is emitted when static analysis is insufficient. The reviewer does not classify JWT rotation races or session pool leakage as "uncertain" — they flag them as requiring runtime observation.
- **Temporal qualification of findings** distinguishes structurally guaranteed findings from time-dependent ones. Findings about deterministic ordering (cache before commit) are structurally guaranteed. Findings about race conditions and interleaving are time-dependent. This prevents treating timing-dependent findings as universally reproducible, and prevents dismissing them as unlikely. → `specs/SPEC-005-agent-operational-behavior.md`
- **Proportional recommendations** are made. Each finding recommends the smallest safe change, not a rewrite. → `principles/minimal-change.md`
- **Human in the loop** is maintained. All findings are presented for review before action. → `principles/human-in-the-loop.md`