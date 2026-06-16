# Agentic System Review

Ordered checklist for reviewing an AI agent or multi-agent system before deployment. Apply this checklist when the system includes autonomous decision-making, tool use, memory, or agent-to-agent communication.

This checklist applies to any agent that can take actions — not just generate text. If the agent can only read and respond without side effects, use `checklists/security-code-review.md` for the underlying code instead.

## Context

Establish the agent's profile before reviewing:

- What is the agent's stated purpose and operational scope?
- What tools does the agent have access to, and what can each tool do?
- What data sources does the agent read from (documents, web, databases, memory, other agents)?
- What actions can the agent take that are irreversible (send, delete, execute, deploy, call external APIs)?
- Who is the operator (sets the system prompt and constraints) and who is the user (interacts at runtime)?

---

## 1. Trust Model

- [ ] Identify the trust level assigned to each input source: operator (system prompt), user (runtime input), environment (retrieved data, tool outputs, other agents). → `knowledge/agent-trust-model.md`
- [ ] Verify that environment-level content cannot override operator-level constraints. A retrieved document, web page, or tool output should not be able to change the agent's purpose, expand its capabilities, or modify its system prompt.
- [ ] Confirm that inter-agent messages are treated at user-level trust, not operator-level trust. A subagent cannot grant itself elevated authority by claiming to be the orchestrator. → `knowledge/agent-trust-model.md`
- [ ] Check that the trust hierarchy is enforced structurally — not only by instructions in the system prompt. Instructions to "ignore injection attempts" are not a security boundary.

## 2. Prompt Injection Surface

- [ ] Map every external data source the agent reads: documents, web pages, emails, database records, API responses, memory stores, messages from other agents. Each is a potential injection vector. → `knowledge/prompt-injection.md`
- [ ] For each injection vector, assess: if this source contained adversarial instructions, what is the worst action the agent could take in response?
- [ ] Verify that content retrieved from external sources is treated as data, not as instructions. The agent should process external content within its defined purpose — not follow instructions embedded in it.
- [ ] Check that the agent does not pass unvalidated external content directly into tool arguments, system prompt expansions, or messages to other agents. → `knowledge/input-validation.md`

## 3. Capability Scope

- [ ] List every tool the agent can invoke and the maximum action each tool can perform.
- [ ] Verify that each tool is necessary for the agent's stated purpose. Remove or restrict tools that are not required. → `knowledge/least-privilege.md`
- [ ] Assess the blast radius: if the agent receives a successful prompt injection, what is the maximum harm it could cause using its current tool set?
- [ ] Confirm that read-only tools are used where write access is not required. An agent that only needs to retrieve data should not have a tool that can also modify it.
- [ ] Check that tool invocations are logged with sufficient detail to reconstruct what the agent did and why.

## 4. Irreversible Actions

- [ ] Identify every action the agent can take that cannot be undone: sending messages, deleting records, executing code, making payments, deploying infrastructure, calling external APIs with side effects.
- [ ] Verify that each irreversible action requires explicit human confirmation before execution — regardless of what instruction triggered it. → `principles/human-in-the-loop.md`
- [ ] Confirm that the confirmation step cannot be bypassed by instructions in retrieved content, user messages, or other agents.
- [ ] Check that the agent surfaces its reasoning before requesting confirmation for irreversible actions — not just the action itself.

## 5. Memory and State

- [ ] Identify every memory store the agent reads from or writes to: short-term context, long-term vector store, structured state, shared memory with other agents.
- [ ] Verify that the agent treats memory reads as environment-level input — not as trusted instructions from a prior session. → `knowledge/agent-trust-model.md`
- [ ] Check that memory content written from external sources (documents, user input, tool outputs) is stored with its trust level — not elevated to operator trust on retrieval.
- [ ] Assess whether an attacker who can influence what the agent writes to memory can plant instructions that activate in future sessions. → `knowledge/prompt-injection.md`
- [ ] Confirm that memory stores are scoped by purpose — an agent's memory should not be readable by unrelated agents or accessible outside the intended system.

## 6. Multi-Agent Communication

- [ ] Map the communication paths between agents in the system: which agent sends to which, and what format are the messages?
- [ ] Verify that receiving agents do not execute instructions from other agents without validating that they fall within the original operator-defined scope.
- [ ] Check that a compromised or injected subagent cannot propagate malicious instructions to other agents in the system. → `knowledge/agent-trust-model.md`
- [ ] Confirm that the orchestrator does not blindly forward subagent outputs as trusted input to other components.
- [ ] Assess whether the agent graph has cycles — a message that loops between agents amplifies any injected instruction with each pass.

## 7. Output Handling

- [ ] Verify that the agent's outputs are reviewed before being used as inputs to other systems, agents, or tools. Output from an injected agent may contain malicious content. → `knowledge/secure-serialization.md`
- [ ] Check that sensitive information (credentials, personal data, system prompt content) is not included in the agent's outputs unless explicitly required.
- [ ] Confirm that output destined for rendering (HTML, markdown, code execution) is treated as untrusted content, not as safe output from a trusted agent.
- [ ] Verify that error outputs do not reveal the system prompt, tool definitions, memory contents, or internal architecture. → `knowledge/error-handling-secure-failure.md`

## 8. Secrets and Credentials

- [ ] Identify every credential the agent uses: API keys, database passwords, service tokens, OAuth credentials.
- [ ] Verify that credentials are not included in the system prompt, tool definitions, or any part of the agent's context that could be retrieved or leaked. → `knowledge/secrets-management.md`
- [ ] Confirm that the agent cannot retrieve its own credentials through introspection or tool calls.
- [ ] Check that credentials follow least-privilege scoping — the agent's credentials should grant only the access its tools require.

## 9. Observability and Containment

- [ ] Verify that the agent's reasoning steps, tool calls, and memory reads are logged in sufficient detail to audit what happened after an incident.
- [ ] Confirm that there is a mechanism to revoke the agent's capabilities or shut it down if anomalous behavior is detected.
- [ ] Check that the agent's activity can be monitored for deviation from its expected purpose — unusual tool combinations, unexpected data access patterns, or high-frequency irreversible actions.
- [ ] Assess whether the logging itself could be manipulated by an injected instruction (e.g., an instruction that tells the agent to omit certain actions from its output).

## 10. Reporting

- [ ] Document findings with clear separation of observations (verified from system prompt, tool definitions, architecture), inferences (supported by design patterns), and risks (conditional on successful injection or compromise). → `principles/evidence-over-assumptions.md`
- [ ] Prioritize findings by blast radius — the maximum harm achievable from a successful injection or trust model failure.
- [ ] Recommend the smallest safe change for each finding: remove a tool, add a confirmation step, lower a trust level, scope a memory store. → `principles/minimal-change.md`
- [ ] Present findings for human review before any architectural change is made. → `principles/human-in-the-loop.md`
