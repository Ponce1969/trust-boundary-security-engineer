# What It Is

An agent trust model is the explicit definition of which actors and data sources an agent treats as authoritative, and under what conditions that trust is granted.

In conventional software, trust is enforced structurally: a database connection uses authenticated credentials, an HTTP server validates headers, a process boundary enforces isolation. The runtime makes trust decisions before application code runs.

In agentic systems, trust is often implicit. An agent receives instructions from an orchestrator, retrieves data from a tool, communicates with other agents, and reads from memory — frequently without any mechanism that verifies the identity or integrity of the source. Trust is assumed based on position in the conversation, not based on authentication.

This implicit trust is the foundation of most agentic security failures. An agent that trusts its orchestrator unconditionally can be manipulated by anything that can impersonate or inject into the orchestrator. An agent that trusts tool outputs can be manipulated by anything that controls those tools. An agent that trusts its own memory can be manipulated by anything that has written to that memory.

# The Agentic Trust Hierarchy

A typical agentic system has three trust levels that must be modeled explicitly:

**Operator trust** — The highest trust level. Instructions from the operator (system prompt, configuration, deployment context) define the agent's purpose, constraints, and capabilities. These instructions should be the only unconditionally trusted input. They should not be modifiable at runtime by users, tools, or other agents.

**User trust** — Instructions and requests from the human or system that interacts with the agent at runtime. User input is trusted to the degree the operator has explicitly granted. A user should not be able to override operator-level constraints, expand the agent's capabilities beyond what the operator defined, or impersonate the operator.

**Environment trust** — The lowest trust level. Everything the agent reads from the world: tool outputs, retrieved documents, web pages, database records, messages from other agents, responses from external APIs. This content is untrusted by default. It may contain injected instructions, false information, or manipulated data. It should never be treated as authoritative — only as content to be processed within the constraints the operator defined. → `knowledge/prompt-injection.md`

The most common trust model failure is **trust level collapse**: when environment-level content is processed in a way that allows it to acquire operator-level authority. This is what prompt injection exploits. → `knowledge/prompt-injection.md`

# Multi-Agent Trust

When an agent communicates with other agents, the trust model must explicitly answer:

- What trust level does this agent grant to messages from other agents?
- How does the receiving agent verify that a message actually comes from the claimed sender?
- Can a subagent escalate its own trust level by claiming to be the orchestrator?
- If a subagent is compromised or injected, can it pass malicious instructions to other agents in the same system?

These questions do not have answers in most current agentic frameworks. The default behavior — trusting messages from agents in the same system as if they were operator-level instructions — is the equivalent of trusting any network packet that arrives on a local subnet as if it came from a trusted server.

**The trust level of a message from another agent should be at most the trust level of the agent that sent it.** An orchestrator does not elevate the trust of its subagents' outputs by forwarding them. A subagent cannot grant itself elevated trust by claiming to be the orchestrator.

# Capability Scoping

Trust and capability are related but distinct. An agent can be granted operator-level trust with minimal capabilities, or user-level trust with broad capabilities. Both combinations have different risk profiles.

The correct approach is to scope both independently:

- Grant the minimum trust level that allows the agent to function correctly.
- Grant the minimum capabilities (tools, data access, output channels) that the task requires.
- Treat the combination of high trust + broad capabilities as the highest-risk configuration.

An agent that is both highly trusted and broadly capable creates a single point of failure: if it is compromised, injected, or manipulated, the attacker inherits both the trust and the capabilities. → `principles/control-sufficiency.md`

# Irreversibility and Human Oversight

Trust decisions become more consequential when they enable irreversible actions. Sending an email, deleting a record, executing code, calling an external API — these actions cannot be undone after the fact.

The trust model must explicitly identify which actions are irreversible and require human confirmation before execution, regardless of the trust level of the instruction that requested them. This is not a limitation on the agent's capability — it is a structural safeguard against trust model failures causing permanent harm. → `principles/human-in-the-loop.md`

A useful heuristic: the more irreversible an action, the higher the trust threshold required to execute it without confirmation. Reversible actions (reading, summarizing, drafting) can proceed with user-level trust. Irreversible actions (sending, deleting, executing, deploying) should require operator-level confirmation or human review.

# Memory as a Trust Boundary

Agent memory — whether short-term context, long-term vector store, or structured state — is a trust boundary. → `knowledge/trust-boundaries.md`

What the agent writes to memory today becomes input to future reasoning. If an attacker can influence what the agent writes to its memory — through injected instructions, false tool outputs, or manipulated documents — they can plant beliefs, instructions, or behavioral biases that activate in future sessions.

This is a form of persistent injection: the attacker does not need access to the current session. They need only access to a data source that the agent will read and store. Once the malicious content is in memory, it influences every subsequent session that retrieves it.

The trust model must treat memory reads with the same skepticism as any other external data source. Content retrieved from memory is environment-level trust, not operator-level trust — even if the agent itself wrote it in a previous session.

# Semantic Race Condition

In concurrent multi-agent systems, trust is not only a spatial property — it is also temporal. The trust level effectively assigned to an instruction depends on *when* that instruction entered the agent's context, not only *where* it came from.

A **semantic race condition** occurs when an attacker manipulates the timing or asynchronous flow of messages to inject environment-level content into the agent's episodic memory at a moment when the agent's attention is degraded or its context is saturated. Under these conditions, the injected content may be processed as if it carried higher trust than it would receive in isolation.

This creates a class of vulnerability that static analysis cannot detect. The code may be correct. The trust hierarchy may be correctly defined. The injection succeeds because of *when* the malicious content arrived relative to the agent's execution state.

**How it manifests:**

- An orchestrator receives asynchronous responses from multiple subagents simultaneously. The responses arrive out of order. A malicious response crafted to resemble an authoritative subagent output arrives at the moment the orchestrator is processing a high-priority task, and is incorporated into the working context without full evaluation.
- A long-running agent accumulates context across many tool calls. As the context window fills, the model's effective attention to early operator instructions degrades. An injection planted late in the session — after significant context accumulation — has a higher probability of being treated as authoritative than the same injection at session start.
- Two subagents write to shared memory concurrently. One write is a legitimate update; the other is an injected instruction. The order of resolution determines which one shapes the orchestrator's next reasoning step.

**What to look for when reviewing a multi-agent system:**

- Does the orchestrator process subagent responses sequentially with full evaluation, or does it merge concurrent responses without ordering guarantees?
- Is there a mechanism to detect context saturation and reset or summarize the working memory before it degrades attention to operator-level constraints?
- Are writes to shared memory from multiple agents serialized and validated, or is the last writer's content accepted without verification?
- Does the system treat the timing of an instruction's arrival as irrelevant to its trust level, or does it have safeguards that maintain trust evaluation independent of execution order?

A semantic race condition is a time-dependent finding. It does not manifest on every run. It manifests under specific interleavings of concurrent execution — which means it may not appear during testing but will appear under adversarial conditions designed to exploit the timing. → `specs/SPEC-005-agent-operational-behavior.md`

# Relationship to Other Concepts

- **Trust Boundaries:** The agentic trust hierarchy is an extension of spatial trust boundaries to the agent's internal authority model. The boundary between operator trust and environment trust is the most critical one to maintain. → `knowledge/trust-boundaries.md`
- **Prompt Injection:** Trust level collapse is the mechanism by which prompt injection succeeds. Maintaining explicit trust levels is the structural defense. → `knowledge/prompt-injection.md`
- **Least Privilege:** Capability scoping is the application of least privilege to agent design. → `knowledge/least-privilege.md`
- **Defense in Depth:** No single trust mechanism is sufficient. Layered trust enforcement — input filtering, capability restriction, human oversight for irreversible actions, output monitoring — reduces the impact of any single failure. → `knowledge/defense-in-depth.md`

# Common Risks and Misconceptions

- **"The orchestrator controls the subagents, so they're trusted."** Control is not authentication. An orchestrator that passes instructions in plaintext without a signature or verification mechanism can be impersonated by anything that can inject into the communication channel.
- **"We trust our own agents."** A subagent that has been successfully injected is no longer acting on behalf of the system. Trust should be granted to verified behavior, not to assumed identity.
- **"Memory is internal, so it's safe."** Memory that was written from external data inherits the trust level of that data. Past sessions do not elevate trust.
- **"We review the agent's outputs."** Output review catches the results of trust failures, but does not prevent them. The trust model must be enforced before the action, not after.

# Success Criteria

The agent trust model is working when:

- Every instruction source is assigned an explicit trust level before the agent processes it.
- Environment-level content cannot acquire operator-level authority, regardless of how it is framed.
- Inter-agent messages are treated at user-level trust unless verified by an operator-controlled mechanism.
- Irreversible actions require a trust level and confirmation mechanism proportional to their impact.
- Memory reads are treated as environment-level input, not as trusted instructions from a prior self.
