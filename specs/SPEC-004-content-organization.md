# Purpose

Content without structure becomes noise. This document defines how to write and organize the content inside each directory of security-engineer-skill so the repository stays coherent as it grows.

This is not about behavior or architecture. It is about how the words inside the files should be written.

# General Writing Principles

- **One concern per file.** If a file addresses two topics, split it.
- **Prefer clarity over completeness.** A short, clear explanation beats an exhaustive one that nobody reads.
- **Human-readable first.** Every file must make sense to a person before it makes sense to an agent.
- **Avoid duplication.** State knowledge once. Reference it everywhere else.
- **Use references instead of repetition.** Link to the canonical location rather than restating content.
- **Keep documents small and focused.** If a file needs a table of contents, it is too long.

# Principles Files

Each file in `principles/` covers one principle.

- State the principle in a single sentence at the top.
- Explain why it matters. A principle without a reason is a rule without authority.
- Keep the explanation grounded. Avoid abstract or philosophical language.
- Do not include tool-specific guidance. Principles are about how to think, not what to use.
- A principle file should be short enough to read in under two minutes.

# Knowledge Files

Each file in `knowledge/` covers one topic.

- Start with what it is. Define the topic plainly before going deeper.
- Explain why it matters. Connect the topic to real security outcomes.
- Highlight common risks and misconceptions. These are often more useful than the correct path alone.
- Keep the scope narrow. One topic per file, no exceptions.
- Prefer explaining concepts over enumerating tools.
- When a topic naturally connects to a principle, reference it. Do not restate the principle.

# Checklist Files

Each file in `checklists/` is an ordered set of steps for a recurring security task.

- Steps must be actionable. Every item starts with a verb.
- Order matters. Place steps in the sequence they should be performed.
- Make the checklist easy to scan. Use short lines and consistent formatting.
- A checklist is suitable for repeated use. If the task is one-time, it is not a checklist.
- Do not embed explanations inside checklists. Link to the relevant knowledge file instead.

# Example Files

Each file in `examples/` is a worked walkthrough demonstrating how principles and knowledge apply in a realistic scenario.

- Show the reasoning process, not just the conclusion.
- Prioritize educational value over brevity. The reader should understand why, not only what.
- Explain the context. A finding without context is a fact without meaning.
- Include before-and-after comparisons when they clarify the improvement.
- Keep examples grounded in realistic scenarios. Avoid hypotheticals that do not occur in practice.
- Examples should teach transferable reasoning, not memorized solutions.

# Reference Files

Each file in `references/` curates pointers to external sources.

- References support knowledge. They are not the source of truth for the repository. That role belongs to the knowledge and principles files.
- State what each source covers and why it is relevant. A link without context is noise.
- Prefer authoritative sources: standards, specifications, well-maintained documentation, established references.
- Do not use references as a substitute for explaining a topic in the knowledge files.
- Keep annotations short. One or two sentences per source are enough.

# Growth Policy

- Prefer adding a new file over expanding an unrelated document. Growth means more files, not longer files.
- Split a file when it covers multiple concerns. The split is the signal that the content has matured.
- Keep complexity low. Every new file should justify its existence.
- Grow incrementally. Add one concept, validate it, then add the next.
- When content duplicates an existing entry, remove the duplicate and add a reference instead.