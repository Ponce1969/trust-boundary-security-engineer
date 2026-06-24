# AGENTS.md

## Purpose

This repository defines a portable AI Skill for Security Engineering. Its goal is to teach AI agents how to behave like responsible security engineers — thinking adversarially, defending methodically, and communicating clearly.

This is a documentation-first project. It contains no executable code.

## How to Use This Skill

Three entry points depending on what you need:

| Goal | Start here |
|------|-----------|
| Review an unfamiliar project for vulnerabilities | `checklists/repository-entry-scan.md` |
| Understand how to reason about security | `specs/SPEC-005-agent-operational-behavior.md` |
| Know what to load for a specific context | `SKILL.md` |

**The full cycle for a project review:**
1. Read `SKILL.md` — understand when this skill activates and what to load
2. Open the project and run `checklists/repository-entry-scan.md` — classify, scan, orient
3. Apply `specs/SPEC-005-agent-operational-behavior.md` — the six-step reasoning model
4. Use the checklist selected in step 2 for the deeper review
5. If you find something worth preserving → follow "When an Agent Learns from a Real Project" at the bottom of this file

---

## Project Philosophy

- **Knowledge is the asset.** The skill transmits expertise through structured documentation, not through scripts or tools.
- **Methodology over tools.** A security engineer's value comes from how they think, not which scanner they run.
- **Platform independence.** This skill must work across OpenCode, Claude Code, Cursor, Gemini CLI, and any agent that reads AGENTS.md or SKILL.md.
- **Less is more.** Every file, section, and rule must justify its existence. When in doubt, leave it out.
- **Education over fear.** The goal is to explain and teach, not overwhelm with warnings.
- **Evidence over assumptions.** Findings should be based on observable evidence whenever possible.

## What This Repository Is

- A portable AI Skill based on documentation.
- A structured collection of principles, checklists, knowledge, examples, and references.
- A teaching resource that shapes AI agent behavior toward security engineering best practices.
- A methodology reference for responsible Security Engineering practices.

## What This Repository Is Not

- A scanner, linter, or vulnerability detection tool.
- A Python package, framework, or library.
- An MCP server or executable service.
- A collection of scripts or automation tooling.
- A replacement for human security judgment or professional review.

## Repository Structure

```
security-engineer-skill/
├── README.md          # Project overview
├── LICENSE            # License terms
├── AGENTS.md          # Agent behavior rules (this file)
├── SKILL.md           # Skill definition and triggers
├── specs/             # Specification documents (SPEC-001 through SPEC-005)
├── principles/        # Security engineering principles
├── knowledge/         # Domain knowledge entries
├── checklists/        # Actionable security checklists
├── examples/          # Worked walkthroughs
└── references/        # External references and further reading
    ├── owasp-and-vulnerability-standards.md
    ├── owasp-llm-top10.md
    ├── threat-modeling-frameworks.md
    └── cloud-and-infrastructure-security.md
```

## Contribution Guidelines

- Write in clear English. Keep sentences short and scannable.
- Every contribution must align with the project purpose stated above.
- Prefer small, incremental changes over large rewrites.
- New files belong in existing directories. Do not create new directories without a strong reason.
- Do not introduce Python tooling, build systems, or dependency files. This project has no executable code.
- Do not add badges, installation instructions, or references to tools that do not exist in this repository.
- Review existing specs before adding content that overlaps with them.
- Avoid duplicating information across files. Prefer references over repetition.

## Design Principles

1. **Documentation-first.** If it isn't written down, it doesn't exist in this skill.
2. **Platform-agnostic.** Use Markdown and plain text only. No tool-specific formats.
3. **Composable.** Each file should address one concern. Avoid monolithic documents.
4. **Referential, not duplicative.** Link to knowledge, principles, or specs rather than restating them.
5. **Incremental.** Add one concept at a time. Validate before extending.
6. **Human-readable first.** Every document must make sense without an AI agent interpreting it.

## Incremental Development

- Start from the specs. Each spec defines what the skill covers and why.
- Fill directories one file at a time. A single well-written file beats an empty skeleton.
- Merge only when the content is clear, correct, and concise.
- When a file grows beyond a single concern, split it — don't create a new directory.
- If a principle, checklist entry, or example doesn't teach something specific, remove it.

## When an Agent Learns from a Real Project

This section is specifically for AI agents that have used this skill to review a real project, found real vulnerabilities, and want to contribute that learning back to the skill.

If you reviewed a project and found something worth preserving — a pattern that is not covered, a scenario that adds to the examples, a reference that belongs in the knowledge base — follow this protocol before creating or modifying any file.

### Decision tree

**Did you find a vulnerability pattern that no existing `knowledge/` file covers?**
→ Create `knowledge/<concept-name>.md`. Follow the structure of existing knowledge files: what it is, why it matters, common risks and misconceptions, relationship to other concepts, success criteria. Do not duplicate content that already exists — add a reference instead.

**Did you apply the skill's reasoning model to a real scenario and produce a full finding?**
→ Create `examples/<project-type>-<vulnerability-type>-review.md`. Follow the six-step structure of existing examples. Name the file after what the example demonstrates, not after the specific project. The goal is a walkthrough that teaches transferable reasoning, not a project-specific report.

**Did you encounter a standard, framework, or external source that would improve a reference file?**
→ Add an entry to the relevant file in `references/`. State what the source covers and why it is relevant. Keep the annotation to two sentences maximum.

**Did you find a new context signal that should trigger context loading in the skill?**
→ Update `SKILL.md` context loading rules only. Do not modify any other file for this purpose.

### Rules that must not be broken

- **Never modify `specs/SPEC-005-agent-operational-behavior.md` directly.** This is the core reasoning model. Changes to it affect every analysis the skill produces. If you believe the reasoning model needs updating, document the proposed change as a comment in the relevant example file and flag it for human review.
- **Never create a new directory.** All content belongs in existing directories. If no directory fits, the content may not belong in this skill.
- **Never add content that duplicates an existing file.** If the knowledge already exists, add a reference to it. Duplication inflates the skill without adding value.
- **Always update these three files when adding a new file:** `SKILL.md` (context loading), `README.md` (relevant table), `specs/SPEC-002-architecture.md` (structure). A file that is not registered in these three places will not be loaded by agents.

### Commit format for agent-generated learning

When committing changes that came from real project findings, use this prefix so the history shows what the skill absorbed:

```
learn: <what was found> in <project type>

Example:
learn: password reset token race condition in Python/PostgreSQL web app
learn: IAM wildcard policy in Terraform infrastructure review
learn: indirect prompt injection via knowledge base in research agent
```

This prefix distinguishes organic learning from deliberate design decisions and makes it easy to trace which real-world findings shaped the skill over time.
