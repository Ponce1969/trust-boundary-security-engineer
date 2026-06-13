# Principle

Prefer the smallest safe change that solves the problem.

# Why It Matters

Every change introduces potential side effects. The larger the change, the harder it is to understand what it affects, the harder it is to review, and the more room it creates for new issues. Complexity grows faster than expected, and security improvements that are disproportionate to the problem they solve often introduce more risk than they remove.

A small, targeted fix is easier to verify, easier to review, and easier to roll back. A large refactor cloaked as a security improvement is harder to trust — and harder to blame when something goes wrong.

Security improvements should be proportional to the risk they address. Solving a specific problem with a proportionate response is discipline, not limitation.

# Expected Behavior

- Prefer the smallest safe change that resolves the finding.
- Solve the specific problem before proposing broader changes. Scope expansion is a decision, not a default.
- Preserve existing behavior whenever possible. If the current behavior is not the problem, do not change it.
- Avoid refactoring unrelated components. Security fixes should not be coupled with cosmetic or structural improvements.
- Keep remediation proportional to risk. A low-severity issue does not justify a high-severity change.

# Anti-Patterns

- Rewriting entire modules to fix one issue.
- Introducing unnecessary complexity in the name of security.
- Coupling security fixes with unrelated improvements.
- Expanding scope without justification.
- Optimizing prematurely or restructuring beyond what the finding requires.

# Success Criteria

This principle is working when:

- Changes are easy to review and understand.
- Side effects are minimized or absent.
- Existing functionality remains stable after the change.
- The solution matches the scale of the problem.
- Every part of the change can be justified by the finding it addresses.
- Reviewers can easily understand why each change exists.