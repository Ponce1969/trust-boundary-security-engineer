# What It Is

Secure by Default means systems should start in the safest reasonable state. Unsafe behavior should require explicit decisions, not the other way around.

When a system is secure by default, the path of least resistance is the safe path. Users who take no action are still protected. Unsafe configurations exist, but they require deliberate choice — not a setting changed, but a decision made.

Defaults influence long-term habits. When the default is safe, the majority stays safe. When the default is insecure, the majority stays insecure — not because they chose risk, but because they never chose at all.

# Why It Matters

People often keep default settings. This is not laziness; it is human behavior. Defaults are powerful precisely because they require no action to maintain.

Safe defaults prevent accidental exposure. Open ports, permissive access controls, and unencrypted connections are rarely intentional — they are what happens when security requires opt-in instead of opt-out.

Security should not depend on perfect users. A system that is only secure when configured correctly by an informed administrator is a system that will be insecure in practice. Secure defaults acknowledge reality: people forget, people miss documentation, and people do not know what they do not know.

# Common Risks and Misconceptions

- **Assuming users will configure security later.** They will not. What ships is what stays.
- **Treating convenience as more important than safety.** Convenient insecure defaults create convenient insecure systems.
- **Believing documentation alone compensates for insecure defaults.** Documentation helps informed users. It does not protect uninformed ones.
- **Expecting users to know what they do not know.** A user who does not realize a setting matters will never change it.
- **Equating "configurable" with "secure."** A secure option that is not the default is a secure option most people will never use.

# Relationship to Other Concepts

- **Least Privilege:** Secure by Default and Least Privilege overlap. Least Privilege restricts what a system can do; Secure by Default restricts what it does by default. Together they reduce both capability and likelihood of misuse.
- **Defense in Depth:** Secure by Default is the first layer. If one default fails, Defense in Depth ensures other layers still protect the system.
- **Human in the Loop:** Secure by Default reduces the number of decisions a human must make to stay safe. Human in the Loop ensures the decisions that remain are reviewed by a person.

# Success Criteria

Secure by Default is working when:

- Safe behavior exists without additional configuration.
- Risky actions require deliberate choices, not just missing settings.
- Users remain protected even when they forget optional security settings.
- The default state is the one safest for the majority of use cases.
- Security does not rely on users reading documentation before they are protected.