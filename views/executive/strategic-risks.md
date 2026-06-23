# Executive View — Strategic Risks

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Executive View](./README.md) · Audience: CTO / Engineering leadership · Confidence: Likely_

## Strategic Risks

### Business & Security Risks

The most significant pattern across this environment is **incomplete coverage of core AWS security services** — GuardDuty threat detection and IAM Access Analyzer are each active in only 1 of 6 scanned accounts. The five accounts without these controls include the Management Account, Workload Prod Account, Workload Dev Account, Log Archive Account, and Sandbox Ma Account. This means the majority of the estate — including production and the management plane — is operating without automated threat detection or external-access analysis. The fix is straightforward: enable both services org-wide via a delegated administrator, which eliminates the coverage gap without per-account effort.

A secondary concern is an IAM role named **cloudox-demo-sandbox-unused-admin** in the Sandbox Ma Account. Its name suggests broad administrative access, and the role appears unused. Confidence here is lower (Assumed) — privilege breadth has not been directly verified — but the naming pattern warrants a prompt review and least-privilege remediation before the role is exploited or inherited by another workload.

| Risk | Accounts Affected | Severity | Action |
|---|---|---|---|
| GuardDuty not enabled | 5 of 6 (incl. Prod, Management) | Medium | Enable org-wide via delegated admin |
| IAM Access Analyzer not enabled | 5 of 6 (incl. Prod, Management) | Medium | Enable org-wide via delegated admin |
| Broadly-privileged unused IAM role | Sandbox Ma Account | Medium | Review and apply least privilege |

**Two material gaps** limit full confidence in this picture: Security Hub enablement was not discovered in the environment, and 761 resources carry no Environment/Stage/Tier tag, meaning their classification relies on inference rather than explicit labelling. Both gaps should be addressed to improve visibility and future risk assessments.
