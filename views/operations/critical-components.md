# Operations View — Critical Components

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Operations View](./README.md) · Audience: Platform / Operations Engineers · Confidence: Likely_

## Critical Components

The most operationally significant finding in this section is a single-AZ database instance sitting directly in the critical path of a production workload — a zone failure would take the entire data tier offline with no automatic failover.

### Critical Workloads

The following workloads are identified as key entities in this environment:

| Friendly Name | Account | Region | Confidence |
|---|---|---|---|
| Cloudox Demo Atlas Prod API (`cloudox-demo-atlas-prod-api`) | 122122642149 | eu-central-1 | Likely |
| Cloudox (`cloudox`) | 122122642149 | eu-central-1 | Verified |
| Cloudox Demo Atlas Dev API (`cloudox-demo-atlas-dev-api`) | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Atlas Dev (`cloudox-demo-atlas-dev`) | 105769365151 | eu-central-1 | Likely |
| Cloudox Demo Sandbox Scratch (`cloudox-demo-sandbox-scratch`) | 161388682021 | eu-central-1 | Assumed |

> **Coverage gap:** 761 resources carry no Environment / Stage / Tier tag; their classification relies on inference. Workload boundaries and criticality assignments for untagged resources should be treated as approximate until tagging is enforced.

Internet-facing API Gateway endpoints are present across multiple accounts and regions, indicating externally reachable surfaces for at least some of these workloads:
- `https://gfwaiva01f.execute-api.eu-central-1.amazonaws.com`
- `https://xdmn5ldmif.execute-api.eu-central-1.amazonaws.com`

Internet gateways are also present in both `eu-central-1` and `us-east-1` across accounts `110019496666`, `105769365151`, and `122980216815` (`igw-0d14f1dd4e54d5906`, `igw-00ed21b9a0e6596a8`, `igw-0567575921f471548`, `igw-0cff0d66b4fd90803`), confirming outbound/inbound internet paths exist in the environment.

### Shared Dependencies

**⚠ Single-AZ Datastore in Production Critical Path**

Intelligence item `dependency_concern:architecture:cloudox-demo-atlas-prod-pg` (priority 3, significance 0.57) identifies a verified architectural risk:

- **Affected datastore:** `cloudox-demo-atlas-prod-pg` (DBInstance)
- **Affected workload:** Cloudox Demo Atlas Prod API (`cloudox-demo-atlas-prod-api`, account 122122642149, eu-central-1)
- **Impact:** The datastore is not Multi-AZ. A single Availability Zone failure will take the production data tier offline for the dependent workload. There is no evidence of automatic failover.
- **What can break:** Any operation served by Cloudox Demo Atlas Prod API that requires database reads or writes will fail for the duration of a zone outage.
- **Recovery risk:** Recovery depends on manual intervention or a restore-from-backup process. RTO is unknown — no backup or recovery configuration evidence is present in this package.
- **Recommended action:** Confirm whether Multi-AZ is enabled or can be enabled on `cloudox-demo-atlas-prod-pg`. Verify that automated backups are configured and that the dependent workload has a tested failover or degraded-mode path for data-tier unavailability.

DynamoDB tables are present across the dev, prod, and sandbox workloads:
- `arn:aws:dynamodb:eu-central-1:105769365151:table/cloudox-demo-atlas-dev-items` (dev)
- `arn:aws:dynamodb:eu-central-1:122122642149:table/cloudox-demo-atlas-prod-items` (prod)
- `arn:aws:dynamodb:eu-central-1:161388682021:table/cloudox-demo-sandbox-scratch` (sandbox)

No evidence is available in this package regarding backup configuration, point-in-time recovery status, or replication settings for these tables. This is a coverage gap that warrants direct verification.

> **Observability gap:** No evidence of monitoring, alerting, or on-call runbooks for `cloudox-demo-atlas-prod-pg` or the DynamoDB tables is present in this package. Failure detection and escalation paths are unknown.

![Workload architecture](./diagrams/operations-workload-architecture.png)
