# Repository Entry Scan

Entry protocol for reviewing an unfamiliar repository. Apply this before any other checklist. Its purpose is to orient the analysis — classify the project, surface immediate signals, and select the correct checklist for deeper review.

This checklist does not find vulnerabilities. It determines where to look for them.

---

## Step 1 — Classify the Project

Read these files first, in this order:

- `README.md` or `docs/` — stated purpose, architecture, deployment model
- `package.json`, `requirements.txt`, `Cargo.toml`, `go.mod`, or equivalent — language, runtime, dependencies
- `Dockerfile`, `docker-compose.yml`, `kubernetes/`, `terraform/`, `*.tf` — infrastructure and deployment
- `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile` — CI/CD pipeline

Determine:

- [ ] What does this project do? (web app, API, CLI tool, library, agent, infrastructure)
- [ ] What is the execution model? (sequential, concurrent, distributed, agentic)
- [ ] What does it connect to? (databases, external APIs, cloud services, other agents)
- [ ] Who are the users? (public internet, authenticated users, internal services, other agents)

This classification drives everything that follows. A concurrent API connecting to a payment gateway requires different analysis than a CLI tool that reads local files.

---

## Step 2 — Immediate Signal Scan

Before reading any code, scan for high-severity signals that require urgent attention regardless of project type.

**Secrets and credentials — scan these locations first:**

- [ ] Any `.env`, `.env.*`, `config/`, `settings/` files committed to the repository
- [ ] CI/CD pipeline files for hardcoded `export KEY=`, `password=`, or `token=` patterns
- [ ] `README.md` or documentation files with example credentials that may be real
- [ ] Any file matching `*secret*`, `*credential*`, `*password*`, `*key*` in the filename

If hardcoded credentials are found: this is the highest priority finding. Stop and report before continuing. → `knowledge/secrets-management.md`

**Configuration signals — scan second:**

- [ ] IAM policies or permission files with `"*"` wildcards on both Action and Resource
- [ ] S3, storage, or database configurations with public access enabled
- [ ] Network configurations with `0.0.0.0/0` ingress rules without justification
- [ ] Authentication disabled or bypassed in any configuration file

**Dependency signals — scan third:**

- [ ] Dependency files for packages with known vulnerabilities (look for very old pinned versions)
- [ ] Direct inclusion of packages with unusual names (possible typosquatting)
- [ ] Unpinned dependency versions (`*`, `latest`, `>=`) in production dependency files

→ `knowledge/supply-chain-security.md`

---

## Step 3 — Identify Trust Boundaries

Based on Step 1 classification, map where data crosses trust levels:

- [ ] Where does external input enter the system? (HTTP endpoints, file uploads, CLI arguments, environment variables, messages from other agents)
- [ ] Where does the application cross into persistent storage? (databases, file system, caches)
- [ ] Where does the application call external services? (APIs, payment gateways, email, cloud services)
- [ ] If the project is an agent: what tools does it have, and what can each tool do irreversibly?

Draw a simple list: `[external input] → [app boundary] → [storage/external services]`

This list becomes the hypothesis surface for the deeper review. Every item on it is a location where a finding is possible. → `knowledge/trust-boundaries.md`

---

## Step 4 — Select the Checklist

Based on Steps 1-3, select the appropriate checklist for the deeper review:

| If the project has... | Use this checklist |
|---|---|
| Code changes to review (functions, classes, endpoints) | `checklists/security-code-review.md` |
| Credentials, tokens, API keys, secrets handling | `checklists/secrets-management-review.md` |
| Architecture with multiple services or complex data flows | `checklists/threat-modeling-review.md` |
| An AI agent with tools, memory, or multi-agent communication | `checklists/agentic-system-review.md` |

More than one checklist may apply. Start with the one that matches the highest-severity signals found in Step 2.

---

## Step 5 — Apply SPEC-005 Reasoning

With the checklist selected and the trust boundary map from Step 3, apply the six-step reasoning model: → `specs/SPEC-005-agent-operational-behavior.md`

1. **Perceive** — What are the security-relevant signals in this project?
2. **Construct context** — What are the trust boundaries? What is the execution model?
3. **Generate hypotheses** — Given the project type and boundaries, what could go wrong?
4. **Evaluate evidence** — What is directly observable? What requires external evidence?
5. **Synthesize** — Which findings are structurally guaranteed? Which are time-dependent?
6. **Output** — Observations, Inferences, Risks — proportional to what was found.

---

## What to Do With Findings

- Report findings in order of blast radius — the maximum harm if exploited, not the category name.
- Distinguish what is confirmed from what is inferred. → `principles/evidence-over-assumptions.md`
- Recommend the smallest safe change for each finding. → `principles/minimal-change.md`
- Present findings for human review before any action is taken. → `principles/human-in-the-loop.md`

A repository scan with no findings is a valid result. Report what was checked, not just what was found.
