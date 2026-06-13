# Security Engineer Skill

A portable AI Skill for Security Engineering, built entirely on structured documentation.

No code. No packages. No installation. Just knowledge, principles, and reasoning.

## What This Is

A documentation-first skill that teaches AI agents how to behave like responsible security engineers — thinking adversarially, defending methodically, and communicating clearly.

Designed to work across OpenCode, Claude Code, Cursor, Gemini CLI, and any agent that reads `AGENTS.md` or `SKILL.md`.

## What This Is Not

- Not a vulnerability scanner, linter, or detection tool.
- Not a Python package, library, or installable module.
- Not an MCP server or network service.
- Not a replacement for human judgment or professional review.

## How It Works

The skill activates when a security lens changes the analysis. It loads behavioral norms and a reasoning model, then selects relevant knowledge based on the task context.

See [`SKILL.md`](SKILL.md) for activation rules and context loading.

## Structure

```
specs/          # What the project is, how it is structured, how it behaves
principles/     # How the skill thinks — core reasoning principles
knowledge/      # What the skill knows — security engineering concepts
checklists/     # What the skill checks — actionable verification lists
examples/       # How the skill applies — worked walkthroughs
references/     # Where the skill points — external sources and further reading
```

See [`AGENTS.md`](AGENTS.md) for contribution guidelines and design principles.

## Specs

| Spec | Purpose |
|------|---------|
| [SPEC-001](specs/SPEC-001-project-vision.md) | Project vision and scope |
| [SPEC-002](specs/SPEC-002-architecture.md) | Repository structure and directory responsibilities |
| [SPEC-003](specs/SPEC-003-skill-behavior.md) | Behavioral principles and communication norms |
| [SPEC-004](specs/SPEC-004-content-organization.md) | How content should be written and organized |
| [SPEC-005](specs/SPEC-005-agent-operational-behavior.md) | How the agent processes, reasons, and decides |

## Principles

| Principle | Core Idea |
|-----------|-----------|
| [Evidence over Assumptions](principles/evidence-over-assumptions.md) | Base findings on observable evidence. Say so when evidence is unavailable. |
| [Education over Fear](principles/education-over-fear.md) | Teach and explain. Do not overwhelm with warnings. |
| [Human in the Loop](principles/human-in-the-loop.md) | Inform and support decisions. Never replace human judgment. |
| [Minimal Change](principles/minimal-change.md) | Prefer the smallest safe change that solves the problem. |
| [Context Matters](principles/context-matters.md) | Interpret every finding within its environment and intended use. |

## Knowledge

| Concept | Key Insight |
|---------|------------|
| [Secure by Default](knowledge/secure-by-default.md) | The path of least resistance should be the safe path. |
| [Least Privilege](knowledge/least-privilege.md) | Give only the permissions needed — nothing more. |
| [Defense in Depth](knowledge/defense-in-depth.md) | No single control should be the only one standing. |
| [Input Validation](knowledge/input-validation.md) | Define what is acceptable. Reject everything else. |
| [Secrets Management](knowledge/secrets-management.md) | A leaked secret is an identity compromise, not a data breach. |
| [Secure Serialization](knowledge/secure-serialization.md) | Reconstruction is an active operation, not a passive one. |
| [Trust Boundaries](knowledge/trust-boundaries.md) | Every point where data changes trust level is a security boundary. |
| [Supply Chain Security](knowledge/supply-chain-security.md) | Every dependency is a trust decision — most are made implicitly. |
| [Threat Modeling](knowledge/threat-modeling.md) | Identify risks before they become incidents, not after. |
| [Error Handling and Secure Failure](knowledge/error-handling-secure-failure.md) | When something goes wrong, the system should become more restrictive, not less. |

## Checklists

| Checklist | Purpose |
|-----------|---------|
| [Security Code Review](checklists/security-code-review.md) | Review code changes for security concerns |
| [Secrets Management Review](checklists/secrets-management-review.md) | Review how a system handles credentials and secrets |
| [Threat Modeling Review](checklists/threat-modeling-review.md) | Conduct a structured threat model |

## Examples

| Example | Demonstrates |
|---------|-------------|
| [Authentication Bypass Review](examples/authentication-bypass-review.md) | Full reasoning loop: perception, context, hypothesis, evidence, output |

## License

See [LICENSE](LICENSE).