# Threat Modeling Review

Ordered checklist for conducting a structured threat model on a system or component.

## Context

Define the scope before modeling:

- What system, service, or component is being modeled.
- What data flows through the system and where it goes.
- What trust boundaries the system crosses. → `knowledge/trust-boundaries.md`
- What the system's intended use and operating environment are.

## 1. Identify Trust Boundaries

- [ ] Map every point where data or control crosses between different levels of trust. → `knowledge/trust-boundaries.md`
- [ ] Distinguish external boundaries (user-facing, third-party) from internal boundaries (between services, between processing stages).
- [ ] Identify implicit boundaries that are not documented in architecture — shared databases, message queues, configuration stores.
- [ ] Verify that every boundary is named and its crossing rules are explicit.

## 2. Map Data Flows

- [ ] Trace every path that data follows through the system, from entry to storage to exit.
- [ ] Identify where data enters the system and what validation exists at each entry point. → `knowledge/input-validation.md`
- [ ] Identify where data is stored, cached, or persisted and what protection exists at rest.
- [ ] Identify where data leaves the system and what sanitization or filtering exists at exit points.

## 3. Identify Threats

- [ ] For each trust boundary crossing, ask: what can go wrong if this boundary is crossed with unexpected or malicious data?
- [ ] For each data flow, ask: what happens if this data is tampered with, intercepted, or blocked?
- [ ] For each privilege level, ask: what can an actor with this access do that they should not? → `knowledge/least-privilege.md`
- [ ] Consider both external attackers and internal misuse, misconfiguration, and accidental data exposure.
- [ ] Consider threats to availability, integrity, and confidentiality — not only one category. → `knowledge/threat-modeling.md`
- [ ] Apply STRIDE per trust boundary to generate a mandatory hypothesis set: → `references/threat-modeling-frameworks.md`
  - **Spoofing** — can an actor falsely claim an identity to cross this boundary?
  - **Tampering** — can data crossing this boundary be modified without detection?
  - **Repudiation** — can an actor deny having crossed this boundary, and would the system be unable to prove they did?
  - **Information Disclosure** — can data crossing this boundary be observed by an unauthorized actor?
  - **Denial of Service** — can this boundary crossing be abused to degrade availability for legitimate actors?
  - **Elevation of Privilege** — can crossing this boundary grant more access than the actor is entitled to?

## 4. Evaluate Impact and Likelihood

- [ ] Assess the impact of each identified threat if it were realized — what systems, data, or users would be affected.
- [ ] Assess the likelihood of each threat based on the current architecture, not theoretical possibility. → `principles/evidence-over-assumptions.md`
- [ ] Prioritize threats by real impact in the specific context, not generic severity scores. → `principles/context-matters.md`
- [ ] Mark threats where evidence is insufficient as uncertain — do not inflate severity without evidence.

## 5. Identify Mitigations

- [ ] For each prioritized threat, identify existing controls that already mitigate it. → `knowledge/defense-in-depth.md`
- [ ] For threats without adequate mitigation, identify the smallest safe change that addresses the risk. → `principles/minimal-change.md`
- [ ] Verify that proposed mitigations do not introduce new trust boundaries or privilege escalations. → `knowledge/least-privilege.md`
- [ ] Confirm that mitigations are proportional to the risk — low-impact threats do not justify high-impact changes.

## 6. Validate the Model

- [ ] Review the model with someone who was not involved in creating it. → `principles/human-in-the-loop.md`
- [ ] Verify that no implicit trust assumptions remain unchallenged.
- [ ] Check that the model covers all data flows, not only the obvious ones.
- [ ] Confirm that the model will be updated when the system changes. → `knowledge/threat-modeling.md`

## 7. Reporting

- [ ] Document each threat as an observation (verified), inference (supported but uncertain), or risk (conditional assessment). → `principles/evidence-over-assumptions.md`
- [ ] Prioritize findings by context-specific impact, not by category count. → `principles/education-over-fear.md`
- [ ] Present the threat model for human review before any mitigation action is taken. → `principles/human-in-the-loop.md`
- [ ] Ensure the threat model is accessible and understandable to stakeholders, not only security specialists. → `principles/education-over-fear.md`