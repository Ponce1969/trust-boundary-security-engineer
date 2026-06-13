# What It Is

Least Privilege means systems, users, and processes should have only the permissions they actually need — nothing more.

More permissions create more opportunities for misuse. Every permission granted beyond what is necessary expands the surface where a mistake or a compromise can cause damage. Privileges should be intentional, not accidental.

Least Privilege is not about distrust. It is about acknowledging that even trusted users and systems make mistakes, and that the impact of those mistakes should be as small as possible.

# Why It Matters

Mistakes happen. A developer accidentally deletes a production database because their credentials allowed it. A service exposes sensitive data because its account had access to more than the task required. These are not hypothetical scenarios — they are recurring patterns in real incidents.

Compromises happen. When an attacker gains access, the first thing they encounter is the privilege level of whatever they compromised. If that level is already minimal, the blast radius is small. If it is broad, the attacker starts with excessive power.

Limiting privileges limits impact. It turns catastrophic failures into contained ones.

Excessive permissions often remain unnoticed until something goes wrong. Permissions granted for convenience or during development persist in production, forgotten and unreviewed, waiting to amplify the next mistake.

# Common Risks and Misconceptions

- **Granting broad permissions "just in case."** Convenience today becomes risk tomorrow.
- **Treating administrator access as normal.** Routine work should not require administrative privileges.
- **Assuming trusted users cannot make mistakes.** Trust is about intent, not about impact. A trusted user with excessive permissions can cause the same damage as a malicious one.
- **Forgetting to remove permissions that are no longer needed.** Temporary access that becomes permanent is one of the most common privilege accumulation patterns.
- **Equating convenience with necessity.** If a task can be completed with fewer permissions, the extra ones are convenience, not necessity.

# Relationship to Other Concepts

- **Secure by Default:** Secure by Default ensures systems start safe. Least Privilege ensures they stay safe by restricting what they can do. Together they narrow both the starting state and the reachable states.
- **Defense in Depth:** Least Privilege is a layer within Defense in Depth. If one control fails, restricted privileges limit what an attacker can reach.
- **Human in the Loop:** Least Privilege reduces the decisions a human must make about access. Human in the Loop ensures the access decisions that remain are reviewed by a person.

# Success Criteria

Least Privilege is working when:

- Permissions are intentionally limited, not just minimally documented.
- Access reflects actual responsibilities, not organizational seniority.
- Compromises have a smaller blast radius because the compromised principal had limited reach.
- Unused permissions are removed, not just left in place.
- Temporary access does not silently become permanent access.