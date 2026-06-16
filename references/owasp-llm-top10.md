# OWASP Top 10 for LLM Applications

Curated reference for the OWASP Top 10 risks specific to Large Language Model applications and agentic systems. Load when reviewing an agent, LLM-powered application, or multi-agent architecture.

**Source:** [https://owasp.org/www-project-top-10-for-large-language-model-applications/](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

This list is distinct from the classic OWASP Top 10 for web applications. It addresses risks that emerge specifically from the properties of language models: the blurred boundary between instruction and data, the autonomy of agentic systems, and the non-deterministic nature of model outputs.

---

## LLM01: Prompt Injection

An attacker manipulates LLM inputs through crafted prompts, causing the model to execute unintended actions or produce outputs that override its original instructions.

**Two forms:** Direct (user injects into their own prompt) and indirect (attacker plants instructions in external data the agent retrieves — documents, web pages, tool outputs).

**Skill mapping:** → `knowledge/prompt-injection.md`, `checklists/agentic-system-review.md` (sections 1 and 2)

**Why it is LLM01:** It is the root cause that enables most other risks on this list. Successful injection can chain into unauthorized actions (LLM02), data exfiltration (LLM06), and supply chain attacks (LLM05).

---

## LLM02: Insecure Output Handling

LLM output is passed downstream to other systems (browsers, code interpreters, other agents, databases) without sufficient validation, enabling XSS, SSRF, privilege escalation, or remote code execution.

**Example:** An agent generates HTML that is rendered in a browser without escaping — the HTML contains a script tag injected by a malicious document the agent processed.

**Skill mapping:** → `knowledge/secure-serialization.md`, `knowledge/trust-boundaries.md`, `checklists/agentic-system-review.md` (section 7)

---

## LLM03: Training Data Poisoning

Training data is manipulated to introduce vulnerabilities, backdoors, or biases into the model. The model behaves normally in most conditions but produces attacker-controlled outputs when specific triggers are present.

**Skill mapping:** → `knowledge/supply-chain-security.md` — the model is a dependency. Its training pipeline is part of the supply chain.

**Note:** This risk applies at training time, not inference time. For most users deploying third-party models, the relevant defense is verifying model provenance and monitoring for unexpected behavior drift.

---

## LLM04: Model Denial of Service

Attackers craft inputs that consume excessive resources, degrade performance, or drive up inference costs — making the service unavailable or economically unsustainable.

**Examples:** Inputs designed to generate maximum token output, recursive tool calls that amplify compute usage, prompts that trigger repeated retry loops.

**Skill mapping:** → `knowledge/error-handling-secure-failure.md` (failure modes), `principles/control-sufficiency.md` (rate limiting sufficiency under adversarial load)

---

## LLM05: Supply Chain Vulnerabilities

Risks from third-party model providers, fine-tuning datasets, plugins, tool integrations, and deployment infrastructure. A compromised model provider, dataset, or plugin can introduce vulnerabilities that no application-level control can prevent.

**Examples:** A malicious fine-tuning dataset that introduces a backdoor. A compromised plugin that exfiltrates data. A third-party model update that changes behavior without notification.

**Skill mapping:** → `knowledge/supply-chain-security.md` — extends directly to LLM components.

---

## LLM06: Sensitive Information Disclosure

The LLM reveals sensitive data — training data, system prompt contents, user data from prior sessions, internal tool definitions, or credentials — either through direct prompting, inference attacks, or injection-triggered exfiltration.

**Skill mapping:** → `knowledge/secrets-management.md`, `knowledge/error-handling-secure-failure.md`, `checklists/agentic-system-review.md` (section 8)

**Key concern:** The system prompt often contains credentials, internal architecture details, and behavioral constraints. Its contents should be treated as sensitive and never retrievable through user interaction.

---

## LLM07: Insecure Plugin Design

LLM plugins and tools are designed without sufficient access controls, input validation, or output sanitization. Because the LLM constructs tool calls dynamically, an injected instruction can craft a tool call that the LLM would not generate under normal conditions.

**Examples:** A plugin that executes arbitrary SQL because the LLM constructs the query. A file-access plugin with no path restriction that the LLM calls with `../../etc/passwd` after injection.

**Skill mapping:** → `knowledge/least-privilege.md`, `knowledge/input-validation.md`, `checklists/agentic-system-review.md` (section 3)

---

## LLM08: Excessive Agency

The LLM is granted more permissions, tool access, or autonomy than its task requires. When combined with prompt injection or model error, excessive agency amplifies the blast radius of any failure.

**Three dimensions of excess:** Excessive functionality (tools the agent doesn't need), excessive permissions (tools scoped broader than required), excessive autonomy (actions taken without human confirmation).

**Skill mapping:** → `knowledge/least-privilege.md`, `knowledge/agent-trust-model.md`, `checklists/agentic-system-review.md` (sections 3 and 4)

**This is the most directly controllable risk.** Removing a tool or adding a confirmation step is a deterministic defense — it reduces blast radius regardless of whether injection is possible.

---

## LLM09: Overreliance

Systems or users trust LLM outputs without sufficient verification, leading to decisions based on hallucinated facts, fabricated citations, or confident-sounding incorrect analysis.

**Skill mapping:** → `principles/evidence-over-assumptions.md` — the principle applies to the agent's outputs as much as to its analysis. An agent that produces findings should distinguish what it observed from what it inferred.

**Security-specific concern:** An agent used for security review that hallucinates findings creates false negatives (real vulnerabilities missed) or false positives (nonexistent vulnerabilities reported). Both erode trust in the review process.

---

## LLM10: Model Theft

Adversaries extract model weights, architecture, or training data through repeated probing, prompt reconstruction attacks, or access to model serving infrastructure.

**Skill mapping:** → `knowledge/trust-boundaries.md` (inference endpoints as trust boundaries), `knowledge/least-privilege.md` (access controls on model serving)

**Note:** This risk is primarily relevant to organizations that train or fine-tune proprietary models. For users of third-party APIs, the relevant defense is access control on the API keys and monitoring of usage patterns.

---

## Using This Reference with the Skill

When reviewing an agentic system, apply the OWASP LLM Top 10 alongside the checklist as a classification layer:

1. Use `checklists/agentic-system-review.md` to identify findings.
2. Map each finding to its LLM Top 10 category for classification and communication.
3. Use the skill mappings above to load the relevant knowledge files for deeper analysis.

The OWASP categories are useful for communicating findings to stakeholders who know the LLM Top 10. The skill's knowledge files provide the reasoning depth to evaluate and explain each finding. Both serve different purposes — classification versus understanding.
