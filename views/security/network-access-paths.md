# Security View — Network Access Paths

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

> _Part of the [Security View](./README.md) · Audience: Security & Governance teams · Confidence: Verified_

## Network Access Paths

**Confidence: Verified** — Derived from complete graph evidence for this domain.

Three separate VPCs, each with a direct internet gateway attachment, define the external exposure boundary for this environment. All observed subnets in scope are public-facing, meaning resources placed in them are reachable from the internet subject only to security group and network ACL controls. This is the most significant network-access posture signal available from the current evidence set and warrants explicit validation of what is actually running in those subnets.

### Ingress Paths

Three VPCs are confirmed across two accounts and one sandbox account, all in `eu-central-1`, each with a dedicated internet gateway:

| VPC | Account | Internet Gateway | Environment Signal |
|---|---|---|---|
| Cloudox Demo Atlas Prod VPC (`vpc-0aaa6d4f2f981e945`) | `122122642149` | Cloudox Demo Atlas Prod Igw (`igw-01dd5625ec1ea7a54`) | Production |
| Cloudox Demo Atlas Dev VPC (`vpc-0a4d44a07f48d7ca0`) | `105769365151` | Cloudox Demo Atlas Dev Igw (`igw-0155958a0a5e9c500`) | Development |
| Cloudox Demo Sandbox VPC (`vpc-0c97d53850c027a14`) | `161388682021` | Sandbox Internet Gateway (`igw-08017528fa36ccb4d`) | Sandbox |

All five confirmed subnets in scope are classified as **Public Subnets** in `eu-central-1`:

- `subnet-016c22941a019a137` (account `122122642149` — Prod)
- `subnet-013a24d318ed6f3d0` (account `122122642149` — Prod)
- `subnet-0f64d71c952a7898a` (account `105769365151` — Dev)
- `subnet-065f522206524ab12` (account `105769365151` — Dev)
- `subnet-029e2cceb3d0beff7` (account `161388682021` — Sandbox)

The presence of public subnets in the **production** VPC (`vpc-0aaa6d4f2f981e945`) is the highest-priority exposure signal here. Security and governance teams should confirm which compute or data resources are placed in `subnet-016c22941a019a137` and `subnet-013a24d318ed6f3d0`, and verify that security groups and NACLs are the intended last line of defence for those resources.

AWS-managed prefix lists `pl-66a5400f` and `pl-6ea54007` (eu-central-1 regional prefix lists) are referenced in the evidence, indicating that at least some access rules are scoped to AWS-managed IP ranges rather than arbitrary open CIDRs — however, the specific rules and resources they are attached to are not detailed in this section's evidence set.

A **DynamoDB Interface Endpoint** (`vpce-02ddbd8d63a6ab447`) is present in the Prod account (`122122642149`), providing a private path to DynamoDB from within the Prod VPC. This reduces the need for DynamoDB traffic to traverse the public internet from that VPC.

### Lateral Connectivity

The package confirms the existence of three distinct VPCs across three accounts. No VPC peering connections, Transit Gateway attachments, or PrivateLink cross-VPC service endpoints are recorded in this section's evidence. The absence of observed lateral connectivity paths between Prod, Dev, and Sandbox VPCs is noted, but coverage of inter-VPC routing is not guaranteed to be complete from this section alone — other sections may carry additional evidence.

One architecture-level dependency concern is recorded that has direct network-access implications:

> **Critical dependency: DBInstance `cloudox-demo-atlas-prod-pg`** (`dependency_concern:architecture:cloudox-demo-atlas-prod-pg`) — The workload **Cloudox Demo Atlas Prod API** (`cloudox-demo-atlas-prod-api`, account `122122642149`, `eu-central-1`) depends on the datastore `cloudox-demo-atlas-prod-pg`, which is **not Multi-AZ**. A zone failure would take the data tier offline for this production workload. *Confidence on the workload entity: Likely; confidence on the datastore entity: Unknown.*

From a governance perspective, this means the network path to the production database is a single point of failure: if the availability zone hosting `cloudox-demo-atlas-prod-pg` becomes unreachable, the dependent API loses its data tier entirely. The recommended action is to confirm whether Multi-AZ or automated backups are configured, and to review how the API handles a data-tier failure (graceful degradation vs. hard outage).

> **Evidence gap:** No Security Hub enablement was discovered for this environment. This means there is no confirmed centralised findings aggregation for network-level security signals (e.g., GuardDuty network findings, Security Hub network reachability checks). Governance teams should treat network exposure assessments as potentially incomplete until Security Hub or an equivalent control is confirmed active.

![Workload VPC network topology](./diagrams/security-network-topology.png)
