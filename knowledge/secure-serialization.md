# What It Is

Serialization converts objects into a storable or transmittable format. Deserialization reconstructs the original object from that format. Together they allow data to travel across processes, networks, and time.

Unsafe deserialization can execute arbitrary code or manipulate data in ways the system never intended. When a deserialization format allows object reconstruction with behavior — not just data — the recreated object can carry executable logic that runs during reconstruction.

Serialization formats must be treated as untrusted input when they come from external sources. A serialized stream is a contract between the sender and the receiver. If the sender is untrusted, the contract is unreliable.

# Why It Matters

Deserialization can execute arbitrary code if unsafe formats are used. This is not a theoretical risk. Formats that embed object types and methods in their serialized output allow an attacker to construct payloads that execute code during reconstruction. Remote code execution through deserialization has been one of the most consistently exploited vulnerability classes.

External data streams may be tampered with before being processed. A serialized object that passes through an untrusted channel — a network, a queue, a file system — can be modified, replaced, or injected.

Trusting serialized objects assumes the data has not been modified. In most real systems, serialized data crosses trust boundaries where modification is possible.

Serialization boundaries are trust boundaries. Every point where data is deserialized is a point where untrusted input enters the system. The same discipline applied to user input must be applied to serialized data.

Reconstruction is an active operation, not a passive one. Deserialization does not just read data — it rebuilds state, and that process can execute logic.

# Common Risks and Misconceptions

- **Using unsafe serialization formats that allow object reconstruction with behavior.** Formats that embed executable logic in serialized data turn deserialization into an execution vector. Data-only formats avoid this class of risk entirely.
- **Assuming internal systems always produce safe serialized data.** Internal boundaries are still boundaries. A compromised or misbehaving service can produce malicious payloads that another service trusts.
- **Deserializing external input without validation.** Deserialization without validation is input processing without input validation. The risks are the same.
- **Mixing data formats with executable object graphs.** When a serialized format carries both data and behavior, reconstruction becomes unpredictable. Data and logic should travel separately.
- **Treating serialization as a neutral technical step rather than a security boundary.** Serialization is a trust boundary. Data crosses it. Every crossing deserves scrutiny.
- **Forgetting that serialized data may persist outside the system and be reused later in a malicious context.** Cached data, stored sessions, and persisted objects may be modified at rest and deserialized in a different security context than the one that created them.

Serialization is not just data transformation. It is a potential execution vector. Treating it as a neutral technical step ignores the most dangerous property of unsafe deserialization: it can turn data into code.

# Relationship to Other Concepts

- **Input Validation:** Input Validation defines what data is acceptable. Secure Serialization defines how serialized data is reconstructed. Both address trust boundaries — one for raw input, one for structured input.
- **Secrets Management:** Serialized objects may contain credentials. Secrets Management ensures those credentials are protected. Deserializing a serialized secret is as dangerous as storing it in plaintext.
- **Trust Boundaries:** Every deserialization point is a trust boundary. Data that crosses it must be treated as untrusted, regardless of where it originated.
- **Supply Chain Security:** Serialized data in dependencies, caches, and message queues travels through the supply chain. Tampering at any point compromises every consumer.

# Success Criteria

Secure Serialization is working when:

- Only safe, data-only formats are used for external input. Formats that reconstruct executable objects are reserved for trusted, internal contexts.
- Serialized input is treated as untrusted by default. Trust is earned, not assumed.
- Deserialization does not trigger code execution paths. Reconstruction produces data, not behavior.
- Data integrity is verified before reconstruction. Tampering is detected before it reaches the deserializer.
- Systems avoid reconstructing complex object graphs from external sources. Simpler structures reduce the attack surface.