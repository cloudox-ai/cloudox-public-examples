# Generic View — Operations & Reliability

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Generic View](./README.md) · Audience: Any technical reader · Confidence: Verified_

## Operations & Reliability

807 resources were captured across this environment with no coverage gaps identified in the collected data. The two known unknowns below are worth keeping in mind when interpreting completeness.

> **Confidence: Verified** — derived from complete graph evidence for the domains in scope (networking, coverage).

### Reliability Signals

No reliability signal items were present in this section's package. Coverage breadth is the primary reliability-relevant data point available here: 807 resources were captured with 0 coverage gaps flagged, suggesting the typed collectors reached their intended targets without missing resource categories they were configured to collect.

Two AWS-managed prefix lists are referenced in the networking evidence:
- **Amazon S3 prefix list** (`pl-6ea54007`, `arn:aws:ec2:eu-central-1:aws:prefix-list/pl-6ea54007`) — eu-central-1
- **Amazon DynamoDB prefix list** (`pl-66a5400f`, `arn:aws:ec2:eu-central-1:aws:prefix-list/pl-66a5400f`) — eu-central-1

The presence of these AWS-managed prefix lists in eu-central-1 indicates that at least some network paths in this region are scoped to AWS service endpoints (S3 and DynamoDB), which is a common pattern for keeping service traffic off the public internet via VPC routing or security group rules.

### Operational Concerns

Two meta-collector gaps reduce the confidence that the 807-resource count represents the full AWS-visible estate:

| Concern | Impact |
|---|---|
| **Resource Explorer meta-collector disabled or unavailable** | AWS-visible breadth could not be cross-checked against the typed collector results. Resources in regions or of types not covered by typed collectors may be absent. |
| **Cloud Control meta-collector disabled** | Long-tail and newer resource types that lack a dedicated typed collector are not represented. The inventory may undercount less common resource types. |

Neither gap affects the reliability of what *was* collected, but both mean the inventory should not be treated as exhaustive. Enabling Resource Explorer and Cloud Control collection in a future run would allow a completeness cross-check.
