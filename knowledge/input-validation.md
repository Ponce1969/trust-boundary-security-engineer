# What It Is

Input Validation means external data should never be trusted automatically. Validation determines whether input is acceptable before it is used, protecting systems from unexpected, malformed, or malicious data.

Validation is about defining acceptable input, not about guessing dangerous input. Whitelisting known-good patterns is more reliable than blacklisting known-bad ones, because the set of acceptable input is finite and predictable, while the set of malicious input is not.

Input Validation is a gate, not a filter. It decides what enters the system. Everything else is rejected.

# Why It Matters

Every external input crosses a trust boundary.

Users make mistakes. Forms receive typos, APIs receive unexpected formats, and integrations receive data that no one intended. Without validation, these mistakes propagate through the system and become failures in places far from where they entered.

Attackers intentionally provide unexpected input. Injection attacks, buffer overflows, and parameter manipulation all exploit systems that trust input without verifying it.

Invalid data propagates through systems. A poorly validated field does not cause a problem at the point of entry — it causes a problem where the data is used, which may be deep in business logic that was never designed to handle it.

Small input errors can become larger failures later. A missing null check at the boundary becomes a cascade of errors inside the system. Prevention is easier than correction.

# Common Risks and Misconceptions

- **Trusting client-side validation.** Client validation improves user experience. Server validation ensures security. The client can always be bypassed.
- **Validating too late.** Validation at the point of entry catches problems early. Validation deep in business logic means invalid data has already traveled through the system.
- **Trying to block bad input instead of defining good input.** Blacklists miss new attack patterns. Whitelists define what is acceptable and reject everything else.
- **Assuming internal systems always provide valid data.** Internal boundaries are still boundaries. A bug in one service should not cascade into another.
- **Treating validation as optional because "users know what they are doing."** Users do not always know what they are doing. And not every sender is a user.

Malformed input is not always malicious. A mistyped date, a truncated JSON field, or an unexpected character encoding can cause the same failures as a deliberate attack. Validation protects against mistakes and attacks alike.

# Relationship to Other Concepts

- **Secure by Default:** Secure by Default ensures the starting state is safe. Input Validation ensures the system remains safe when external data arrives. Together they protect both the initial state and every subsequent interaction.
- **Defense in Depth:** Input Validation is an early layer in Defense in Depth. When it fails or is bypassed, other layers should still contain the damage.
- **Evidence over Assumptions:** Input Validation is the practical application of Evidence over Assumptions. It replaces the assumption that input is valid with the evidence that it was verified.

# Success Criteria

Input Validation is working when:

- Invalid input is rejected early, before it reaches business logic.
- Systems fail predictably instead of unpredictably when they receive bad data.
- Errors are easier to diagnose because they are caught close to their origin.
- Data quality improves throughout the system because only verified data enters it.
- Unexpected input does not become unexpected behavior.