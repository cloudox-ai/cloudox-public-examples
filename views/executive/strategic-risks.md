# Executive View — Strategic Risks

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Likely_

## Strategic Risks

Three security gaps stand out across the organisation's AWS accounts — two of them affect the majority of the estate and warrant prompt remediation planning.

### Business & Security Risks

**Threat detection and access analysis are largely absent.** GuardDuty (threat detection) and IAM Access Analyzer (external-access visibility) are each enabled in only 1 of 6 scanned accounts. The five accounts without coverage include the Management Account, Workload Prod Account, Workload Dev Account, Log Archive Account, and Sandbox Ma Account. Running production workloads without GuardDuty means active threats — credential compromise, unusual API calls, network reconnaissance — can go undetected. The absence of IAM Access Analyzer means unintended external access to resources may not be surfaced at all.

| Risk | Accounts Uncovered | Severity |
|---|---|---|
| GuardDuty not enabled | 5 of 6 | Medium |
| IAM Access Analyzer not enabled | 5 of 6 | Medium |

The recommended path for both is org-wide enablement via a delegated administrator — a single action that closes the gap across all accounts simultaneously.

**A broadly-privileged IAM role exists in the Sandbox Ma Account.** The role `cloudox-demo-sandbox-unused-admin` carries a name strongly suggesting administrative access. Its actual policy breadth has not been collected, so the true blast radius is unconfirmed (confidence: Assumed). Until reviewed, it represents a potential over-privilege risk — particularly if the role can be assumed by unintended principals or is no longer actively needed. The recommended action is to review attached policies and apply least privilege.

**Additional context.** Two security groups are open to the internet, and Security Hub enablement has not been discovered across the estate. With 761 of 807 resources lacking environment or stage tags, classification of what is production versus non-production relies on inference rather than explicit tagging — limiting the accuracy of any automated risk prioritisation.

> **Confidence: Likely.** Findings are derived from graph evidence. The IAM role risk is Assumed and requires manual validation before acting on it.
