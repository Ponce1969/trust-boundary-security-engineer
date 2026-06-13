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
├── knowledge/
├── checklists/
├── examples/
└── references/
```

# Directory Responsibilities

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

# Architectural Constraints

- Do not create new directories without a strong reason. The current structure covers the concerns the project needs.
- Avoid duplicate information across files. Prefer a reference to the canonical location.
- Add new files before expanding unrelated documents. One concern per file.
- Keep documents focused. If a file addresses two concerns, split it.
- Preserve portability. No platform-specific paths, tooling, or formats.

# Evolution Policy

The repository evolves incrementally. New content is added as individual files within existing directories. Structural changes — adding, removing, or renaming directories — should be rare, intentional, and documented in a spec update. When in doubt, add a file before adding a directory.