# Threat Modeling Frameworks

Curated references for structured threat modeling methodologies. Load when conducting a threat model, evaluating architecture security, or selecting a methodology appropriate to the system under review.

---

## STRIDE

**Source:** [STRIDE Threat Model — Microsoft](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats)

The most widely used threat categorization framework in software development. Developed at Microsoft. Each letter represents a threat category:

| Letter | Threat | Violated Property | Skill Mapping |
|---|---|---|---|
| S | Spoofing | Authentication | `knowledge/trust-boundaries.md` |
| T | Tampering | Integrity | `knowledge/input-validation.md` |
| R | Repudiation | Non-repudiation | `knowledge/error-handling-secure-failure.md` |
| I | Information Disclosure | Confidentiality | `knowledge/secrets-management.md` |
| D | Denial of Service | Availability | `examples/concurrent-payment-review.md` (retry storms) |
| E | Elevation of Privilege | Authorization | `knowledge/least-privilege.md` |

STRIDE is applied per trust boundary. For each boundary identified in the context construction model, the agent can ask: which STRIDE categories apply to this crossing? → `specs/SPEC-005-agent-operational-behavior.md`

**How to use STRIDE with this skill:** After identifying trust boundaries (Step 2 of the reasoning model), apply STRIDE to each boundary to generate a mandatory set of hypotheses. A boundary that crosses from unauthenticated to authenticated context mandates Spoofing and Elevation of Privilege hypotheses. A boundary that crosses to persistent storage mandates Tampering and Information Disclosure hypotheses.

---

## PASTA (Process for Attack Simulation and Threat Analysis)

**Source:** [PASTA Threat Modeling](https://owasp.org/www-pdf-archive/AppSecEU2012_PASTA.pdf)

A risk-centric methodology that aligns threat analysis with business objectives. Seven stages: define objectives → define technical scope → application decomposition → threat analysis → vulnerability analysis → attack modeling → risk impact analysis.

Relevant to this skill when: the team needs to connect technical findings to business risk, or when stakeholders require a formal risk register rather than a developer-focused findings list.

PASTA is more heavyweight than STRIDE but produces output that executives and compliance teams can consume directly.

---

## LINDDUN

**Source:** [LINDDUN Privacy Threat Modeling](https://www.linddun.org/)

A threat modeling methodology specifically focused on privacy threats. The threat categories are: Linkability, Identifiability, Non-repudiation, Detectability, Disclosure of information, Unawareness, Non-compliance.

Relevant to this skill when: the system processes personal data, health information, or any data subject to GDPR, HIPAA, or similar regulations. STRIDE covers security; LINDDUN covers privacy. The two are complementary, not competing.

---

## Attack Trees

**Source:** [Attack Trees — Bruce Schneier](https://www.schneier.com/academic/archives/1999/12/attack_trees.html) (1999, foundational)

A formalism for representing how a system can be attacked. The root is the attacker's goal; children are sub-goals or methods. Leaf nodes are atomic attack steps.

Relevant to this skill when: a complex attack path involves multiple steps, or when the team needs to evaluate the combined probability of a multi-stage attack. Attack trees complement STRIDE: STRIDE identifies what can go wrong; attack trees model how an attacker would chain steps to achieve it.

---

## MITRE ATT&CK

**Source:** [https://attack.mitre.org/](https://attack.mitre.org/)

A knowledge base of adversary tactics and techniques based on real-world observations. Organized by tactic (the goal) and technique (the method). Includes sub-techniques, mitigations, and detection strategies.

Relevant to this skill when: analyzing a system that is a likely target of sophisticated attackers, correlating findings with known attack patterns, or designing detection strategies.

ATT&CK is post-exploitation focused — it describes what attackers do after initial access. Combine with STRIDE for pre-exploitation design review and ATT&CK for post-exploitation modeling.

**Key tactic categories relevant to this skill:**

- Initial Access — trust boundary crossings, phishing, supply chain compromise
- Credential Access — `knowledge/secrets-management.md`
- Lateral Movement — internal trust boundaries, least privilege gaps
- Exfiltration — data flows out of the system, logging and monitoring
- Impact — availability attacks, data destruction, ransomware

---

## NIST SP 800-30: Guide for Conducting Risk Assessments

**Source:** [https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-30r1.pdf](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-30r1.pdf)

The formal NIST risk assessment methodology. Defines threat sources, threat events, vulnerabilities, likelihood, and impact in a structured process. Used as the baseline for US federal security requirements.

Relevant to this skill when: the system operates in a regulated environment, the team needs a formal risk register, or findings need to be communicated in terms of NIST risk levels (Very High / High / Moderate / Low / Very Low).
