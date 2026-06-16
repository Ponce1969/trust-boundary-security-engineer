# IAC and Cloud Misconfiguration Review

A worked walkthrough demonstrating how to apply the Security Engineering Skill to infrastructure-as-code and cloud configuration review. This example covers Terraform, IAM policies, and CI/CD pipeline secrets.

This example shows the reasoning process for cloud-native attack surfaces: misconfigured IAM roles, overly permissive storage policies, and secrets exposed in pipeline configuration. It follows the same six-step model as the other examples. → `specs/SPEC-005-agent-operational-behavior.md`

---

## Scenario

A startup deploys a web application on AWS using Terraform. The infrastructure team shares the following code for review before the first production deployment:

**`main.tf` — IAM role for the application:**

```hcl
resource "aws_iam_role" "app_role" {
  name = "app-production-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Effect    = "Allow"
      Principal = { Service = "ec2.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy" "app_policy" {
  name = "app-production-policy"
  role = aws_iam_role.app_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action   = "*"
      Effect   = "Allow"
      Resource = "*"
    }]
  })
}
```

**`storage.tf` — S3 bucket for user uploads:**

```hcl
resource "aws_s3_bucket" "uploads" {
  bucket = "myapp-user-uploads-prod"
}

resource "aws_s3_bucket_public_access_block" "uploads" {
  bucket = aws_s3_bucket.uploads.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_policy" "uploads" {
  bucket = aws_s3_bucket.uploads.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "s3:GetObject"
      Effect    = "Allow"
      Resource  = "arn:aws:s3:::myapp-user-uploads-prod/*"
      Principal = "*"
    }]
  })
}
```

**`.github/workflows/deploy.yml` — CI/CD deployment pipeline:**

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        run: |
          export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
          export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
          export AWS_DEFAULT_REGION=us-east-1

      - name: Deploy application
        run: ./deploy.sh
```

---

## Step 1 — Perceive Input

The reviewer identifies three infrastructure files covering identity, storage, and deployment:

**Spatial signals:**
- IAM role attached to an EC2 instance — permission boundary between the instance and AWS services → `knowledge/least-privilege.md`
- S3 bucket with public access configuration — potential data exposure boundary → `knowledge/trust-boundaries.md`
- CI/CD pipeline with AWS credentials — secret lifecycle and supply chain boundary → `knowledge/secrets-management.md`, `knowledge/supply-chain-security.md`

**Immediate observations before full analysis:**
- The IAM policy uses `Action: "*"` and `Resource: "*"` — wildcards on both dimensions.
- The S3 public access block has all four settings set to `false` — all public access protections disabled.
- AWS credentials appear as plaintext strings in the pipeline YAML file.

The reviewer flags these three signals immediately. Each represents a different category of misconfiguration: privilege, data exposure, and secrets. The execution model is sequential (Terraform applies resources one at a time), but the IAM and S3 configurations create persistent attack surface that exists independently of execution timing.

## Step 2 — Construct Context

**Execution model classification:** The infrastructure is declared statically. Once deployed, the configuration is persistent state, not runtime behavior. There are no concurrency concerns in the IAC itself. The threat model is spatial: what can an actor do given these permissions and configurations? → `specs/SPEC-005-agent-operational-behavior.md`

**Spatial trust boundaries:**
- Boundary 1: Internet → S3 bucket. The public access block configuration controls what the internet can read.
- Boundary 2: EC2 instance → AWS APIs. The IAM role controls what the application can do across all AWS services.
- Boundary 3: GitHub Actions runner → AWS. The CI/CD pipeline authenticates to AWS and deploys. The credentials in the YAML file are the authentication mechanism.
- Boundary 4: GitHub repository → CI/CD runner. Any developer with read access to the repository can read the YAML file and extract the credentials.

**Context notes:**
- This is a production environment. The startup's first production deployment means these configurations will be active against real user data.
- The S3 bucket name is `myapp-user-uploads-prod` — it stores user uploads. Public read access to user uploads creates user data exposure beyond just operational data.
- The IAM wildcard policy is attached to an EC2 instance, not a human user. An EC2 instance with full `*/*` permissions is effectively a root-equivalent actor within the AWS account.

## Step 3 — Generate Hypotheses

The reviewer generates hypotheses for each trust boundary:

**Boundary 1 — S3 public access:**

- H1: **Unrestricted public read of user uploads.** The S3 bucket policy grants `s3:GetObject` to `Principal: "*"` with public access blocks disabled. Any URL of the form `https://myapp-user-uploads-prod.s3.amazonaws.com/<key>` is publicly accessible without authentication. → `knowledge/trust-boundaries.md`

- H2: **Object enumeration if bucket listing is also public.** The current policy grants `s3:GetObject`. If bucket listing (`s3:ListBucket`) is also permitted — either through this policy or ACLs — an attacker can enumerate all stored objects. → `knowledge/least-privilege.md`

**Boundary 2 — IAM wildcard policy:**

- H3: **Full AWS account compromise via instance metadata.** An EC2 instance with `Action: "*"` on `Resource: "*"` can perform any operation in the AWS account: create IAM users, disable CloudTrail logging, delete S3 buckets, exfiltrate data from RDS, access Secrets Manager, modify security groups. If the application running on this EC2 instance has any code execution vulnerability — server-side request forgery, command injection, deserialization — an attacker can retrieve the instance's IAM credentials via the metadata endpoint (`http://169.254.169.254/latest/meta-data/iam/security-credentials/`) and operate as a root-equivalent actor in the account. → `knowledge/least-privilege.md`, `knowledge/input-validation.md`

- H4: **Privilege escalation path for any user who compromises the instance.** The IAM wildcard policy means the instance can create additional IAM users, attach policies, and establish persistence beyond the instance itself. A temporary compromise of the application layer becomes a permanent account-level compromise. → `knowledge/defense-in-depth.md`

**Boundary 3 and 4 — Hardcoded credentials in CI/CD:**

- H5: **Long-lived static credentials exposed in source code.** The AWS access key and secret appear as plaintext strings in the YAML file. The format `AKIA...` indicates a long-lived IAM user access key, not a temporary STS credential. Long-lived keys do not expire, are not scoped to the pipeline's context, and can be used from any IP address. → `knowledge/secrets-management.md`

- H6: **Supply chain exposure via repository access.** The credentials are in a YAML file committed to the repository. Anyone with read access to the repository — any contributor, any developer given access, any service that reads the repository — can extract these credentials. GitHub Actions secrets exist precisely to prevent this pattern. → `knowledge/supply-chain-security.md`

- H7: **Credential exposure in CI/CD logs.** When credentials are passed via `export` in a `run:` step, they may appear in CI/CD logs depending on the pipeline's log verbosity settings. If logs are retained and accessible to a broader audience than the secrets themselves, the credentials are exposed through a secondary channel. → `knowledge/secrets-management.md`

## Step 4 — Evaluate Evidence

**H1 (Unrestricted public read):**
- `block_public_acls: false`, `block_public_policy: false`, `restrict_public_buckets: false` — all four public access controls are explicitly disabled.
- The bucket policy grants `s3:GetObject` to `Principal: "*"`.
- **Confirmed by directly observable configuration.** The reviewer does not need to test the bucket — the Terraform declaration is the configuration. Confidence: **Observed.**

**H2 (Object enumeration):**
- The bucket policy grants only `s3:GetObject`, not `s3:ListBucket`.
- However, public access blocks are disabled, which means ACLs could also grant listing permissions.
- The Terraform code does not show ACL configuration explicitly.
- Confidence: **Inferred.** The current policy does not explicitly allow listing, but the disabled public access blocks mean an ACL-based listing grant would take effect if present. `[REQUIRES_EXTERNAL_EVIDENCE: S3_ACL_CONFIGURATION]`

**H3 (Full account compromise via metadata):**
- The IAM policy shows `Action: "*"` and `Resource: "*"` — this is a wildcard policy with no restrictions.
- AWS documents the instance metadata service endpoint at `http://169.254.169.254/latest/meta-data/iam/security-credentials/` — any process on the EC2 instance can retrieve the IAM credentials without authentication.
- **Confirmed by absence of control.** There is no IAM permission boundary, no `Condition` block restricting the policy, and no evidence of IMDSv2 enforcement (which would require a session token to access the metadata service). → `specs/SPEC-005-agent-operational-behavior.md`
- **Impact amplification:** The wildcard IAM policy means a single application-layer vulnerability (SSRF, RCE, SSTI) escalates directly to full AWS account compromise. → `knowledge/defense-in-depth.md`

**H4 (Privilege escalation persistence):**
- A wildcard IAM policy that includes `iam:CreateUser`, `iam:AttachUserPolicy`, and `iam:CreateAccessKey` (all covered by `Action: "*"`) means the instance can create persistent backdoor credentials.
- **Confirmed by absence of control.** The policy has no explicit deny on IAM write operations. Confidence: **Observed** — the policy text is the evidence.

**H5 (Long-lived static credentials):**
- The format `AKIA...` is the AWS prefix for long-lived IAM user access keys. This is documented by AWS.
- The value appears as a plaintext string in the YAML file.
- **Confirmed by directly observable configuration.** The credentials are visible in the code. Confidence: **Observed.**

**H6 (Supply chain exposure):**
- The credentials are in a committed file. Once committed, they exist in git history even if removed from the current HEAD.
- **Confirmed.** The exposure is structural, not speculative.

**H7 (Credential exposure in logs):**
- The `export` command in a `run:` step may echo the variable assignment in debug-level logs.
- GitHub Actions masks secrets defined via `secrets:` but does not mask values set via `export` in `run:` steps.
- **Confirmed by absence of control.** The credentials are not using GitHub's secrets masking mechanism.

## Step 5 — Synthesize Decisions

**Finding 1 (Observed — H5, H6): Hardcoded long-lived AWS credentials in CI/CD pipeline.** Structurally guaranteed — the credentials exist in the file and the git history.

Long-lived IAM user credentials are hardcoded as plaintext strings in the deployment pipeline YAML. The format `AKIA...` confirms these are permanent access keys, not temporary STS tokens. Anyone with read access to the repository has these credentials. They are not subject to the pipeline's lifecycle — they do not expire when a deployment completes, a branch is deleted, or a developer leaves the team.

- **Impact:** Full, persistent AWS access for anyone who reads the repository. The credentials are valid indefinitely unless manually revoked.
- **Recommendation:** Remove the credentials from the file immediately. Use GitHub Actions OIDC to obtain short-lived STS credentials for the deployment pipeline — this eliminates the need for any stored credential entirely. If OIDC is not immediately available, use GitHub Actions secrets (`${{ secrets.AWS_ACCESS_KEY_ID }}`) rather than plaintext. Rotate the exposed credentials now, regardless of the fix applied. → `principles/minimal-change.md`

**Finding 2 (Observed — H1): S3 bucket with all public access protections disabled and a public read policy.** Structurally guaranteed — the configuration is explicit and persistent.

All four S3 public access block settings are `false`. The bucket policy grants unauthenticated read access to all objects (`Principal: "*"`). User uploads are accessible to anyone with the object URL. The bucket name is guessable and the object key structure may be enumerable.

- **Impact:** All files uploaded by users are publicly readable without authentication. Depending on what users upload, this may include private documents, images, or sensitive data.
- **Recommendation:** Set all four `block_public_acls`, `block_public_policy`, `ignore_public_acls`, and `restrict_public_buckets` to `true`. Replace the public policy with pre-signed URLs generated at request time, scoped to the specific object and time window needed. → `principles/minimal-change.md`

**Finding 3 (Confirmed by absence of control — H3, H4): IAM wildcard policy grants full AWS account permissions to an EC2 instance.** Structurally guaranteed — the policy applies persistently to the instance.

The IAM role attached to the production EC2 instance has `Action: "*"` on `Resource: "*"` with no permission boundaries, no condition blocks, and no explicit denies on sensitive operations. Any process running on the instance — including the application code, any dependency it loads, or any attacker who achieves code execution through an application vulnerability — can perform any operation in the AWS account.

The instance metadata service provides unauthenticated access to these credentials from within the instance. No SSRF protection or IMDSv2 enforcement is visible.

- **Impact:** Any application-layer vulnerability (SSRF, RCE, deserialization, SSTI, path traversal) escalates directly to full AWS account compromise. This includes access to all other services, data stores, and the ability to create persistent backdoor credentials.
- **Recommendation:** Replace the wildcard policy with a policy that grants only the specific actions and resources the application requires. Apply least privilege: if the application reads from one S3 bucket and writes to one DynamoDB table, the policy should name those resources explicitly. Enforce IMDSv2 on the instance to require a session token for metadata access, reducing the blast radius of SSRF vulnerabilities. → `knowledge/least-privilege.md`, `principles/minimal-change.md`

## Step 6 — Construct Output

**Observations:**
- IAM role policy grants `Action: "*"` on `Resource: "*"` — full account permissions for the EC2 instance.
- S3 bucket has all four public access block settings set to `false` with a public read policy.
- AWS access key ID (`AKIA...` prefix, indicating a permanent key) and secret are hardcoded as plaintext in the deployment YAML.
- No IMDSv2 enforcement is visible in the Terraform configuration.
- No permission boundaries or explicit deny policies are applied to the IAM role.

**Inferences:**
- Any code execution on the EC2 instance results in full AWS account compromise via the instance metadata endpoint.
- All objects in the user uploads bucket are publicly readable without authentication.
- The hardcoded credentials are accessible to anyone with repository read access and persist in git history.
- Object enumeration is possible if ACLs grant ListBucket (requires verification of ACL configuration).

**Risks:**
- An attacker who exploits any application vulnerability (SSRF, RCE, injection) gains root-equivalent access to the entire AWS account.
- User-uploaded files are readable by unauthenticated external actors.
- The hardcoded credentials enable unauthorized AWS access from outside the pipeline, indefinitely.
- The instance can create persistent backdoor credentials, making a temporary compromise permanent.

All findings are presented for human review before any action is taken. → `principles/human-in-the-loop.md`

---

## What This Example Demonstrates

- **Cloud configurations are security-relevant code.** Terraform files, IAM policies, and CI/CD YAML are not infrastructure setup — they are security decisions with permanent consequences once deployed. The same reasoning model applies to them as to application code. → `specs/SPEC-005-agent-operational-behavior.md`
- **Structurally guaranteed findings dominate cloud misconfiguration.** Unlike race conditions, IAM wildcards and public S3 policies are not probabilistic — they apply on every request, to every actor, without conditions. The execution model is irrelevant; the configuration is the vulnerability. → `specs/SPEC-005-agent-operational-behavior.md`
- **Confirmed by absence of control applies to IAC.** The absence of permission boundaries, IMDSv2 enforcement, and S3 public access blocks are each evidence — not gaps in analysis. → `specs/SPEC-005-agent-operational-behavior.md`
- **Blast radius amplification is a distinct finding.** The wildcard IAM policy does not just create a vulnerability — it amplifies every other vulnerability in the system. This is a defense-in-depth failure that compounds the impact of unrelated application flaws. → `knowledge/defense-in-depth.md`
- **Secrets in CI/CD are supply chain vulnerabilities.** Credentials committed to a repository are not secrets — they are public keys with delayed discovery. The supply chain boundary is the repository, not the deployment environment. → `knowledge/supply-chain-security.md`
- **Context determines recommendation priority.** Finding 1 (hardcoded credentials) requires immediate action — rotation now, fix applied immediately. Finding 3 (IAM wildcard) also requires urgent remediation but can be planned over hours rather than minutes. Prioritization is context-driven, not category-driven. → `principles/context-matters.md`
- **Proportional recommendations avoid over-engineering.** Each fix is the smallest safe change: scoped IAM policy, pre-signed URLs, OIDC for CI/CD. No architectural rewrite. → `principles/minimal-change.md`
