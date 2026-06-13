# Authentication Bypass Review

A worked walkthrough demonstrating how to apply the Security Engineering Skill to a realistic authentication bypass scenario.

This example shows the reasoning process, not just the conclusion. It follows the operational behavior model: perceive input, construct context, generate hypotheses, evaluate evidence, synthesize decisions, and construct output. → `specs/SPEC-005-agent-operational-behavior.md`

---

## Scenario

A web application implements password-based authentication for its admin panel. The authentication logic uses a database query to verify credentials. After login, the application sets a session cookie and redirects to the admin dashboard.

## Step 1 — Perceive Input

The reviewer identifies the authentication code path:

```
POST /admin/login
  → query: SELECT * FROM users WHERE username = '{input}' AND password = '{input}'
  → if result: set session cookie, redirect to /admin/dashboard
  → if no result: return "Invalid credentials"
```

Initial observations:

- User input is inserted directly into the database query without parameterization.
- The session cookie is set based solely on whether the query returns a result.
- No rate limiting or account lockout mechanism is visible.

## Step 2 — Construct Context

The reviewer identifies trust boundaries and data flows: → `knowledge/trust-boundaries.md`

- **Boundary 1:** User input → application. External data enters the system here. → `knowledge/input-validation.md`
- **Boundary 2:** Application → database. The query crosses from application logic to data storage.
- **Boundary 3:** Database → application. The result crosses back. If the query is manipulated, the database may return data the application did not intend to expose.

The reviewer notes the context:

- This is an admin panel, not a public-facing user login. The impact of a bypass is higher.
- The application uses password-based authentication, which makes credential validation the primary security control. → `knowledge/least-privilege.md`

## Step 3 — Generate Hypotheses

Based on the input and context, the reviewer generates hypotheses:

**Hypothesis A: SQL Injection bypass.** If the username field accepts `' OR 1=1 --`, the query returns all rows, and the application authenticates without verifying a specific password. → `knowledge/input-validation.md`

**Hypothesis B: Session hijacking after bypass.** Even if SQL injection is not possible, the session cookie may lack proper protections (secure flag, HttpOnly, SameSite), enabling session theft after authentication. → `knowledge/trust-boundaries.md`

**Hypothesis C: No brute force protection.** Without rate limiting, an attacker can attempt credentials indefinitely, making weak passwords vulnerable.

The reviewer does not commit to a single hypothesis. Each is evaluated in parallel.

## Step 4 — Evaluate Evidence

**Evidence for Hypothesis A (SQL Injection):**

- The query concatenates user input directly into the SQL string. → `knowledge/input-validation.md`
- No parameterized query or prepared statement is visible.
- This is a directly observable fact. Confidence: **Observed.** → `specs/SPEC-005-agent-operational-behavior.md`

**Evidence for Hypothesis B (Session weakness):**

- No evidence of cookie flags is visible in the code path shown. This is absence of evidence, not evidence of absence.
- The reviewer flags this for verification. Confidence: **Uncertain.** → `principles/evidence-over-assumptions.md`

**Evidence for Hypothesis C (No brute force protection):**

- No rate limiting code is visible in the authentication path. This is also absence of evidence.
- The reviewer notes this for verification but does not inflate severity without confirming it is missing. Confidence: **Uncertain.**

## Step 5 — Synthesize Decisions

The reviewer synthesizes findings with explicit confidence levels:

**Finding 1 (Observed):** The authentication query is vulnerable to SQL injection. User input is concatenated directly into the database query without parameterization. An attacker can bypass authentication by injecting SQL into the username field.

- **Impact:** Full authentication bypass for the admin panel.
- **Context:** This is an admin panel, so the impact is higher than a standard user login. → `principles/context-matters.md`

**Finding 2 (Uncertain):** Session cookies may lack security flags. This needs verification before it can be stated as a finding.

**Finding 3 (Uncertain):** No rate limiting or account lockout is visible. This needs verification before it can be stated as a finding.

The reviewer does not escalate Findings 2 and 3 to the same level as Finding 1. Evidence for Finding 1 is directly observable. Findings 2 and 3 require verification before they can be treated as confirmed. → `principles/evidence-over-assumptions.md`

## Step 6 — Construct Output

The reviewer structures the output in three tiers: → `specs/SPEC-005-agent-operational-behavior.md`

**Observations:**

- User input is concatenated directly into the SQL query without parameterization.
- No rate limiting is visible in the authentication path.
- No session cookie flags are visible in the available code.

**Inferences:**

- The query is vulnerable to SQL injection, which enables authentication bypass.
- Missing session flags would allow session hijacking, but this requires verification.
- Missing rate limiting would allow brute force attacks, but this requires verification.

**Risks:**

- An attacker can bypass admin authentication by injecting SQL into the username field.
- If session cookies lack security flags, stolen cookies enable persistent unauthorized access.
- Without rate limiting, weak admin passwords can be brute-forced.

## Recommended Mitigations

The reviewer recommends mitigations proportional to each finding: → `principles/minimal-change.md`

**For Finding 1 (SQL Injection):** Use parameterized queries. This is the smallest safe change that eliminates the vulnerability. Do not rewrite the entire authentication module — replace the concatenation with a parameterized query. → `principles/minimal-change.md`

**For Finding 2 (Session flags):** Verify before recommending. Check the session cookie configuration. If flags are missing, add Secure, HttpOnly, and SameSite attributes. This is a targeted configuration change, not an architectural redesign.

**For Finding 3 (Rate limiting):** Verify before recommending. Check if rate limiting exists elsewhere in the application. If it does not, add a rate limiter to the authentication endpoint. This is a single, focused addition.

All findings are presented for human review before any action is taken. → `principles/human-in-the-loop.md`

---

## What This Example Demonstrates

- **Trust boundaries** are identified before hypotheses are generated. The analysis starts with context, not with a list of vulnerabilities. → `knowledge/trust-boundaries.md`
- **Input validation** problems are traced to the trust boundary where external data crosses into the system. → `knowledge/input-validation.md`
- **Evidence is separated from inferences.** Observed facts are reported with higher confidence than uncertain findings. Inconclusive findings are verified before they are escalated. → `principles/evidence-over-assumptions.md`
- **Context determines impact.** The same SQL injection in a public user login would have different impact than in an admin panel. The reviewer adjusts severity based on context. → `principles/context-matters.md`
- **Mitigations are proportional.** Each recommendation is the smallest safe change that addresses the finding. No rewrite, no overengineering. → `principles/minimal-change.md`
- **Uncertainty is communicated explicitly.** The reviewer states what is known, what is inferred, and what needs verification. → `principles/evidence-over-assumptions.md`
- **Findings are presented for human review.** The skill informs and supports — it does not act autonomously. → `principles/human-in-the-loop.md`
- **The explanation teaches, not alarms.** The finding is clear, calm, and actionable. No inflated severity, no alarmism. → `principles/education-over-fear.md`