# What It Is

A trust boundary is a point where data or control moves between different levels of trust. On one side, the data is trusted within its context. On the other side, it is not — until it is verified.

Any data crossing a boundary must be treated as untrusted. The trust assigned in one context does not transfer automatically to another. Trust is not inherent; it is assigned based on context, and context changes at every boundary.

Systems often contain multiple implicit trust boundaries. Between a client and a server, between two internal services, between a process and its configuration, between a database and the application that reads from it. Identifying these boundaries is the first step in securing them.

# Why It Matters

Most security vulnerabilities occur when trust assumptions are incorrect. A system that assumes all internal data is safe, all authenticated users are authorized, or all network traffic is legitimate is a system built on invisible boundaries that attackers can exploit.

Data changes risk profile when it crosses boundaries. The same value that is safe inside a trusted process becomes untrusted the moment it enters a different process, service, or context. Ignoring this transition is ignoring the point where most failures occur.

Internal systems are not automatically trusted. A compromised service, a misconfigured integration, or a migrating workload can produce data that was not created by the system that receives it. Internal is not synonymous with safe.

Attackers exploit weak or invisible boundaries. Boundaries that are not identified are boundaries that are not defended. Every implicit trust assumption is a potential attack surface.

Security failures often happen at integration points, not inside isolated components. Components that are secure in isolation can fail when they accept data from another context without re-validating it. Boundaries define where validation, authentication, and verification must occur.

# Common Risks and Misconceptions

- **Assuming internal services are always trusted.** Internal boundaries are still boundaries. A service inside the network is inside the network, not inside the trust boundary.
- **Failing to identify implicit boundaries between components.** Boundaries that are not named are boundaries that are not defended.
- **Treating network location as a proxy for trust.** Being inside a VPN or a private subnet does not make data trustworthy. It makes the network location different, not the data.
- **Assuming authentication at one layer applies everywhere.** Authentication establishes identity at a boundary. It does not grant automatic trust at every subsequent boundary.
- **Not re-validating data after it crosses a system boundary.** Data that was valid in one context may be invalid or dangerous in another. Validation must happen at the boundary, not only at the origin.
- **Mixing trusted and untrusted data without distinction.** When trusted and untrusted data share the same processing path without separate handling, the entire flow inherits the risk of the untrusted input.

"Inside the system" does not mean "safe." A boundary exists wherever context changes, regardless of whether the crossing happens inside or outside a network perimeter.

# Relationship to Other Concepts

- **Input Validation:** Input Validation is the enforcement mechanism at trust boundaries. It defines what data is acceptable after it crosses from untrusted to trusted context.
- **Secure Serialization:** Every deserialization point is a trust boundary. Data that crosses it must be treated as untrusted, regardless of where it originated.
- **Secrets Management:** Secrets cross trust boundaries when they move between services, environments, and storage systems. Each crossing requires protection.
- **Least Privilege:** Least Privilege minimizes the trust assigned to any identity. Trust boundaries define where that trust is validated.
- **Defense in Depth:** Defense in Depth assumes trust boundaries will be crossed and that controls will fail. Multiple independent layers ensure that crossing one boundary does not expose everything behind it.

# Success Criteria

Trust Boundaries are working when:

- Every external input is treated as untrusted until verified.
- Boundaries are explicitly identified in system design, not discovered after an incident.
- Validation occurs at every boundary crossing, not only at the origin.
- Security controls are placed at interfaces, not only inside isolated components.
- Systems assume failure at boundaries and design accordingly.