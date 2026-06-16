# TalkTalk Data Breach Case Study (2015)

A worked walkthrough demonstrating how the Trust Boundary reasoning model would have detected a real-world breach — the TalkTalk attack that exposed 157,000 customer records using SQL Injection on a legacy web page.

This example follows the same reasoning process as the authentication bypass and concurrent payment reviews, applying each step of the SPEC-005 model to a documented incident. → `specs/SPEC-005-agent-operational-behavior.md`

---

## Background

In October 2015, the British telecommunications company TalkTalk suffered a data breach. A 17-year-old attacker accessed customer data — including names, addresses, dates of birth, email addresses, and in some cases financial information — by exploiting a SQL Injection vulnerability on a legacy web page that had not been updated or removed.

The breach resulted in a record fine from the Information Commissioner's Office, significant reputational damage, and the loss of approximately 101,000 customers. The attack technique was not novel. SQL Injection had been documented in the OWASP Top 10 for over a decade.

---

## Step 1 — Perceive Input

The reviewer examines the system with the signals available before the breach:

**Spatial signals:**
- Legacy web pages still active and connected to the production customer database → `knowledge/trust-boundaries.md`
- User input (search fields, form parameters) reaching database queries on legacy code → `knowledge/input-validation.md`
- Customer data stored in a database accessible from web-facing applications → `knowledge/least-privilege.md`

**Ambiguity detected:**
- It is unclear from the outside whether legacy pages use parameterized queries or string concatenation. This ambiguity must be resolved before the hypothesis can be confirmed or denied. → `specs/SPEC-005-agent-operational-behavior.md`

The reviewer does not dismiss legacy pages as low-priority noise. Pages that are connected to production data are security-relevant regardless of their age.

## Step 2 — Construct Context

**Execution model classification:** The system operates with sequential control flow for individual requests. Each request builds a database query, sends it to the database, and returns a result. The execution model is sequential. Concurrency hypotheses are not mandatory, but spatial trust boundary hypotheses are. → `specs/SPEC-005-agent-operational-behavior.md`

**Spatial boundaries:**
- Boundary 1: User browser → legacy web page (external input crossing into the application)
- Boundary 2: Application → production database (application logic crossing to data storage)
- Boundary 3: Production database → application (query results crossing back to the application and potentially to the user)

The critical boundary is Boundary 2. User input crosses from an untrusted context (the browser) through the application, directly into a database query. This is the data-command boundary where input validation failures become exploitable. → `knowledge/input-validation.md`

The reviewer notes: this system has legacy components connected to production data. Legacy code is often written before current security practices were standard. It is not an exception to security rules — it is where security rules matter most.

## Step 3 — Generate Hypotheses

The reviewer generates hypotheses across the relevant categories:

**Spatial hypotheses (input validation at trust boundaries):**

- H1: **SQL Injection via string concatenation.** If the legacy pages construct database queries by concatenating user input into SQL strings, an attacker can close the intended query and append commands. The database engine cannot distinguish between the developer's commands and the attacker's data because they travel in the same channel. → `knowledge/input-validation.md`

- H2: **Cross-Site Scripting via unescaped output.** If query results containing user input are rendered in web pages without escaping, an attacker can inject script tags that execute in other users' browsers. → `knowledge/input-validation.md`

- H3: **Excessive data exposure via permissive queries.** If legacy pages query tables that include sensitive columns (financial data, password hashes) rather than only the columns needed for the page's function, a successful injection can expose more data than the page was designed to display. → `knowledge/least-privilege.md`

**Credential and access hypotheses:**

- H4: **Over-privileged database credentials.** If the application connects to the database with an account that has broader permissions than the page requires (e.g., write access for a page that only reads), a successful injection gains more capability than the page's function justifies. → `knowledge/least-privilege.md`, `knowledge/secrets-management.md`

The reviewer does not commit to a single hypothesis. Each is evaluated in parallel.

## Step 4 — Evaluate Evidence

**H1 (SQL Injection via string concatenation):**
- Legacy pages were written before current security practices were standard.
- The TalkTalk breach investigation confirmed that the vulnerable pages used string concatenation to build SQL queries from user input.
- No parameterized queries or prepared statements were in use on the affected pages.
- **Confirmed by absence of control.** The absence of parameterized queries is itself evidence that injection is possible. → `specs/SPEC-005-agent-operational-behavior.md`

**H2 (Cross-Site Scripting via unescaped output):**
- Legacy pages that do not validate input are unlikely to properly escape output.
- No direct evidence of XSS in the available information, but the same trust boundary failure that enables SQL Injection also enables XSS.
- Confidence: **Inferred.** The absence of input validation makes XSS likely, but it is not confirmed without observing output handling. → `principles/evidence-over-assumptions.md`

**H3 (Excessive data exposure via permissive queries):**
- The breach exposed customer names, addresses, dates of birth, email addresses, and financial information — far more than any single page should need to display.
- This indicates that database queries accessed more columns than necessary, or that the application had access to tables containing sensitive data that the page did not need.
- Confidence: **Observed.** The scope of data exposed is documented in the breach report. → `knowledge/least-privilege.md`

**H4 (Over-privileged database credentials):**
- The attacker was able to extract data from multiple tables, suggesting the application's database connection had broad read permissions.
- If the application had used credentials scoped to only the tables needed for each page, the damage would have been contained.
- Confidence: **Inferred.** The breadth of data access suggests over-privileged credentials, but the specific database permissions are not publicly documented.

## Step 5 — Synthesize Decisions

The reviewer synthesizes findings with explicit confidence levels. Each finding is qualified as structurally guaranteed or time-dependent.

**Finding 1 (Confirmed by absence of control — H1): SQL Injection via string concatenation.** Structurally guaranteed — the vulnerability exists regardless of execution order. The code concatenates user input into SQL queries without parameterization. An attacker can always close the intended query and append commands because the data and commands share the same channel.
- **Impact:** Full database read access to customer data, including personal and financial information.
- **Recommendation:** Replace all string concatenation with parameterized queries. This is the smallest safe change that eliminates the vulnerability. If a full rewrite is not immediately feasible, apply parameterized queries to the affected queries first, then remove or isolate the legacy pages. → `principles/minimal-change.md`

**Finding 2 (Observed — H3): Excessive data exposure via permissive queries.** Structurally guaranteed — the queries access more data than needed. Even without injection, the page queries columns and tables that are not required for its function, creating unnecessary exposure.
- **Impact:** Any data access vulnerability on this page exposes more information than the page needs.
- **Recommendation:** Restrict queries to only the columns needed for the page's function. Apply the principle of least privilege at the query level, not just at the application level. → `knowledge/least-privilege.md`

**Finding 3 (Inferred — H4): Over-privileged database credentials.** Structurally guaranteed — if the database connection has broader permissions than the page requires, every vulnerability on that page has amplified impact.
- **Impact:** A vulnerability in one page grants access to data that the page should never be able to read.
- **Recommendation:** Scope database credentials to the minimum permissions needed for each page or service. Read-only access for pages that only display data. Column-level permissions where possible. → `knowledge/least-privilege.md`

**Finding 4 (Inferred — H2): Cross-Site Scripting risk.** Time-dependent — XSS would manifest under specific conditions (other users viewing a page that renders attacker-controlled input). The same trust boundary failure that enables SQL Injection also enables XSS.
- **Impact:** Session hijacking, credential theft, or further attacks against other users.
- **Recommendation:** Verify output escaping on legacy pages. Apply context-appropriate encoding (HTML entity encoding for HTML contexts, JavaScript encoding for script contexts). → `knowledge/input-validation.md`

## Step 6 — Construct Output

The reviewer structures the output in three tiers: → `specs/SPEC-005-agent-operational-behavior.md`

**Observations:**
- Legacy pages were connected to the production database and accessible from the public internet.
- Query construction used string concatenation with user input (confirmed by breach investigation).
- Database queries accessed columns and tables beyond what the pages needed to function.
- Database credentials had broader permissions than the pages required.

**Inferences:**
- SQL Injection was structurally guaranteed by the absence of parameterized queries. The vulnerability did not depend on timing or concurrent execution — it existed in every request.
- Excessive data exposure was structurally guaranteed by permissive queries. Even if the injection had been prevented, the page still queried more data than needed.
- Over-privileged database credentials amplified the impact of any data access vulnerability on the legacy pages.

**Risks:**
- Any user input reaching a database query without parameterization can extract, modify, or delete data.
- Legacy pages connected to production data create persistent, exploitable trust boundary failures.
- The combination of injection vulnerability, permissive queries, and over-privileged credentials created a single point of failure that compromised the entire customer database.

All findings are presented for human review before any action is taken. → `principles/human-in-the-loop.md`

---

## What This Example Demonstrates

- **The Trust Boundary skill would have detected this breach before it happened.** The same reasoning process that identifies a textbook authentication bypass also identifies a real-world SQL Injection — the root cause is the same: user input crosses a trust boundary without proper validation. → `knowledge/trust-boundaries.md`
- **Structurally guaranteed findings are deterministic.** SQL Injection via string concatenation does not depend on timing, concurrency, or execution order. It exists on every request. This distinguishes it from time-dependent findings like race conditions. → `specs/SPEC-005-agent-operational-behavior.md`
- **Confirmed by absence of control applies to real incidents.** The absence of parameterized queries is not an assumption — it is evidence. The skill treats missing defensive mechanisms as findings, not as gaps in knowledge. → `specs/SPEC-005-agent-operational-behavior.md`
- **Defense in Depth limits blast radius.** Even when one control fails (input validation), other controls (least privilege on database credentials, minimal query scope) contain the damage. The breach was amplified by the absence of these secondary controls. → `knowledge/defense-in-depth.md`
- **Legacy code is not an exception.** Pages connected to production data are security-relevant regardless of their age. The skill does not dismiss legacy components as noise — it identifies every point where untrusted input reaches trusted resources. → `knowledge/input-validation.md`
- **The smallest safe change addresses the root cause.** Parameterized queries eliminate the injection. Least-privilege database credentials contain the blast radius. Both changes are targeted and proportional — no rewrite required. → `principles/minimal-change.md`