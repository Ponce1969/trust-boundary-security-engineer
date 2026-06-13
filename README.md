# Trust Boundary — Security Engineer Skill

A portable, documentation-first AI skill that teaches agents to reason like responsible security engineers — adversarially, methodically, and with evidence over assumptions.

**No code. No packages. No installation.** Just knowledge, principles, and a structured reasoning model.

---

## Why This Exists

Most security tools find problems. This skill teaches **how to think** about security problems. It gives AI agents a reasoning framework — perceive input, construct context, generate hypotheses, evaluate evidence, synthesize decisions — instead of a list of rules to check off.

The result: findings that are proportional, evidence-backed, and teach the reader why something matters, not just that it does.

## Who This Is For

- **AI agent developers** who want their agents to reason about security instead of pattern-matching vulnerability names
- **Security engineers** who review code with AI assistance and want structured, proportional findings
- **Teams** building concurrent or distributed systems who need temporal reasoning, not just spatial analysis

## What Makes This Different

| Typical Security Tool | Trust Boundary Skill |
|----------------------|---------------------|
| Pattern-matches vulnerability names | Reasons from evidence through structured hypotheses |
| Treats all findings equally | Qualifies findings as structurally guaranteed or time-dependent |
| Reports missing controls as binary (present/absent) | Evaluates control **sufficiency** — present but inadequate is a different finding |
| Analyzes code at a single point in time | Models **state evolution** across execution time |
| Ignores concurrency or treats it as a special case | Constructs **temporal trust boundaries** alongside spatial ones |
| Outputs flat vulnerability lists | Structures findings as Observations, Inferences, and Risks with explicit confidence levels |

## How It Works

The skill loads when a security lens changes the analysis. It applies a six-step reasoning model:

1. **Perceive Input** — Distinguish signal from noise. Flag ambiguity.
2. **Construct Context** — Identify trust boundaries (spatial + temporal), classify the execution model, model state evolution.
3. **Generate Hypotheses** — Propose explanations grounded in evidence. Generate across mandated categories: concurrency, credential lifecycle, idempotency, security controls undermined by retries, distributed inconsistency resolution.
4. **Evaluate Evidence** — Separate fact from inference. Classify as confirmed, confirmed by absence of control, confirmed by insufficient control, or requiring external evidence.
5. **Synthesize Decisions** — Qualify conclusions as structurally guaranteed or time-dependent. Communicate confidence levels explicitly.
6. **Construct Output** — Structure findings as Observations, Inferences, and Risks with proportional recommendations.

## Key Reasoning Capabilities

### Execution Model Classification
Before reasoning, the agent classifies the system's execution model (sequential, concurrent, distributed, or mixed). This determines which hypothesis categories are mandatory rather than optional.

### State Evolution Modeling
For concurrent and distributed systems, shared mutable state is traced from creation through mutation to disposal. Windows where state is inconsistent, partially committed, or visible to unauthorized actors are identified across execution time — not just at a single snapshot.

### Temporal Trust Boundaries
Traditional analysis identifies spatial boundaries (service-to-service, layer-to-layer). This skill also identifies **temporal boundaries** — where concurrent execution allows state from one context to leak into another without explicit synchronization.

### Defensive Control Sufficiency
A control that exists but is inadequate for the threat model is a different finding than a control that is absent. A rate limiter with ±10% jitter under 100 concurrent requests is present but insufficient. The skill evaluates sufficiency, not just presence.

### Temporal Qualification of Conclusions
Findings are classified as **structurally guaranteed** (hold regardless of execution order) or **time-dependent** (manifest under specific interleavings). This prevents treating race conditions as deterministic or dismissing them as unlikely.

## Repository Structure

```
security-engineer-skill/
├── README.md          # This file — project overview and usage guide
├── LICENSE            # MIT License
├── AGENTS.md         # Agent behavior rules and contribution guidelines
├── SKILL.md          # Skill definition, activation rules, and context loading
├── specs/             # Specification documents
│   ├── SPEC-001       # Project vision and scope
│   ├── SPEC-002       # Architecture and directory responsibilities
│   ├── SPEC-003       # Behavioral principles and communication norms
│   ├── SPEC-004       # Content organization rules
│   └── SPEC-005       # Agent reasoning model (the core)
├── principles/        # Core reasoning principles
├── knowledge/         # Domain knowledge entries
├── checklists/        # Actionable verification checklists
├── examples/          # Worked walkthroughs
└── references/        # External references and further reading
```

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
| [Concurrent Payment Review](examples/concurrent-payment-review.md) | Temporal trust boundaries, state evolution modeling, execution model classification, defensive control sufficiency, temporal qualification, `[REQUIRES_EXTERNAL_EVIDENCE]` |

## Platform Compatibility

This skill works with any AI agent that reads `AGENTS.md` or `SKILL.md`:

- **OpenCode** — Place `SKILL.md` in `~/.config/opencode/skills/security-engineer/`
- **Claude Code** — Reference `AGENTS.md` in the project root
- **Cursor** — Include `AGENTS.md` in workspace settings
- **Gemini CLI** — Reference `AGENTS.md` as context

## License

[MIT](LICENSE)