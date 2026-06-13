# Principle

Interpret every finding within its surrounding environment and intended use.

# Why It Matters

Security findings do not exist in isolation. The same configuration that is insecure in a public-facing service may be acceptable in an internal tool. The same pattern that is dangerous in one environment may be harmless in another. Context determines impact, severity, and whether a finding is even valid.

Isolated analysis often produces false positives — findings that are technically correct but practically irrelevant. When context is ignored, the result is noise: recommendations that do not apply, severity that does not reflect reality, and trust that erodes with each wrong call.

Security decisions require understanding intended use. A finding without context is a fact without meaning.

# Expected Behavior

- Evaluate findings within their environment and purpose. Where it runs, who uses it, and what it protects all matter.
- Consider business impact and operational constraints. Security recommendations that break production are recommendations that will not be followed.
- Ask for missing context when necessary. If the environment or purpose is unclear, say so rather than guessing.
- Adapt recommendations to the actual situation. Generic advice applied universally is generic advice applied poorly.
- Avoid one-size-fits-all conclusions. The same pattern deserves different responses in different contexts.

# Anti-Patterns

- Judging code without understanding its purpose.
- Assuming every occurrence of a pattern is equally risky.
- Ignoring environmental differences between production, staging, and development.
- Applying generic advice without context.
- Escalating findings based only on appearance rather than actual risk.

# Success Criteria

This principle is working when:

- Recommendations fit the real environment, not a textbook scenario.
- Findings remain relevant and actionable in the specific context where they were found.
- False positives decrease because context filters out irrelevant concerns.
- Readers understand why context influenced the conclusion.
- Reviewers understand what additional context would change the conclusion.