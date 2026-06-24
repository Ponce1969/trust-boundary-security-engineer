# Web Application Security Review — Temporal Boundaries and Infrastructure Trust

A worked walkthrough demonstrating SPEC-005 reasoning applied to a real-world Flet/Docker application audit. This example focuses on three categories that are underrepresented in typical security reviews: credential lifecycle races in password reset flows, network trust boundaries between Docker containers, and HTML template injection in server-rendered forms.

---

## Scenario

A family expense tracking application (Python/Flet + PostgreSQL + Docker) with:
- Password reset via email tokens stored in PostgreSQL
- An OCR microservice for ticket scanning (FastAPI, separate Docker container)
- A monitoring service (Guardian) that checks container health via Docker socket
- HTML forms rendered by f-string interpolation in the OCR microservice

The application has rate limiting on login and password reset, password hashing (Argon2), and session management via in-memory dictionaries.

## Step 1 — Perceive Input

**Spatial signals:**
- Browser → Flet app (user input, URLs with query parameters)
- Flet app → PostgreSQL (queries, financial data, password reset tokens)
- Flet app → Resend API (password reset emails with tokens in URL)
- Flet app → OCR microservice (ticket image upload, OCR results)
- OCR microservice → Ollama (local AI inference)
- Guardian → Docker socket (container status checks)
- Guardian → Discord webhook (alert notifications)

**Temporal signals (SPEC-005 expanded):**
- Password reset tokens: two-step find-then-mark-used creates a temporal window between validation and consumption
- In-memory session dict and rate limiter: shared mutable state accessed by concurrent Flet sessions
- Docker socket mount: Guardian container has access to host-level Docker control plane

The reviewer classifies execution model as **concurrent-with-distributed** (multiple async Flet sessions, separate Docker services, external API calls). This makes temporal boundary analysis mandatory.

## Step 2 — Construct Context

**Spatial trust boundaries:**
- Internet → App (Flet Web, untrusted)
- App → Database (semi-trusted, same Docker network)
- App → Resend (external, untrusted response)
- App → OCR microservice (internal Docker network, should be trusted)
- OCR microservice → Ollama (local, trusted)
- Guardian → Docker socket (host-level, privileged)

**Temporal trust boundaries (SPEC-005 expanded):**
- Password reset flow: token state transitions from "valid" to "used" with a non-atomic gap between SELECT and UPDATE
- Rate limiter state: per-process in-memory, resets on container restart
- Session state: per-process in-memory dict, not shared across replicas

## Step 3 — Generate Hypotheses

**H1: TOCTOU race in password reset token lifecycle.** The reset flow performs `find_valid_token` (SELECT) then `mark_used` (UPDATE) as separate operations. Between these operations, a concurrent request with the same token can also validate the token. Both requests pass the SELECT check, and both can change the password. The second UPDATE overwrites the first password change.

**H2: Unauthenticated OCR microservice exposes financial data.** The OCR API container is exposed on a separate port (8551) with no authentication. The `/pendiente/{familia_id}` endpoint allows any caller to retrieve OCR results by enumerating sequential integer IDs (familia_id starts at 1).

**H3: Reflected XSS in OCR upload form.** The `/upload-form` endpoint embeds `session_id` and `familia_id` URL parameters directly into HTML via f-string without escaping. An attacker can craft a URL with `session_id="><script>...</script>` to inject arbitrary JavaScript.

**H4: Guardian runs as root with Docker socket access.** The container runs as UID 0 with the Docker socket mounted read-only. If any dependency vulnerability is exploited, the attacker gains full host root access.

**H5: CORS wildcard on unauthenticated API.** The OCR microservice has `allow_origins=["*"]` with `allow_credentials=True`. Combined with no authentication, any website can query financial data from the OCR API.

**H6: In-memory rate limiting resets on container restart.** The rate limiter and session store are in-memory Python dictionaries. Container restarts clear all rate limit counters and sessions, allowing an attacker who triggers a lockout to wait for a redeployment.

## Step 4 — Evaluate Evidence

**H1 (TOCTOU race):** Confirmed by absence of control. The code performs two separate SQL operations without row-level locking or atomic conditional update. No `SELECT ... FOR UPDATE` or `UPDATE ... RETURNING` pattern is used. The temporal window between find and mark is non-zero and exploitable under concurrent requests.

**H2 (Unauthenticated OCR):** Observed. The code has zero authentication middleware. Port 8551 is mapped to the host. familia_id values are sequential integers starting from 1, making enumeration trivial.

**H3 (Reflected XSS):** Observed. The f-string embedding `value="{session_id}"` does not use `html.escape()`. The endpoint returns `text/html` content type, so browsers will execute any injected script.

**H4 (Root + Docker socket):** Observed. `user: "0:0"` and `docker.sock:ro` mount in docker-compose.yml. The Docker socket grants full host access when mounted into a root container.

**H5 (CORS wildcard):** Observed. `allow_origins=["*"]` with `allow_credentials=True` in the CORS middleware.

**H6 (In-memory state):** Observed. `_sessions` dict and `RateLimiter` use module-level dictionaries. No persistence or sharing mechanism exists.

## Step 5 — Synthesize Decisions

**Finding 1 (Observed — H1): Password reset TOCTOU race condition.** Structurally guaranteed. The two-step token validation is deterministic — the gap between SELECT and UPDATE always exists.

- **Impact:** A captured password reset token (from email logs, URL referers, browser history) can be reused by a concurrent request, allowing an attacker to reset the victim's password even after the legitimate user has already used the token.
- **Recommendation:** Replace the two-step find-and-mark with a single atomic `UPDATE ... SET used_at = NOW() WHERE token = :token AND used_at IS NULL AND expires_at > NOW() RETURNING *`. If a row is returned, the token was valid and is now consumed. If no row is returned, the token was already used or expired. This eliminates the race window entirely. → `principles/minimal-change.md`

**Finding 2 (Observed — H2 + H3): OCR microservice accessible without authentication and vulnerable to XSS.** Structurally guaranteed. No authentication exists and HTML templates embed untrusted input.

- **Impact:** Any network client can enumerate familia_id values (sequential integers) to read every family's financial OCR data. XSS allows session hijacking via crafted URLs.
- **Recommendation:** Remove the external port mapping from docker-compose.yml (the OCR service only needs to be accessible from the app container via the Docker network). Add `html.escape()` to all user-controlled parameters embedded in HTML templates. Restrict CORS from wildcard to the app's origin only. → `principles/minimal-change.md`

**Finding 3 (Observed — H4): Container escape via root + Docker socket.** Structurally guaranteed. Root with Docker socket access is equivalent to host root.

- **Impact:** Full host compromise — all secrets (.env, database credentials, API keys), all data, and persistent access.
- **Recommendation:** Remove `user: "0:0"` from docker-compose.yml. Use `group_add: ["<docker_gid>"]` to grant Docker socket access without root. Alternatively, use a Docker socket proxy (tecnativa/docker-socket-proxy) that restricts allowed API operations to read-only container inspection. → `principles/minimal-change.md`

**Finding 4 (Inferred — H6): In-memory rate limiting resets on container restart.** Time-dependent. The rate limiter works correctly within a single process lifetime but resets when the container restarts or replicates.

- **Impact:** An attacker who triggers a rate lockout can wait for a container restart to clear it. In multi-replica deployments, each replica has independent rate limiter state, allowing N times the intended budget.
- **Recommendation:** For the current single-container deployment, document this as accepted risk. If multi-replica deployment is planned, migrate to a shared store (Redis or database-backed rate limiting). → `principles/context-matters.md`

## Step 6 — Construct Output

**Observations:**
- Password reset uses two-step SELECT then UPDATE without atomic guarantee
- OCR API has zero authentication middleware and exposes sequential familia_id enumeration
- OCR upload form embeds URL parameters into HTML without escaping
- Guardian container runs as root with Docker socket mounted
- CORS middleware allows all origins on unauthenticated API
- Rate limiter and session state are in-memory process-level dictionaries

**Inferences:**
- A password reset token can be reused under concurrent access within its validity window
- OCR financial data (amounts, merchants, dates) is accessible to any network client
- Crafted URLs can execute arbitrary JavaScript in victim browsers on the OCR origin
- Guardian compromise leads to full host root access
- Rate limiting is effective within a single process lifetime but resets across restarts

**Risks:**
- Password reset token reuse allows account takeover even after legitimate use
- Financial PII exposure via unauthenticated OCR endpoint
- XSS enables session hijacking and cross-origin data theft
- Container escape to host root
- Rate limit bypass after container restart or redeployment

All findings presented for human review before action. → `principles/human-in-the-loop.md`

---

## What This Example Demonstrates

- **TOCTOU in credential lifecycle** extends the concurrent-payment-review pattern (H5/H6 in that example) to password reset flows. The same temporal boundary applies whenever a credential or token transitions between states (valid → used) without atomic guarantee. Credential lifecycle races are not limited to JWT rotation — they apply to any credential with a non-atomic state transition. → `specs/SPEC-005-agent-operational-behavior.md`

- **Docker network trust boundaries** illustrate that "internal" does not mean "trusted." The OCR microservice is inside the Docker network but should not trust all callers. Every service boundary, even inside a Docker compose network, requires authentication if it handles sensitive data. → `knowledge/trust-boundaries.md`

- **Least privilege at the infrastructure level** is not limited to application code. Running a container as root with Docker socket access is the infrastructure equivalent of running an application with admin privileges. The smallest safe change is to remove root and use group-based socket access. → `knowledge/least-privilege.md`

- **HTML template injection via f-strings** is a specific instance of the general input-validation principle. When server-generated HTML includes user-controlled parameters without escaping, the trust boundary is between the URL parameter and the HTML output. The defense is always context-appropriate escaping at the boundary, not only at the origin. → `knowledge/input-validation.md`

- **Control sufficiency evaluation** distinguishes between "the control is absent" and "the control is present but insufficient." Rate limiting exists in this application, but it is insufficient under concurrent access (resets on restart, per-process scope). The review must evaluate whether controls are adequate for the specific threat model, not merely present. → `principles/control-sufficiency.md`

- **Proportional recommendations** focus on the smallest safe change for each finding, not system rewrites. Removing an external port mapping, adding `html.escape()`, using `UPDATE ... RETURNING`, and removing `user: "0:0"` are all single-line or few-line changes that eliminate critical vulnerabilities. → `principles/minimal-change.md`