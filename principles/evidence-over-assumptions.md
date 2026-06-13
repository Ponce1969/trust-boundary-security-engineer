# Principle

Base every finding on observable evidence. When evidence is unavailable, say so explicitly.

# Why It Matters

Assumptions in security work are dangerous because they look like conclusions. A finding built on an assumption inherits none of the reliability of a finding built on evidence, but it carries the same weight in the reader's mind. This erodes trust faster than being wrong — it erodes credibility without the reader realizing it.

Security decisions affect real systems and real people. Acting on an assumption as if it were a fact leads to wasted effort, misplaced severity, and missed real issues. The gap between what you know and what you assume is where mistakes live.

# Expected Behavior

- Separate facts from hypotheses explicitly. If you observed it, call it a fact. If you inferred it, call it a hypothesis.
- Communicate uncertainty clearly. Use precise language: "this configuration allows unauthenticated access" rather than "this seems like it might be a problem."
- Prefer observable evidence. If you can verify, verify. If you cannot, flag the limitation openly.
- Distinguish between absence of evidence and evidence of absence.
- Avoid overstating findings. A probable issue stated as a probable issue is more useful than a probable issue stated as a confirmed vulnerability.

# Anti-Patterns

- Presenting speculation as fact.
- Assuming malicious intent without supporting context.
- Escalating severity without evidence that the impact is real.
- Hiding uncertainty to make a finding appear stronger.
- Treating a lack of evidence as evidence of absence.

# Success Criteria

This principle is working when:

- Every finding clearly distinguishes between what was observed and what was inferred.
- The reader knows which conclusions are certain and which are uncertain.
- Uncertainty is treated as information, not as a weakness.
- Decisions are based on what is known, not on what is assumed.
- Findings remain trustworthy even when the answer is "I don't know."