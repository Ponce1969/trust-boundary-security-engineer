# Security Engineering Skill

## Activation

This skill activates when the security lens changes the decision. If the analysis would be the same without it, do not activate.

### Automatic Activation

Activate when the input context indicates a security-relevant task:

- Code review with potential security impact
- Authentication or authorization analysis
- Handling credentials, tokens, or secrets
- Evaluating data flows between systems
- Configuring systems with security implications
- Architecture analysis with risk implications
- Debugging where a vulnerability is possible

Activation is based on intent and context, not on explicit keywords.

### Explicit Activation

Activate when the user or system directly requests:

- security review
- vulnerability analysis
- threat modeling
- code audit
- secure design review
- any explicit mention of "security engineering skill"

### Ambiguous Cases

When activation is uncertain, default to activation only if data crosses a trust boundary or external input is involved. Otherwise, do not activate.

### Non-Activation

Do not activate when:

- The conversation does not involve systems or code
- The content is purely conceptual with no security implications
- The task is documentation, writing, or general explanation with no technical risk
- There is no interaction with data, systems, or executable logic

## Context Loading

### Analysis Depth

Before loading context, classify the review depth required:

**Lightweight review** — a focused change with limited blast radius (a single function, a configuration value, a dependency update). Load only the directly relevant knowledge file and checklist. Skip examples unless the pattern closely matches.

**Deep review** — a system with multiple trust boundaries, concurrent execution, distributed coordination, or high-stakes data (credentials, payments, PII, infrastructure). Load the full context chain: knowledge, checklist, and the most relevant example. Apply the full six-step reasoning model from SPEC-005.

When uncertain, default to deep review if external input is involved or trust boundaries are crossed.

### Always on Activation

- `specs/SPEC-003-skill-behavior.md` — behavioral norms
- `specs/SPEC-005-agent-operational-behavior.md` — reasoning model

### Based on Context Signals

- Trust boundary or data flow → `knowledge/trust-boundaries.md`, `knowledge/input-validation.md`
- Authentication or authorization → `knowledge/least-privilege.md`, `knowledge/secure-by-default.md`
- Credentials, tokens, or secrets → `knowledge/secrets-management.md`
- Serialization or data format crossing → `knowledge/secure-serialization.md`
- Layered protection or blast radius analysis → `knowledge/defense-in-depth.md`
- Dependencies, CI/CD, or supply chain → `knowledge/supply-chain-security.md`
- Threat modeling or risk assessment → `knowledge/threat-modeling.md`
- Error handling or failure behavior → `knowledge/error-handling-secure-failure.md`
- Cloud, IAC, or infrastructure configuration → `knowledge/least-privilege.md`, `knowledge/secrets-management.md`, `knowledge/supply-chain-security.md`
- Evaluating whether a control is adequate (not just present) → `principles/control-sufficiency.md`
- Agentic system, LLM application, or multi-agent architecture → `knowledge/prompt-injection.md`, `knowledge/agent-trust-model.md`

### Checklists Load Based on Task Type

- Code review or code change analysis → `checklists/security-code-review.md`
- Secrets, credentials, or tokens analysis → `checklists/secrets-management-review.md`
- Threat modeling or architecture review → `checklists/threat-modeling-review.md`
- Agentic system or LLM application review → `checklists/agentic-system-review.md`

### Examples Load for Reference When the Reasoning Model is Applied to a Similar Scenario

- Authentication, input validation, or SQL Injection → `examples/authentication-bypass-review.md`
- Concurrent execution, shared state, or distributed systems → `examples/concurrent-payment-review.md`
- Real-world breach analysis or documented incident → `examples/talktalk-case-study.md`
- Cloud configuration, IAC, or CI/CD pipeline security → `examples/iac-cloud-misconfiguration-review.md`
- Agentic system, prompt injection, or multi-agent trust → `examples/prompt-injection-review.md`

### Principles Load When the Reasoning Model References Them

`principles/evidence-over-assumptions.md`, `principles/education-over-fear.md`, `principles/human-in-the-loop.md`, `principles/minimal-change.md`, `principles/context-matters.md`, `principles/control-sufficiency.md`

### References Load for External Validation and Classification

- Classifying a finding by vulnerability type → `references/owasp-and-vulnerability-standards.md`
- Threat modeling methodology selection → `references/threat-modeling-frameworks.md`
- Cloud, IAC, or container security standards → `references/cloud-and-infrastructure-security.md`
- LLM application or agentic system risk classification → `references/owasp-llm-top10.md`

## Efficiency Rule

Do not load this skill by default for all programming tasks. Activate only when it adds value, changes the interpretation of the system, or introduces a relevant risk perspective.