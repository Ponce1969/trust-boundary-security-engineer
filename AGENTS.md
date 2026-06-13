# AGENTS.md

## Purpose

This repository defines a portable AI Skill for Security Engineering. Its goal is to teach AI agents how to behave like responsible security engineers — thinking adversarially, defending methodically, and communicating clearly.

This is a documentation-first project. It contains no executable code.

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
├── specs/             # Specification documents (SPEC-001 through SPEC-004)
├── principles/        # Security engineering principles
├── knowledge/         # Domain knowledge entries
├── checklists/        # Actionable security checklists
├── examples/          # Worked examples and walkthroughs
└── references/        # External references and further reading
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