<div align="center">

# Trust Boundary

**Security Engineering Skill for AI Agents**

*A portable, documentation-first reasoning framework that teaches agents to think like responsible security engineers — adversarially, methodically, and with evidence over assumptions.*

**No code · No packages · No installation · Just knowledge, principles, and structured reasoning**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Docs: Markdown](https://img.shields.io/badge/Format-Markdown-lightgrey.svg)]()
[![Skill: v1.0](https://img.shields.io/badge/Skill-v1.0-green.svg)]()
[![Scope: Security](https://img.shields.io/badge/Scope-Security_Engineering-critical.svg)]()
[![Content: 5 specs · 10 knowledge · 5 principles · 3 checklists · 2 examples](https://img.shields.io/badge/Content-5_specs_·_10_knowledge_·_5_principles-blueviolet.svg)]()

</div>

---

## Why This Exists

Most security tools **find problems**. This skill teaches **how to think** about them.

It gives AI agents a structured reasoning framework — perceive input, construct context, generate hypotheses, evaluate evidence, synthesize decisions — instead of a list of rules to check off.

The result: findings that are **proportional, evidence-backed, and teach** the reader why something matters, not just that it does.

## Who This Is For

| Who | Why |
|-----|-----|
| **AI agent developers** | Want agents that reason about security instead of pattern-matching vulnerability names |
| **Security engineers** | Review code with AI assistance and want structured, proportional findings |
| **Platform teams** | Build concurrent or distributed systems and need temporal reasoning, not just spatial analysis |

## What Makes This Different

| Typical Security Tool | Trust Boundary Skill |
|:---------------------|:---------------------|
| Pattern-matches vulnerability names | Reasons from evidence through structured hypotheses |
| Treats all findings equally | Qualifies findings as **structurally guaranteed** or **time-dependent** |
| Reports controls as binary present/absent | Evaluates control **sufficiency** — present but inadequate is a different finding |
| Analyzes code at a single point in time | Models **state evolution** across execution time |
| Ignores concurrency or treats it as a special case | Constructs **temporal trust boundaries** alongside spatial ones |
| Outputs flat vulnerability lists | Structures findings as **Observations · Inferences · Risks** with explicit confidence levels |
| Stops at "control missing" | Asks "is the control sufficient for this threat model?" |

## How It Works

The skill applies a **six-step reasoning model** when a security lens changes the analysis:

<table>
<tr><td width="60"><strong>1</strong></td><td><strong>Perceive Input</strong> — Distinguish signal from noise. Flag ambiguity.</td></tr>
<tr><td><strong>2</strong></td><td><strong>Construct Context</strong> — Identify trust boundaries (spatial + temporal), classify the execution model, model state evolution.</td></tr>
<tr><td><strong>3</strong></td><td><strong>Generate Hypotheses</strong> — Propose explanations grounded in evidence. Generate across mandated categories: concurrency, credential lifecycle, idempotency, security controls undermined by retries, distributed inconsistency resolution.</td></tr>
<tr><td><strong>4</strong></td><td><strong>Evaluate Evidence</strong> — Separate fact from inference. Classify as <em>confirmed</em>, <em>confirmed by absence of control</em>, <em>confirmed by insufficient control</em>, or <em>requiring external evidence</em>.</td></tr>
<tr><td><strong>5</strong></td><td><strong>Synthesize Decisions</strong> — Qualify conclusions as structurally guaranteed or time-dependent. Communicate confidence levels explicitly.</td></tr>
<tr><td><strong>6</strong></td><td><strong>Construct Output</strong> — Structure findings as Observations, Inferences, and Risks with proportional recommendations.</td></tr>
</table>

## Key Reasoning Capabilities

> **Execution Model Classification**
> Before reasoning, the agent classifies the system's execution model (sequential, concurrent, distributed, or mixed). This determines which hypothesis categories are mandatory rather than optional.

> **State Evolution Modeling**
> For concurrent and distributed systems, shared mutable state is traced from creation through mutation to disposal. Windows where state is inconsistent, partially committed, or visible to unauthorized actors are identified across execution time — not just at a single snapshot.

> **Temporal Trust Boundaries**
> Traditional analysis identifies spatial boundaries (service-to-service, layer-to-layer). This skill also identifies **temporal boundaries** — where concurrent execution allows state from one context to leak into another without explicit synchronization.

> **Defensive Control Sufficiency**
> A control that exists but is inadequate for the threat model is a different finding than a control that is absent. A rate limiter with ±10% jitter under 100 concurrent requests is present but insufficient. The skill evaluates **sufficiency**, not just presence.

> **Temporal Qualification of Conclusions**
> Findings are classified as **structurally guaranteed** (hold regardless of execution order) or **time-dependent** (manifest under specific interleavings). This prevents treating race conditions as deterministic or dismissing them as unlikely.

> **Security Controls Undermined by Retries**
> A security mechanism that works in normal conditions may fail under retry or concurrent execution. Rate limits consumed by retry storms, account lockout counters bypassed by parallel attempts — these are hypotheses where the control exists but is undermined by the execution model.

> **Distributed Inconsistency Resolution**
> When a system can reach inconsistent state, the resolution mechanism determines the security impact. Last-writer-wins silently discards authorized changes. Manual reconciliation leaves inconsistency indefinitely. Automatic merge may produce unintended state. The skill evaluates the resolution, not just the inconsistency.

## Repository Structure

```
security-engineer-skill/
├── README.md              ← You are here
├── LICENSE                ← MIT
├── AGENTS.md              ← Agent behavior rules and contribution guidelines
├── SKILL.md               ← Skill definition, activation, and context loading
│
├── specs/                 ← What the project is and how it behaves
│   ├── SPEC-001           ← Project vision and scope
│   ├── SPEC-002           ← Architecture and directory responsibilities
│   ├── SPEC-003           ← Behavioral principles and communication norms
│   ├── SPEC-004           ← Content organization rules
│   └── SPEC-005           ← Agent reasoning model (the core)
│
├── principles/            ← How the skill thinks
├── knowledge/             ← What the skill knows
├── checklists/            ← What the skill checks
├── examples/              ← How the skill applies
└── references/            ← Where the skill points
```

## What's Inside

<details>
<summary><strong>📄 Specs — Project definitions</strong></summary>

| Spec | Purpose |
|------|---------|
| [SPEC-001](specs/SPEC-001-project-vision.md) | Project vision and scope |
| [SPEC-002](specs/SPEC-002-architecture.md) | Repository structure and directory responsibilities |
| [SPEC-003](specs/SPEC-003-skill-behavior.md) | Behavioral principles and communication norms |
| [SPEC-004](specs/SPEC-004-content-organization.md) | How content should be written and organized |
| [SPEC-005](specs/SPEC-005-agent-operational-behavior.md) | How the agent processes, reasons, and decides |

</details>

<details>
<summary><strong>🧭 Principles — Core reasoning rules</strong></summary>

| Principle | Core Idea |
|-----------|-----------|
| [Evidence over Assumptions](principles/evidence-over-assumptions.md) | Base findings on observable evidence. Say so when evidence is unavailable. |
| [Education over Fear](principles/education-over-fear.md) | Teach and explain. Do not overwhelm with warnings. |
| [Human in the Loop](principles/human-in-the-loop.md) | Inform and support decisions. Never replace human judgment. |
| [Minimal Change](principles/minimal-change.md) | Prefer the smallest safe change that solves the problem. |
| [Context Matters](principles/context-matters.md) | Interpret every finding within its environment and intended use. |

</details>

<details>
<summary><strong>📚 Knowledge — Security engineering concepts</strong></summary>

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

</details>

<details>
<summary><strong>✅ Checklists — Actionable verification</strong></summary>

| Checklist | Purpose |
|-----------|---------|
| [Security Code Review](checklists/security-code-review.md) | Review code changes for security concerns |
| [Secrets Management Review](checklists/secrets-management-review.md) | Review how a system handles credentials and secrets |
| [Threat Modeling Review](checklists/threat-modeling-review.md) | Conduct a structured threat model |

</details>

<details>
<summary><strong>🔍 Examples — Worked walkthroughs</strong></summary>

| Example | Demonstrates |
|---------|-------------|
| [Authentication Bypass Review](examples/authentication-bypass-review.md) | Full reasoning loop: perception, context, hypothesis, evidence, output |
| [Concurrent Payment Review](examples/concurrent-payment-review.md) | Temporal trust boundaries, state evolution modeling, execution model classification, defensive control sufficiency, temporal qualification, `[REQUIRES_EXTERNAL_EVIDENCE]` |
| [TalkTalk Breach Case Study](examples/talktalk-case-study.md) | Real-world incident analysis: structurally guaranteed findings, confirmed by absence of control, defense in depth, legacy system risks |

</details>

## Platform Compatibility

This skill works with any AI agent that reads `AGENTS.md` or `SKILL.md`:

| Platform | Setup |
|----------|-------|
| **OpenCode** | Place `SKILL.md` in `~/.config/opencode/skills/security-engineer/` |
| **Claude Code** | Reference `AGENTS.md` in the project root |
| **Cursor** | Include `AGENTS.md` in workspace settings |
| **Gemini CLI** | Reference `AGENTS.md` as context |
| **Any agent** | Point to `SKILL.md` as the activation contract |

## Quick Start

1. **Read** [`SKILL.md`](SKILL.md) — the activation contract and context loading rules
2. **Start** with [`SPEC-005`](specs/SPEC-005-agent-operational-behavior.md) — the core reasoning model
3. **Try** the [`Concurrent Payment Review`](examples/concurrent-payment-review.md) — see the skill in action
4. **Activate** by placing `SKILL.md` in your agent's skills directory

## License

[MIT](LICENSE) — Use it, modify it, share it.