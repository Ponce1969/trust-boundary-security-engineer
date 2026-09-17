# Purpose

A security reasoning core must remain stable and technology-agnostic while enabling specialized domains (such as AI agents, multi-agent frameworks, cloud infrastructure, or specific protocols) to build upon its reasoning without fragmenting the methodology. This specification defines the extension contract for Trust Boundary.

# Architecture & Dependency Hierarchy

The extension architecture strictly follows a unidirectional dependency hierarchy:

```text
       Trust Boundary Core (security-engineer-skill)
         [Stable / Technology-Agnostic Reasoning Core]
                            │
                            ▼
          Extension Protocol (SPEC-006)
         [Precedence, Compatibility & Contracts]
                            │
                            ▼
      Specialized Auditors (e.g., agentic-security-auditor)
         [Domain-Specific Specializations & Evidence Collectors]
```

## Core Independence Principle
Trust Boundary Core has zero operational dependency on specialized auditor skills. The Core is completely self-contained and must remain fully capable of conducting general software security audits (data flows, concurrency, transaction isolation, authentication, authorization, secrets) whether extensions exist in the environment or not.

## Extension Dependency Principle
Specialized skills are domain adapters that extend the Core. They explicitly declare their conceptual dependency on Trust Boundary Core, inheriting its reasoning engine (`SPEC-005`), axioms, and finding structure.

# Precedence & Non-Contradiction Rules

1. **Rule of Axiom Preservation:** A specialized extension may specialize, contextualize, and expand the Core, but it **MUST NOT contradict or weaken** the Core's foundational axioms:
   - *Evidence over Assumptions*
   - *Control Sufficiency*
   - *Execution Model Classification*
   - *Spatial and Temporal Trust Boundaries*
   - *State Evolution Tracing*
   - *Structural vs. Time-Dependent Conclusions*
   - *Proportionality and Education over Fear*
   - *Human in the Loop*
   - *Minimal Change*
2. **Rule of Explicit Core Evolution:** If an emerging domain genuinely requires altering an underlying reasoning axiom, that change must be proposed, debated, and formally integrated into the Core specifications (`SPEC-001` through `SPEC-005`). An extension cannot unilaterally redefine core axioms within its local scope.
3. **Rule of Evidence Model Integrity:** An extension must consume and produce evidence classifications conforming to SPEC-005 (`[CONFIRMED]`, `[CONFIRMED_BY_ABSENCE_OF_CONTROL]`, `[CONFIRMED_BY_INSUFFICIENT_CONTROL]`, `[REQUIRES_EXTERNAL_EVIDENCE]`). It may not invent alternative truth states that bypass empirical verification.

# Domain Specialization Contracts

Specialized skills implement three architectural responsibilities:

## 1. Boundary Specialization
Translates abstract spatial and temporal trust boundaries into concrete domain entities:
- In agentic systems: Operator context vs. User input vs. Environment data; Agent-to-Agent message channels; Tool/MCP capability boundaries; Vector/Episodic memory stores.
- In cloud systems: VPC peering, IAM roles, service accounts, control planes.

## 2. Sufficiency Instantiation
Instantiates the generic *Control Sufficiency Principle* (`principles/control-sufficiency.md`) into domain-appropriate containment layers. For example, the agentic domain instantiates sufficiency across three distinct layers:
- Layer 1: Semantic (Prompt / Guardrail)
- Layer 2: Architecture / Sandbox (Physical isolation, readonly mounts, network egress deny)
- Layer 3: API / Tool (Least privilege, ephemeral credentials, strict typing)

## 3. Post-Reasoning Taxonomy Mapping
Specialized domains may utilize industry-standard threat frameworks (such as MAESTRO 7-layer model for AI, or STRIDE for general systems) exclusively as a **post-reasoning taxonomy and classification layer**.
The reasoning workflow must strictly follow:

```text
Observable Evidence
      │
      ▼
Trust Boundary Reasoning (SPEC-005: Boundaries, Execution Model, State Evolution)
      │
      ▼
Threat / Risk Identification (Sufficiency Evaluation, Structural vs Temporal)
      │
      ▼
Domain Taxonomy Mapping (e.g. MAESTRO layer or OWASP classification)
```

Taxonomies must never replace the empirical reasoning loop, nor be used as blind automated checklists.

# Evidence Collectors Contract

Extensions may provide automated scripts or collectors to assist human or AI auditors. All evidence collectors must adhere to the following contract:

1. **Read-Only / Non-Destructive:** Collectors operate strictly under `READ -> ANALYZE -> REPORT`. They never modify files, restart services, alter firewall rules, or mutate running infrastructure.
2. **Observation-First (No Autonomous Verdicts):** Collectors output raw, verifiable technical observations and structured telemetry. They **MUST NEVER** output autonomous subjective verdicts such as `SECURE`, `SAFE`, or `VULNERABLE` based solely on heuristics.
3. **Categorized Uncertainty:** Any heuristic matching (e.g. secret scanners) must explicitly categorize findings:
   - `[PATTERN_MATCH]`: Text matches known regex structure but context is unverified.
   - `[HEURISTIC_CANDIDATE]`: High entropy or suspicious context requiring external confirmation.
   - `[CONFIRMED]`: Directly verified active credential or proven bypass with definitive evidence.
