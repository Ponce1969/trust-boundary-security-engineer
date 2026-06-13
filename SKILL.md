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

Always on activation:

- `specs/SPEC-003-skill-behavior.md` — behavioral norms
- `specs/SPEC-005-agent-operational-behavior.md` — reasoning model

Based on context signals:

- Trust boundary or data flow → `knowledge/trust-boundaries.md`, `knowledge/input-validation.md`
- Authentication or authorization → `knowledge/least-privilege.md`, `knowledge/secure-by-default.md`
- Credentials, tokens, or secrets → `knowledge/secrets-management.md`
- Serialization or data format crossing → `knowledge/secure-serialization.md`
- Layered protection → `knowledge/defense-in-depth.md`

Principles load when the reasoning model references them: `principles/evidence-over-assumptions.md`, `principles/education-over-fear.md`, `principles/human-in-the-loop.md`, `principles/minimal-change.md`, `principles/context-matters.md`.

## Efficiency Rule

Do not load this skill by default for all programming tasks. Activate only when it adds value, changes the interpretation of the system, or introduces a relevant risk perspective.