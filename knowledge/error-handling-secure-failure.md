# What It Is

Secure Failure means that when a system encounters an error, it fails in a way that preserves security. It does not expose sensitive information, grant unintended access, or leave the system in a vulnerable state.

Error handling is the mechanism. Secure failure is the outcome. Not all error handling produces secure failure — an error message that reveals internal state, a fallback that skips authentication, or a crash that leaves data exposed are all error handling that fails insecurely.

The principle is simple: when something goes wrong, the system should become more restrictive, not less.

# Why It Matters

Errors are inevitable. Systems fail, networks drop, configurations drift, and inputs arrive in unexpected formats. The question is not whether errors will occur, but what happens when they do.

Insecure error handling is one of the most common sources of vulnerability. Error messages that reveal stack traces, database schemas, or internal paths give attackers information they cannot obtain through normal interaction. Fallback mechanisms that bypass authentication or authorization create paths that attackers can trigger intentionally.

Failures cascade. An error in one component often triggers behavior in another. If the second component assumes the first is in a valid state, it may make incorrect security decisions based on invalid assumptions. Secure failure ensures that errors do not propagate as elevated privilege, exposed data, or bypassed controls.

The most dangerous secure failure violation is the implicit bypass: a fallback that defaults to allowing access when an error occurs, because denying access might disrupt users. This inverts the security model — instead of failing closed, the system fails open.

# Common Risks and Misconceptions

- **Returning detailed error messages to clients.** Stack traces, database error messages, and internal paths leak information about the system that aids targeted attacks.
- **Defaulting to allowing access on error.** If authentication or authorization fails with an error, the safe default is to deny access. Failing open creates an exploitable path.
- **Logging sensitive data in error handlers.** Passwords, tokens, and personal data should never appear in logs, even in error states.
- **Treating error handling as an afterthought.** Error paths are security paths. They deserve the same scrutiny as the happy path.
- **Assuming that catching an exception is handling it.** Catching an exception without defining a secure response is ignoring it. An empty catch block is a security decision — and usually a bad one.
- **Ignoring error paths during review.** Most code review focuses on the happy path. Attackers focus on the error paths. A system is only as secure as its least secure error handler.

# Relationship to Other Concepts

- **Input Validation:** Input Validation prevents malformed data from entering the system. Secure Failure determines what happens when something reaches the system that validation did not catch or when internal processing fails. Together, they protect both the entry point and the failure point.
- **Secure by Default:** Secure by Default ensures the starting state is safe. Secure Failure ensures the failure state is safe. Both define what the system does when things go right and when they go wrong.
- **Defense in Depth:** Secure Failure is a Defense in Depth principle. If one layer fails, the failure mode of that layer should not expose the layers behind it.
- **Trust Boundaries:** Errors often occur at trust boundaries. Secure Failure ensures that when data crosses a boundary and something goes wrong, the result is more restriction, not less.

# Success Criteria

Secure Failure is working when:

- Error responses reveal no information about internal state, system structure, or sensitive data.
- Authentication and authorization failures default to denying access, not granting it.
- Sensitive data does not appear in error logs or error messages.
- Error handlers define explicit, secure responses rather than relying on implicit defaults.
- Error paths receive the same scrutiny during review as the happy path.