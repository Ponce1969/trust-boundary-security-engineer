# Cloud and Infrastructure Security

Curated references for cloud security, infrastructure-as-code security, and container security. Load when reviewing cloud configurations, IAC templates, CI/CD pipelines, or container workloads.

---

## CSA Cloud Controls Matrix (CCM)

**Source:** [https://cloudsecurityalliance.org/research/cloud-controls-matrix/](https://cloudsecurityalliance.org/research/cloud-controls-matrix/)

A cybersecurity control framework for cloud computing. Maps controls to major compliance standards (ISO 27001, PCI-DSS, HIPAA, SOC 2). Covers 197 control objectives across 17 domains.

Relevant to this skill when: reviewing cloud architecture for compliance coverage, mapping findings to a control framework, or communicating findings to compliance teams.

---

## AWS Security Best Practices

**Source:** [https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)

The AWS Well-Architected Framework Security Pillar. Covers identity and access management, detection, infrastructure protection, data protection, and incident response. Includes specific guidance for IAM policies, S3 bucket policies, VPC configuration, and KMS key management.

Relevant to this skill when: reviewing AWS infrastructure, IAM policies, or cloud architecture. Maps directly to `knowledge/least-privilege.md` (IAM), `knowledge/secrets-management.md` (KMS, Secrets Manager), and `knowledge/defense-in-depth.md` (layered controls).

---

## OWASP Infrastructure as Code Security

**Source:** [https://owasp.org/www-project-devsecops-guideline/latest/02b-Hardening-IaC](https://owasp.org/www-project-devsecops-guideline/latest/02b-Hardening-IaC)

Guidance on security risks specific to infrastructure-as-code tooling (Terraform, CloudFormation, Ansible, Pulumi). Covers misconfigurations, secrets in state files, privilege escalation via IAC roles, and drift between declared and actual state.

Relevant to this skill when: reviewing Terraform, CloudFormation, or similar IaC files. The IAC/Cloud Misconfiguration example in this repository demonstrates the application of these principles. → `examples/iac-cloud-misconfiguration-review.md`

---

## CIS Benchmarks

**Source:** [https://www.cisecurity.org/cis-benchmarks](https://www.cisecurity.org/cis-benchmarks)

Consensus-based configuration guidelines for operating systems, cloud providers, containers, and network devices. Each benchmark provides specific, testable configuration recommendations at two levels: Level 1 (basic hygiene) and Level 2 (defense in depth).

Relevant to this skill when: evaluating whether a cloud or container configuration meets a baseline security standard. CIS Benchmarks translate `knowledge/secure-by-default.md` into specific, verifiable configuration states.

**Commonly used benchmarks:**
- CIS AWS Foundations Benchmark
- CIS Kubernetes Benchmark
- CIS Docker Benchmark
- CIS Google Cloud Platform Foundation Benchmark

---

## NIST SP 800-190: Application Container Security Guide

**Source:** [https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-190.pdf](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-190.pdf)

NIST guidance specifically for container security. Covers image vulnerabilities, configuration defects, build pipeline threats, container runtime threats, and orchestration-level risks.

Relevant to this skill when: reviewing Docker images, Kubernetes configurations, or container registries. Connects to `knowledge/supply-chain-security.md` (base image risks), `knowledge/least-privilege.md` (container capabilities), and `knowledge/trust-boundaries.md` (network policies).

---

## MITRE ATT&CK for Cloud

**Source:** [https://attack.mitre.org/matrices/enterprise/cloud/](https://attack.mitre.org/matrices/enterprise/cloud/)

The cloud-specific subset of the MITRE ATT&CK framework. Covers techniques specific to AWS, Azure, GCP, and container environments. Includes tactics for initial access via exposed credentials, privilege escalation via IAM policy manipulation, and data exfiltration via storage buckets.

Relevant to this skill when: modeling threats against cloud-native or hybrid architectures where adversaries operate through cloud provider APIs rather than traditional network vectors.
