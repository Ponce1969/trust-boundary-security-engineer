# Secrets Management Review

Ordered checklist for reviewing how a system handles credentials, tokens, and secrets.

## Context

Establish the secrets landscape before reviewing:

- What secrets the system uses (API keys, tokens, certificates, database credentials).
- Where secrets are stored and how they are accessed.
- What trust boundaries secrets cross during their lifecycle.
- Who and what has access to each secret.

## 1. Secret Storage

- [ ] List every secret the system requires and where each is stored. → `knowledge/secrets-management.md`
- [ ] Verify that no secrets are hardcoded in source code, configuration files committed to version control, or embedded in build artifacts.
- [ ] Confirm that secrets are stored in dedicated secret management systems, not in environment variables that are logged, shared, or committed.
- [ ] Check that `.env` files, local configuration, and development overrides are excluded from version control.

## 2. Secret Access and Distribution

- [ ] Identify every service, process, and role that accesses each secret. → `knowledge/least-privilege.md`
- [ ] Verify that each secret is scoped to the narrowest set of permissions required by its consumer.
- [ ] Confirm that secrets are not shared across unrelated services or environments.
- [ ] Check that secret access is logged and auditable.

## 3. Secret Lifecycle

- [ ] Verify that secrets can be rotated without system redesign or downtime.
- [ ] Confirm that secret rotation is performed proactively, not only after incidents.
- [ ] Check that temporary credentials, tokens, and access keys have expiration and are revoked when no longer needed. → `knowledge/least-privilege.md`
- [ ] Identify any secrets that have no rotation mechanism and flag them as unmanaged risk.

## 4. Secret Exposure Surfaces

- [ ] Trace every path where a secret appears — logs, error messages, API responses, debug output. → `knowledge/error-handling-secure-failure.md`
- [ ] Verify that error handlers do not leak secrets in error messages or stack traces.
- [ ] Confirm that secrets are stripped from logs at every trust boundary. → `knowledge/trust-boundaries.md`
- [ ] Check that secrets are not transmitted over insecure channels or stored in plaintext in transit.

## 5. Trust Boundaries and Secrets

- [ ] Identify every trust boundary that secrets cross — between services, environments, and deployment stages. → `knowledge/trust-boundaries.md`
- [ ] Verify that secrets are protected at each boundary crossing, not only at the point of origin.
- [ ] Confirm that internal services do not automatically trust secrets passed from other services without verification.
- [ ] Check that secrets used in CI/CD pipelines are isolated from the pipeline's broader environment. → `knowledge/supply-chain-security.md`

## 6. Reporting

- [ ] Document each finding as an observation (verified fact) or inference (supported but uncertain). → `principles/evidence-over-assumptions.md`
- [ ] Classify findings by real impact in the specific context, not generic severity. → `principles/context-matters.md`
- [ ] Recommend the smallest safe change for each finding. → `principles/minimal-change.md`
- [ ] Present findings for human review before any remediation action. → `principles/human-in-the-loop.md`