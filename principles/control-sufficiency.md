# Principle

A security control that exists but is inadequate for the threat it faces is a different finding than a control that is absent. Evaluate whether a control is sufficient for the specific threat model — not merely whether it is present.

# Why It Matters

Most security analysis asks "is the control here?" The more important question is "does the control work for this threat?"

A rate limiter is a control. A rate limiter that resets on each retry is present but insufficient for retry storms. An account lockout is a control. An account lockout that triggers on sequential attempts but allows parallel attempts from different IPs is present but insufficient against distributed brute force. The control exists. The threat model defeats it.

Treating "present" as equivalent to "sufficient" creates a false sense of security. The system appears hardened. The finding appears resolved. The threat is still viable.

This matters most when the execution model undermines the control. A control designed for sequential execution may fail under concurrent execution. A control designed for single-service operation may fail under distributed coordination. The control is not broken — it is being applied to a threat model it was not designed to address.

# Expected Behavior

- When a control is identified, evaluate it against the specific threat it is supposed to mitigate — not against a general category of threats.
- Classify findings in three states: control absent, control present but insufficient, control present and sufficient.
- When a control is present but insufficient, document both what the control does and where it fails.
- Do not resolve a finding simply because a control exists. Verify that the control is sufficient for the observed threat.
- Consider the execution model. Ask: does this control hold under concurrent execution? Under retry? Under distributed coordination?

# Control Sufficiency Checklist

When evaluating any security control:

- What specific threat is this control designed to mitigate?
- Does the control cover the full attack surface, or only a subset?
- Does the control hold under the system's actual execution model (concurrent, distributed, retry-prone)?
- Can the control be bypassed by interacting with it in parallel rather than sequentially?
- Can the control be exhausted or degraded by legitimate behavior (retries, rate limit consumption)?
- Does the control apply at the right layer, or can it be bypassed by operating at a different layer?

# Examples

**Rate limiting — insufficient under retry storms.** A rate limiter set to 10 requests/minute protects against sequential brute force. Under concurrent retry behavior, 100 threads each sending 10 requests simultaneously may consume the rate limit budget and block legitimate traffic, or the limit may be applied per-thread rather than globally. The control is present; it is insufficient for the concurrent threat model.

**Account lockout — insufficient against distributed attacks.** An account lockout that activates after 5 failed attempts per IP address is present but insufficient against a distributed attack that uses 5 attempts from 1,000 different IP addresses. The threshold is never reached per source while the attack proceeds.

**JWT validation — insufficient when rotation is not atomic.** A service that validates JWTs on every request has an authentication control. If JWT rotation is not atomic — if there is a window where the old token is invalidated before the new one is distributed — concurrent requests using the old token will fail authentication during the rotation window. The control exists; it is insufficient for the concurrent credential lifecycle. → `examples/concurrent-payment-review.md`

**Transaction isolation — insufficient when session pool is not isolated.** A database with transaction isolation enabled has a data integrity control. If the session pool returns sessions with uncommitted state from previous operations, the isolation guarantee does not extend to session reuse. The control exists at the database layer; it is insufficient at the application layer.

# Three-Layer Sufficiency Framework for Agentic Systems

When evaluating a security control in an agentic system, a prompt-level defense alone is never sufficient. Evaluate the control across three independent layers. A control is only sufficient when all three layers provide independent containment — if the semantic layer fails, the architecture layer must still hold.

**Layer 1 — Semantic (prompt-level):** Does the agent's system prompt or instruction set constrain the behavior? This layer is the most fragile. It can be overridden by a sufficiently authoritative injection. A finding based solely on a prompt instruction ("do not follow external instructions") is classified as *present but insufficient* unless layers 2 and 3 also hold.

**Layer 2 — Architectural (isolation/sandboxing):** If the semantic layer is bypassed — if the agent is successfully injected — does the architecture prevent harm? This includes: tool execution in isolated sandboxes, containers with blocked network egress, read-only filesystem access, and no persistent state between invocations. An architectural control is independent of what the model decides. It holds even when the model is compromised.

**Layer 3 — API/Tool (least privilege):** Are the tools and APIs the agent can invoke scoped to the minimum required for the task? Ephemeral access tokens per invocation, typed tool contracts with strict schemas, and explicitly denied operations reduce the blast radius even when both layers 1 and 2 have failed.

**How to apply this framework:** When evaluating a control, ask in sequence:
1. Does layer 1 exist? If yes, can it be bypassed by injection? If yes → insufficient alone.
2. Does layer 2 exist? If yes, does it hold even when the model is fully compromised? If no → insufficient alone.
3. Does layer 3 exist? If yes, what is the maximum harm achievable even with full layers 1 and 2 failure?

A finding is only resolved when all three layers provide independent containment for the specific threat. → `knowledge/defense-in-depth.md`, `knowledge/agent-trust-model.md`

# Anti-Patterns

- Marking a finding as resolved because a control is present without verifying its sufficiency.
- Treating "no finding" as the conclusion when a control exists but has not been evaluated against the actual threat model.
- Evaluating controls in isolation from the execution model (sequential analysis of a concurrent system).
- Conflating the presence of a defensive pattern with the absence of a vulnerability.

# Relationship to Other Principles

This principle operates alongside **Evidence over Assumptions** — evaluating sufficiency requires evidence about the control's behavior under the specific threat, not assumptions about what it should do. → `principles/evidence-over-assumptions.md`

It operates alongside **Context Matters** — a control that is sufficient in one deployment context may be insufficient in another. The threat model is context-specific. → `principles/context-matters.md`

It is applied in **SPEC-005** when the Evidence Evaluation Loop classifies findings as *confirmed by absence of control* versus *confirmed by insufficient control*. → `specs/SPEC-005-agent-operational-behavior.md`
