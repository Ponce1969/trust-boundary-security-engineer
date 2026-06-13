# What It Is

Supply Chain Security is the discipline of protecting systems from risks introduced through dependencies, integration points, and external components. Every system that relies on external packages, libraries, services, or pipelines inherits the security posture of those dependencies.

A supply chain attack does not exploit the system itself — it exploits something the system trusts. A compromised dependency, a tampered CI pipeline, or a malicious package update can introduce vulnerabilities that no amount of internal hardening will prevent.

Supply Chain Security examines the full path from source to deployment: where components come from, who maintains them, how they are verified, and what happens when they are compromised.

# Why It Matters

Dependencies dominate modern software. Most systems depend on far more external code than internal code. Each dependency is a trust decision — and most trust decisions are made implicitly, without verification.

Supply chain attacks exploit trust, not vulnerabilities in the system itself. An attacker who compromises a widely used library gains access to every system that depends on it, without attacking any of them directly.

The blast radius of a supply chain compromise is disproportionately large. A single compromised dependency can affect thousands of downstream systems simultaneously, many of which have no direct control over the compromised component.

Most supply chain risks are invisible. Dependencies are often added transitively, updated automatically, and trusted without verification. The system that appears self-contained may depend on hundreds of packages it never explicitly chose.

# Common Risks and Misconceptions

- **Trusting dependencies without verification.** Adding a package without reviewing its maintainers, release history, or security posture is an implicit trust decision.
- **Assuming transitive dependencies are safe because direct dependencies are.** A dependency's dependencies expand the attack surface beyond what was explicitly chosen.
- **Relying on automated updates without integrity verification.** Automatic updates improve currency but also increase the attack surface if updates are not verified.
- **Treating CI/CD pipelines as trusted infrastructure.** Build and deployment pipelines are part of the supply chain. A compromised pipeline can inject malicious code that appears legitimate.
- **Assuming that because a dependency is widely used, it is well maintained.** Popularity is not a proxy for security. Abandoned or poorly maintained packages receive widespread trust without recent security review.
- **Ignoring the supply chain for internal tools and scripts.** Internal tools often depend on the same external packages as production code, with fewer controls and less scrutiny.

# Relationship to Other Concepts

- **Trust Boundaries:** Every dependency, pipeline, and integration point is a trust boundary. Supply Chain Security is the application of trust boundary discipline to the external components the system relies on.
- **Secrets Management:** Secrets travel through the supply chain in CI pipelines, deployment configurations, and dependency credentials. A secret exposed at any point in the chain compromises every system that point serves.
- **Secure by Default:** Secure by Default ensures the starting state is safe. Supply Chain Security ensures that the components the system depends on do not undermine that starting state.
- **Defense in Depth:** Supply chain attacks bypass application-level controls. Defense in Depth ensures that a compromised dependency does not expose the entire system.

# Success Criteria

Supply Chain Security is working when:

- Every direct dependency is chosen intentionally, not added implicitly.
- Dependencies are verified before integration, not trusted by default.
- Transitive dependencies are visible and monitored, not hidden.
- CI/CD pipelines are treated as part of the supply chain and secured accordingly.
- A compromised dependency is detectable and its blast radius is contained.