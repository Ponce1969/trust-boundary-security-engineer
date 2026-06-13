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

### Non-Activation

Do not activate when:

- The conversation does not involve systems or code
- The content is purely conceptual with no security implications
- The task is documentation, writing, or general explanation with no technical risk
- There is no interaction with data, systems, or executable logic

## Context Loading

On activation, load:

- `specs/SPEC-003-skill-behavior.md` — behavioral norms
- `specs/SPEC-005-agent-operational-behavior.md` — reasoning model
- Relevant `principles/` and `knowledge/` entries based on the task context

## Efficiency Rule

Do not load this skill by default for all programming tasks. Activate only when it adds value, changes the interpretation of the system, or introduces a relevant risk perspective.