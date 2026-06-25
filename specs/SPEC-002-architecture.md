# Purpose

A repository's structure is its contract. When directories have clear responsibilities, contributors know where to put content and readers know where to find it. This document defines that contract for security-engineer-skill.

# Design Principles

- **Single responsibility.** Each directory owns one concern.
- **Simplicity over complexity.** Flat structure where possible. No nested hierarchies without reason.
- **Documentation-first.** Everything is Markdown. No code, no build artifacts.
- **Platform independence.** No tool-specific formats or conventions.
- **Prefer references over duplication.** Link to content rather than restate it.

# Repository Structure

```
security-engineer-skill/
├── README.md
├── LICENSE
├── AGENTS.md
├── SKILL.md
├── specs/
│   ├── SPEC-001-project-vision.md
│   ├── SPEC-002-architecture.md
│   ├── SPEC-003-skill-behavior.md
│   ├── SPEC-004-content-organization.md
│   └── SPEC-005-agent-operational-behavior.md
├── principles/
│   ├── context-matters.md
│   ├── control-sufficiency.md
│   ├── education-over-fear.md
│   ├── evidence-over-assumptions.md
│   ├── human-in-the-loop.md
│   └── minimal-change.md
├── knowledge/
│   ├── agent-trust-model.md
│   ├── defense-in-depth.md
│   ├── error-handling-secure-failure.md
│   ├── input-validation.md
│   ├── least-privilege.md
│   ├── prompt-injection.md
│   ├── secrets-management.md
│   ├── secure-by-default.md
│   ├── secure-serialization.md
│   ├── supply-chain-security.md
│   ├── threat-modeling.md
│   └── trust-boundaries.md
├── checklists/
│   ├── agentic-system-review.md
│   ├── repository-entry-scan.md
│   ├── secrets-management-review.md
│   ├── security-code-review.md
│   └── threat-modeling-review.md
├── examples/
│   ├── authentication-bypass-review.md
│   ├── concurrent-payment-review.md
│   ├── iac-cloud-misconfiguration-review.md
│   ├── prompt-injection-review.md
│   └── talktalk-case-study.md
└── references/
    ├── cloud-and-infrastructure-security.md
    ├── owasp-and-vulnerability-standards.md
    ├── owasp-llm-top10.md
    └── threat-modeling-frameworks.md
```

# Directory Responsibilities

Each directory has a single verb that defines its role in the skill. When an agent is unsure where content belongs, the verb answers the question.

| Directory | Verb | Role |
|-----------|------|------|
| `specs/` | **governs** | Defines what the project is and how it is structured |
| `principles/` | **decides** | Guides reasoning when evidence is ambiguous |
| `knowledge/` | **explains** | Teaches what a concept is and why it matters |
| `checklists/` | **verifies** | Confirms that specific controls or patterns are present |
| `examples/` | **demonstrates** | Shows the reasoning model applied to a real scenario |
| `references/` | **points** | Links to external standards, frameworks, and sources |

A file must serve exactly one of these verbs. If content serves two verbs, it belongs in two files — one per directory — with references between them. For example, JWT explained lives in `knowledge/secrets-management.md`. JWT checked lives in `checklists/secrets-management-review.md`. JWT classified lives in `references/owasp-and-vulnerability-standards.md`. Never combine these responsibilities in a single file.

## specs/

Definitions of what the project is, how it is structured, how it behaves, and how its content is organized. Each spec addresses one concern and is numbered sequentially. Specs are normative: they describe what the project intends, not how it is implemented.

- **Belongs:** Project-level definitions, scope boundaries, structural contracts.
- **Does not belong:** Security principles, checklists, worked examples, external references.

## principles/

Fundamental security engineering principles that guide the skill's reasoning. Each file covers one principle, stated plainly and explained concisely.

- **Belongs:** Core principles, heuristics, and rules of thumb.
- **Does not belong:** Implementation details, tool-specific guidance, step-by-step procedures.

## knowledge/

Self-contained entries on security engineering topics. Each file explains one topic: what it is, why it matters, and what to watch for.

- **Belongs:** Concept explanations, threat categories, attack patterns, defensive strategies.
- **Does not belong:** Checklists, worked walkthroughs, principles, external links.

## checklists/

Actionable, ordered lists for recurring security tasks. Each checklist is designed to be followed step by step.

- **Belongs:** Step-by-step verification lists, review guides, assessment procedures.
- **Does not belong:** Explanations of concepts, narrative examples, reference links.

## examples/

Worked walkthroughs that demonstrate how the skill's principles and knowledge apply in realistic scenarios. Each example should prioritize explanation and educational value over brevity.

- **Belongs:** Annotated walkthroughs, scenario analyses, before-and-after comparisons.
- **Does not belong:** Abstract principles, reference lists, checklists without context.

## references/

Curated pointers to external sources. Each entry states what the source covers and why it is relevant. References are supporting material, not the source of truth for the repository.

- **Belongs:** Links, book references, standards, specifications, further reading.
- **Does not belong:** Original content, explanations, checklists, examples.

# Modification Tiers

Not all files carry the same weight. Changes to higher-tier files affect everything below them. The higher the tier, the harder it is to modify.

| Tier | Files | Modification rule |
|------|-------|-------------------|
| **Tier 1** | `specs/SPEC-005-agent-operational-behavior.md` | **Never modify directly.** Document proposed changes as a comment in the relevant example file and flag for human review. |
| **Tier 2** | `specs/SPEC-001` through `SPEC-004` | Requires human review. Changes to architecture, behavior, or content organization affect every file in the repository. |
| **Tier 3** | `principles/*` | Requires human review. A principle change alters how the reasoning model interprets evidence. |
| **Tier 4** | `knowledge/*`, `checklists/*` | Agent may add new files following the contribution protocol. Agent may modify existing files only to add references or fix factual errors. |
| **Tier 5** | `examples/*`, `references/*` | Agent may add new files following the contribution protocol. Agent may extend existing files with new entries. |

The rule is simple: **the higher the tier, the more expensive the mistake.** A bad example teaches the wrong lesson. A bad principle corrupts every analysis. A bad SPEC-005 edit degrades the entire skill.

# Architectural Constraints

- Do not create new directories without a strong reason. The current structure covers the concerns the project needs.
- Avoid duplicate information across files. Prefer a reference to the canonical location.
- Add new files before expanding unrelated documents. One concern per file.
- Keep documents focused. If a file addresses two concerns, split it.
- Preserve portability. No platform-specific paths, tooling, or formats.

# Evolution Policy

The repository evolves incrementally. New content is added as individual files within existing directories. Structural changes — adding, removing, or renaming directories — should be rare, intentional, and documented in a spec update. When in doubt, add a file before adding a directory.