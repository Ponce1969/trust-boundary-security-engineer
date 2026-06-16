# Security Code Review

Ordered checklist for reviewing code changes for security concerns.

## Context

Identify the review context before starting:

- What the change does and why it was made.
- What data flows are affected.
- What trust boundaries the change crosses.
- What permissions or access the change requires.

## 1. Input and Data Flow

- [ ] Identify every point where external data enters the system. → `knowledge/input-validation.md`
- [ ] Verify that input is validated at trust boundaries, not only at the UI layer. → `knowledge/trust-boundaries.md`
- [ ] Confirm that input validation defines acceptable input (whitelist), not blocked input (blacklist).
- [ ] Check that serialized data from external sources is treated as untrusted. → `knowledge/secure-serialization.md`
- [ ] Verify that data crossing boundaries between services is re-validated, not assumed safe because it is internal.

## 2. Authentication and Authorization

- [ ] Identify every authentication check in the change. → `knowledge/least-privilege.md`
- [ ] Verify that authorization checks enforce least privilege — access is limited to what is needed, not what is convenient.
- [ ] Confirm that failed authentication and authorization default to denying access, not granting it. → `knowledge/error-handling-secure-failure.md`
- [ ] Check that authorization is enforced at every relevant trust boundary, not only at the entry point.

## 3. Secrets and Credentials

- [ ] Scan for hardcoded credentials, API keys, tokens, or connection strings. → `knowledge/secrets-management.md`
- [ ] Verify that secrets are not embedded in logs, error messages, or configuration dumps.
- [ ] Confirm that secrets are isolated by environment — development, staging, and production credentials do not overlap.
- [ ] Check that secret rotation is possible without system redesign.

## 4. Error Handling and Failure Behavior

- [ ] Trace every error path in the change. → `knowledge/error-handling-secure-failure.md`
- [ ] Verify that error responses reveal no internal state, stack traces, or sensitive data.
- [ ] Confirm that authentication and authorization failures deny access, not allow it.
- [ ] Check that empty catch blocks or silent error swallowing are not used as error handling.
- [ ] Verify that the system becomes more restrictive on failure, not less.

## 5. Dependencies and Integration

- [ ] Identify every external dependency added or modified by the change. → `knowledge/supply-chain-security.md`
- [ ] Verify that dependencies are from trusted, maintained sources.
- [ ] Check that integration points treat data from other services as untrusted until validated. → `knowledge/trust-boundaries.md`
- [ ] Confirm that serialization formats used for external communication are data-only, not object graphs that reconstruct behavior. → `knowledge/secure-serialization.md`

## 6. Configuration and Defaults

- [ ] Verify that default configurations are secure by default. → `knowledge/secure-by-default.md`
- [ ] Check that security-relevant settings are opt-in for insecure behavior, not opt-in for secure behavior.
- [ ] Confirm that the change does not introduce unnecessary privileges or permissions. → `knowledge/least-privilege.md`

## 7. Control Sufficiency

- [ ] Verify that each security control is sufficient for the specific threat model, not merely present. A control that exists but is inadequate for the threat it faces is a different finding than a control that is absent. → `specs/SPEC-005-agent-operational-behavior.md`
- [ ] Check whether rate limiting, throttling, or retry mechanisms can be circumvented under concurrent load or retry conditions. A rate limiter with narrow jitter may be insufficient under thundering herd conditions.
- [ ] Verify that authentication controls (account lockout, rate limiting) cannot be weaponized to deny service to legitimate users. A lockout counter that allows unlimited attempts from different IPs is insufficient against distributed attacks.
- [ ] Confirm that authorization checks cover the specific resource being accessed, not just whether the user is authenticated. An authorization check that verifies identity without verifying ownership is insufficient for preventing access to other users' resources.
- [ ] Assess whether error handling reveals enough information to aid debugging but not enough to aid attackers. A generic error message that discloses the database engine version is insufficient for security.

## 8. Reporting

- [ ] Document findings with clear separation of observations (verified), inferences (supported but uncertain), and risks (conditional).
- [ ] Prioritize findings by impact and context, not by category. → `principles/context-matters.md`
- [ ] Recommend the smallest safe change for each finding. → `principles/minimal-change.md`
- [ ] Present findings for human review before any action is taken. → `principles/human-in-the-loop.md`