# What It Is

Threat Modeling is a structured method for identifying, evaluating, and addressing security risks in a system before they are exploited. It examines what can go wrong, where it can go wrong, and how to address it — before code is written, before configurations are deployed, and before systems are exposed.

Threat modeling is not a one-time activity. It is a practice that should be repeated when the system changes, when new trust boundaries are introduced, and when the threat landscape evolves.

The core question threat modeling answers is: given what this system does, who might want to compromise it, and how would they try?

# Why It Matters

Security issues found early are cheaper and easier to fix than issues found in production. Threat modeling moves security analysis to the beginning of the process, where corrections cost less and have less impact on existing functionality.

Threat modeling makes trust boundaries explicit. Systems that appear simple often have implicit boundaries between components, services, and data flows that are not visible until they are modeled. Identifying these boundaries before an attack is more effective than discovering them during one.

Threat modeling prioritizes risks. Not every threat requires the same response. A structured approach distinguishes critical risks from minor concerns and directs effort where it matters most.

Without threat modeling, security decisions are reactive. Teams respond to vulnerabilities after they are reported, after they are exploited, or after a compliance audit flags them. Threat modeling enables proactive security — identifying and addressing risks before they become incidents.

# Common Risks and Misconceptions

- **Treating threat modeling as a compliance checkbox.** Filling out a template without genuine analysis produces paperwork, not security. The value is in the thinking, not the document.
- **Modeling only external attackers.** Internal threats, misconfigurations, and accidental data exposure are as significant as external adversaries. Threat modeling that ignores insiders and mistakes is incomplete.
- **Attempting to enumerate every possible threat.** Exhaustive threat lists are impractical and demotivating. Threat modeling should prioritize by impact and likelihood, not by completeness.
- **Performing threat modeling only once.** Systems change. Threat models must be updated when architecture, data flows, or trust boundaries change.
- **Confusing threat modeling with vulnerability scanning.** Vulnerability scanning finds known weaknesses in existing code. Threat modeling identifies potential risks in system design before they become code.
- **Focusing on individual components instead of data flows.** Threats often emerge at the boundaries between components, not inside them. A component-by-component review misses cross-boundary risks.

# Relationship to Other Concepts

- **Trust Boundaries:** Threat Modeling identifies where trust boundaries exist and what happens when they are crossed. Trust Boundaries define the surfaces that Threat Modeling examines.
- **Input Validation:** Threat Modeling reveals where external input enters the system and what validation is required at each entry point.
- **Least Privilege:** Threat Modeling identifies what each component needs access to and where excessive privileges create risk. Least Privilege constrains those risk surfaces.
- **Defense in Depth:** Threat Modeling identifies the threats. Defense in Depth ensures that a single failed control does not expose the entire system.
- **Secure by Default:** Threat Modeling reveals what defaults are safe and which configurations create exposure. Secure by Default ensures the starting state minimizes the threats that modeling identifies.

# Success Criteria

Threat Modeling is working when:

- Trust boundaries and data flows are explicitly identified before implementation begins.
- Threats are prioritized by impact and likelihood, not by completeness.
- Security decisions are proactive, not reactive.
- The threat model is updated when the system changes.
- Identified threats map to specific mitigations, not to vague resolutions.