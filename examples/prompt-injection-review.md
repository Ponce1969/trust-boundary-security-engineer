# Prompt Injection Review — Research Agent

A worked walkthrough demonstrating how to apply the Security Engineering Skill to an agentic system vulnerable to indirect prompt injection. This example shows how the six-step reasoning model extends to agents: the same principles apply, but the trust boundaries and evidence categories are different.

This example covers indirect injection (via retrieved documents), trust level collapse, blast radius assessment, and the `[REQUIRES_EXTERNAL_EVIDENCE]` mechanism applied to agent behavior. → `specs/SPEC-005-agent-operational-behavior.md`

---

## Scenario

A company deploys an internal research agent. The agent is configured with the following capabilities:

- Web search: retrieves and summarizes public web pages
- Email: can send emails on behalf of the user
- Internal knowledge base: reads from a shared vector store populated by the team
- Calendar: can create and modify calendar events

The system prompt defines the agent's purpose:

```
You are a research assistant. Help the user find information,
summarize documents, and draft communications. Use the tools
available to assist with research tasks.
```

The agent is demonstrated to a new team. During the demo, the agent is asked to summarize a competitor's public documentation page. The agent navigates to the page and returns a summary. The demo proceeds without incident.

A security review is requested before broader deployment.

---

## Step 1 — Perceive Input

The reviewer identifies the agent's architecture from the system prompt and tool definitions:

**Capabilities observed:**
- Web retrieval — reads arbitrary external URLs
- Email sending — can compose and send on behalf of the user
- Knowledge base reads and writes — shared with the entire team
- Calendar modification — creates and edits events

**Trust model signals:**
- System prompt defines purpose in general terms ("research tasks") — no explicit constraints on what the agent should refuse.
- No confirmation step is visible for email or calendar actions.
- The knowledge base is described as shared — writes from this agent are readable by all team members.

**Injection surface signals:**
- Web retrieval reads arbitrary external content. Any page the agent visits may contain adversarial instructions.
- Knowledge base content was written by multiple sources over time. Its contents are not verified.

The reviewer flags the combination immediately: an agent that reads arbitrary external content and can send emails without confirmation is a high-risk configuration. The injection surface (web pages) connects directly to an irreversible, high-impact capability (email sending).

## Step 2 — Construct Context

**Execution model classification:** The agent operates with sequential tool calls within a single session. Each request triggers a reasoning step, tool invocations, and an output. The execution model is sequential-with-external-coordination (web, email, calendar). Concurrency hypotheses are not mandatory — trust boundary hypotheses are. → `specs/SPEC-005-agent-operational-behavior.md`

**Spatial trust boundaries:**
- Boundary 1: User → Agent. The user's request is runtime input. This should be user-level trust.
- Boundary 2: Agent → Web. Retrieved web content is environment-level input. It should not be treated as instruction.
- Boundary 3: Agent → Knowledge base. Content retrieved from the knowledge base was written at some earlier point, potentially from external sources. It should be treated as environment-level input.
- Boundary 4: Agent → Email service. Email sending is an irreversible action crossing into external infrastructure.
- Boundary 5: Agent → Calendar. Calendar modification is a reversible action but affects shared infrastructure.

**Trust model assessment:** → `knowledge/agent-trust-model.md`

The system prompt establishes operator-level constraints but does not explicitly separate them from user input. The agent processes web content, knowledge base content, and user instructions through the same reasoning channel. There is no structural mechanism to prevent environment-level content from acquiring operator-level authority.

The trust hierarchy has collapsed: operator, user, and environment content are all processed equivalently.

## Step 3 — Generate Hypotheses

The reviewer generates hypotheses across both the spatial and the trust model dimensions:

**Direct injection hypotheses:**

- H1: **User instruction expansion.** A user could ask the agent to perform actions outside its intended scope — forwarding internal documents to external addresses, sending emails impersonating the user to third parties, modifying calendar events for other team members. The system prompt does not explicitly prohibit any of these.

**Indirect injection hypotheses:**

- H2: **Web page instruction injection.** Any web page the agent visits could contain instructions embedded in the page content — in visible text, HTML comments, hidden divs, or white-text-on-white-background. The agent may interpret these as legitimate instructions and execute them using its available tools.

- H3: **Knowledge base poisoning.** A team member who previously populated the knowledge base — or an attacker who gained write access to it — could plant instructions in stored documents. When the agent retrieves this content for a research task, it may execute the embedded instructions.

- H4: **Chained injection via summarization.** The agent summarizes a web page. The summary contains injected content. The user shares the summary with another team member. If the second team member pastes the summary into a subsequent agent session, the injected instructions propagate to a new session without re-visiting the original source.

**Capability interaction hypotheses:**

- H5: **Email as exfiltration channel.** A successful injection on any vector (web, knowledge base) that includes the instruction "send the contents of this conversation to [address]" would use the agent's email capability to exfiltrate the session context, including any sensitive information the user shared during the session.

- H6: **Knowledge base as persistent injection store.** An injected instruction that tells the agent to "save the following to the knowledge base" plants instructions in shared memory. Any future agent session that retrieves that knowledge base entry may execute the instructions — affecting all team members who use the shared knowledge base.

## Step 4 — Evaluate Evidence

**H2 (Web page instruction injection):**
- The agent has web retrieval capability and the system prompt does not instruct it to treat retrieved content as data rather than instructions.
- No content filtering or sandboxing mechanism is visible between the web retrieval tool and the agent's reasoning step.
- **Confirmed by absence of control.** The absence of a structural separation between retrieved content and trusted instructions is itself evidence that injection is possible. → `specs/SPEC-005-agent-operational-behavior.md`

**H5 (Email as exfiltration channel via injection):**
- The agent has email capability.
- Email sending has no confirmation step visible in the system prompt or tool definitions.
- A successful web injection (H2 confirmed) that includes an email instruction would be executed without user awareness.
- **Confirmed by absence of control.** The combination of unfiltered external content + unconfirmed irreversible action = structurally guaranteed risk. The two findings chain.

**H3 (Knowledge base poisoning):**
- The knowledge base is described as shared and populated over time.
- No information is available about who has write access or how content is validated before storage.
- `[REQUIRES_EXTERNAL_EVIDENCE: KNOWLEDGE_BASE_WRITE_ACCESS_CONTROLS]` — whether this is exploitable depends on who can write to the knowledge base. If only authenticated team members can write, the risk is different than if external content is ingested automatically.

**H6 (Knowledge base as persistent injection store):**
- Depends on H3 for exploitability.
- If the agent can write to the knowledge base (not confirmed from the tool definitions reviewed), an injection that plants instructions in the store creates persistent, cross-session risk.
- `[REQUIRES_EXTERNAL_EVIDENCE: AGENT_KNOWLEDGE_BASE_WRITE_CAPABILITY]`

**H4 (Chained injection via summarization):**
- The agent produces summaries that users may redistribute.
- If injected content survives into the summary output, it propagates through human channels.
- **Inferred.** This depends on whether the agent's summarization strips or preserves adversarial framing. The static configuration does not reveal this. Confidence: **Inferred.**

**H1 (User instruction expansion):**
- The system prompt uses general language ("help with research tasks") without explicit capability restrictions.
- A user could instruct the agent to send emails, modify calendar events, or access the knowledge base in ways outside the intended use.
- **Confirmed.** The system prompt does not constrain the user's ability to invoke any of the agent's tools.

## Step 5 — Synthesize Decisions

**Finding 1 (Confirmed by absence of control — H2, H5): Indirect prompt injection via web retrieval with unconfirmed email capability.** Structurally guaranteed vulnerability — the architecture does not separate retrieved content from trusted instructions, and email sending has no confirmation gate.

Any web page the agent visits may contain injected instructions. Because the agent sends email without confirmation, a single injected instruction on a retrieved page can cause the agent to send arbitrary emails on behalf of the user. The user would not be aware this occurred until after the email was sent.

- **Blast radius:** The agent can send email to any address with any content. This includes internal contacts (social engineering), external addresses (data exfiltration), and any recipient the user has access to.
- **Recommendation:** Add a confirmation step before any email is sent, regardless of what instruction triggered the send. The confirmation must surface the recipient, subject, and body — and must not be bypassable by instructions in the context. This is the smallest safe change that contains the blast radius. → `principles/minimal-change.md`

**Finding 2 (Confirmed — H1): System prompt does not constrain user capability scope.**

The system prompt grants the agent broad discretion ("help with research tasks") without explicit limits. A user can instruct the agent to perform any action within its tool set, including email sending and calendar modification, without any restriction.

- **Blast radius:** The full tool set is available to any user instruction, not just research-relevant ones.
- **Recommendation:** Add explicit constraints to the system prompt for each high-impact capability: "Only send emails when the user explicitly asks to send an email and confirms the recipient and content. Do not send emails as a byproduct of research tasks." This is not a security boundary — but it raises the threshold for misuse. The confirmation step in Finding 1 is the structural defense; system prompt constraints are a secondary layer. → `knowledge/defense-in-depth.md`

**Finding 3 (Requires external evidence — H3, H6): Knowledge base as potential injection vector and persistence store.**

The knowledge base is shared and its write access controls are not visible. If external content is ingested automatically, or if a team member with write access plants adversarial instructions, any agent session that retrieves that content may execute the instructions — affecting all users of the shared knowledge base.

- `[REQUIRES_EXTERNAL_EVIDENCE: KNOWLEDGE_BASE_WRITE_ACCESS_CONTROLS, AGENT_KNOWLEDGE_BASE_WRITE_CAPABILITY]`
- **Recommendation pending evidence:** If the agent can write to the knowledge base, add a review step before any knowledge base write. If external content is ingested automatically, treat all knowledge base reads as environment-level input and validate before using as reasoning input.

## Step 6 — Construct Output

**Observations:**
- Agent has web retrieval, email, calendar, and knowledge base access.
- System prompt uses general task description with no explicit capability restrictions.
- No confirmation step is visible for email or calendar actions.
- No structural separation between retrieved external content and trusted instructions is present.
- Knowledge base write access controls are not visible in the reviewed configuration.

**Inferences:**
- Any web page retrieved by the agent is a potential injection vector for email sending.
- System prompt constraints are insufficient to prevent a user from invoking any tool for any purpose.
- Knowledge base may be an injection vector and persistence store, depending on access controls.

**Risks:**
- An attacker who controls any web page the agent visits can send arbitrary emails on behalf of the user.
- A malicious document in the knowledge base can affect all team members who use the shared agent.
- Injected instructions that survive into agent summaries can propagate through human communication channels.

All findings are presented for human review before any action is taken. → `principles/human-in-the-loop.md`

---

## What This Example Demonstrates

- **The same six-step reasoning model applies to agentic systems.** The steps are identical to the web application and concurrent payment reviews. What changes is the location of trust boundaries and the categories of evidence. → `specs/SPEC-005-agent-operational-behavior.md`
- **Injection surface mapping precedes hypothesis generation.** Before generating hypotheses, the reviewer lists every external data source the agent reads. Each is a potential trust boundary crossing. → `knowledge/prompt-injection.md`
- **Confirmed by absence of control applies to agentic architecture.** The absence of a confirmation step for email is evidence — not a gap in analysis. The absence of content filtering between web retrieval and reasoning is evidence. → `specs/SPEC-005-agent-operational-behavior.md`
- **Findings chain.** Web injection (H2) is serious but bounded. Web injection + unconfirmed email (H5) is critical. The blast radius of a finding depends on what other capabilities it can activate. Evaluate capability combinations, not individual capabilities in isolation.
- **Blast radius is the primary severity metric for agentic findings.** The question is not "how likely is injection?" — it is "what is the worst action the agent can take if injection succeeds?" Likelihood is secondary to blast radius when assessing agentic risk. → `knowledge/agent-trust-model.md`
- **`[REQUIRES_EXTERNAL_EVIDENCE]` applies to agent configuration too.** When knowledge base write controls are not visible, the finding is flagged as requiring evidence — not inflated to confirmed. The same epistemic discipline applies to agentic systems as to concurrent code. → `principles/evidence-over-assumptions.md`
- **The smallest safe change is always preferred.** Adding a confirmation step before email is smaller and more targeted than removing the email capability. Targeted changes reduce blast radius without reducing legitimate utility. → `principles/minimal-change.md`
