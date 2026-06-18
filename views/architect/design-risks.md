# Architect View — Design Risks

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Architect View](./README.md) · Audience: Solutions / Cloud Architects · Confidence: Likely_

## Design Risks

The most consequential finding across this environment is a **systematic gap in detective security controls**: both GuardDuty and IAM Access Analyzer are enabled in only 1 of 6 scanned accounts, leaving the majority of the estate — including production, dev, management, and log archive accounts — without consistent threat detection or external-access analysis. This is an architectural control-plane gap, not an isolated misconfiguration, and it affects the blast radius of any future compromise.

### Architectural Risks

Two medium-severity, org-wide risks dominate the design-risk picture. They share the same affected account set and the same remediation pattern, which makes them efficient to address together.

| Risk | Severity | Accounts Uncovered | Recommended Action |
|---|---|---|---|
| Uneven GuardDuty coverage (`risk:security:aws-guardduty-detector`) | Medium | Log Archive (122980216815), Workload Dev (105769365151), Workload Prod (122122642149), Management (110319895932), Sandbox Ma (161388682021) | Enable org-wide via delegated administrator |
| Uneven IAM Access Analyzer coverage (`risk:security:aws-accessanalyzer-analyzer`) | Medium | Same 5 accounts as above | Enable org-wide via delegated administrator |

**GuardDuty gap** (`risk:security:aws-guardduty-detector`): Threat detection is absent from 5 of 6 scanned accounts, including the Workload Prod Account (122122642149) and the Management Account (110319895932). The management account is a particularly sensitive gap — compromise there has org-wide reach. The recommended architectural fix is to designate a delegated administrator account and push GuardDuty enablement org-wide, rather than enabling it account-by-account.

**IAM Access Analyzer gap** (`risk:security:aws-accessanalyzer-analyzer`): External-access analysis is similarly absent from the same 5 accounts. Without Access Analyzer, resource policies that inadvertently grant cross-account or public access (on S3 buckets, KMS keys, API Gateway endpoints, DynamoDB tables, etc.) will go undetected. Given that internet-facing API Gateway endpoints are present in eu-central-1 (`https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`, `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`) and internet gateways exist across multiple accounts and regions, the absence of Access Analyzer in the Workload Dev and Workload Prod accounts is a meaningful exposure. The same delegated-administrator pattern applies.

**No Security Hub enablement discovered**: The package notes that Security Hub has not been found in this environment. Combined with the GuardDuty and Access Analyzer gaps, this means there is no centralised aggregation point for security findings — a structural gap for any architect designing a detection and response capability.

**Internet gateway spread**: Internet gateways are present across at least four accounts and two regions (us-east-1 and eu-central-1), including in the Log Archive Account (igw-0cff0d66b4fd90803) and Workload Dev Account (igw-0567575921f471548). The architectural intent behind internet gateway presence in a log archive account is not evident from available evidence and warrants review.

### Technical Debt Signals

**Broadly-privileged IAM role in Sandbox** (`risk:security:cloudox-demo-sandbox-unused-admin`): The IAM role `cloudox-demo-sandbox-unused-admin` (AROAAAAADPCL3BVEXUDTH) in the Sandbox Ma Account (161388682021) carries a name that suggests broad administrative access. *Confidence on this item is Assumed* — privilege breadth was not directly collected and is inferred from naming alone. Validation against the attached policies is required before acting. If the name is accurate, the role represents an over-privilege risk with a large identity blast radius. Recommended action: review attached policies and apply least privilege.

**Tagging debt**: 761 of 807 resources carry no Environment / Stage / Tier tag, meaning workload classification across this estate relies on inference rather than explicit metadata. For architects, this limits the reliability of any environment-boundary analysis, cost allocation, or automated policy enforcement that depends on resource tags. Addressing tagging at the infrastructure-as-code level is the durable fix.

> **Confidence note (Likely):** The GuardDuty and Access Analyzer findings are derived from graph evidence and are assessed as Likely. The IAM role risk is Assumed and must be validated. Security Hub absence is an unknown (not confirmed disabled — simply not discovered).
