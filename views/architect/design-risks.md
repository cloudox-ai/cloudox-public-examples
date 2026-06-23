# Architect View — Design Risks

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Likely_

## Design Risks

The most consequential architectural risk here is a **detection blind spot spanning the majority of the account estate**: both GuardDuty and IAM Access Analyzer are enabled in only 1 of 6 scanned accounts, leaving five accounts — including production and the management plane — without consistent threat detection or external-access analysis. This is a structural gap, not an isolated misconfiguration, and it shapes how any workload running in those accounts should be assessed for blast radius.

*Overall confidence for this section: Likely — derived from graph evidence; some unknowns remain.*

### Architectural Risks

#### Uneven GuardDuty Coverage (5 accounts uncovered)

GuardDuty threat detection is absent from **Workload Dev Account** (`105769365151`), **Workload Prod Account** (`122122642149`), **Management Account** (`110319895932`), **Sandbox Ma Account** (`161388682021`), and **Log Archive Account** (`122980216815`). Only one of the six scanned accounts has it active. [`risk:security:aws-guardduty-detector`]

For architects, the key concern is that the **production workload account and the management account are both uncovered**. Any compromise or anomalous API activity in those accounts — including lateral movement from a misconfigured role or a leaked credential — will not generate findings. The Log Archive Account being uncovered is also notable: that account is typically a high-value target precisely because it holds audit trails.

**Recommended action:** Enable GuardDuty org-wide via a delegated administrator. This eliminates per-account enablement drift and ensures new accounts are covered automatically.

#### Uneven IAM Access Analyzer Coverage (5 accounts uncovered)

IAM Access Analyzer is missing from the same five accounts (`122980216815`, `105769365151`, `122122642149`, `110319895932`, `161388682021`). [`risk:security:aws-accessanalyzer-analyzer`]

From an architecture standpoint, this means **unintended cross-account or internet-accessible resource policies** — on S3 buckets, KMS keys, SQS queues, API Gateway endpoints, or DynamoDB tables — will not be surfaced automatically in those accounts. Given that the environment includes internet-facing API Gateway endpoints (`https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`, `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`) and multiple internet gateways across accounts (`igw-00ed21b9a0e6596a8`, `igw-0d14f1dd4e54d5906`, `igw-0567575921f471548`, `igw-0cff0d66b4fd90803`), the absence of Access Analyzer in the workload accounts is a meaningful gap in the external-exposure detection chain.

**Recommended action:** Enable IAM Access Analyzer org-wide via a delegated administrator, mirroring the GuardDuty remediation pattern.

> **Note:** No Security Hub enablement was discovered in this environment. Security Hub would provide a unified findings aggregator for both GuardDuty and Access Analyzer signals across accounts — its absence means there is no centralised control plane for these detections even after individual services are enabled.

### Technical Debt Signals

#### Broadly-Privileged IAM Role in Sandbox

The IAM role **`cloudox-demo-sandbox-unused-admin`** (`AROAAAAADPCL3BVEXUDTH`) in **Sandbox Ma Account** (`161388682021`) carries a name strongly suggesting broad administrative access. [`risk:security:cloudox-demo-sandbox-unused-admin`]

> **Confidence: Assumed.** Privilege breadth was not collected; this risk is inferred from naming convention and must be validated by reviewing the attached policies directly.

The architectural concern is **identity blast radius**: if this role is assumable by a broad principal set — or if it is unused but not deactivated — it represents a standing privilege that could be exploited. Sandbox accounts are frequently used as pivot points in multi-account environments.

**Recommended action:** Review attached policies; apply least-privilege scoping. If the role is genuinely unused, consider deactivating or deleting it to reduce the standing attack surface.

#### Tagging Gaps Limiting Environment Classification

761 resources across the environment have no Environment / Stage / Tier tag and rely on inference for classification. This is a technical debt signal with architectural consequences: without consistent tagging, automated guardrails (SCPs, resource policies, cost allocation, and blast-radius scoping) cannot be reliably applied by environment boundary. This also makes it harder to reason about which workloads are production-grade and which are not.
