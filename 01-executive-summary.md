> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

## Key Takeaways

- **A structured multi-account AWS Organization is in place** across 7 accounts in 4 OUs (Root, Security, Platform, Workloads, Sandbox), with dedicated `audit` (110019496666) and `log-archive` (122980216815) accounts suggesting a security baseline pattern — though the activation state of those controls is not confirmed by the available data.
- **Internet exposure is confirmed in the production workload account** (122122642149): ALB `cloudox-demo-atlas-prod-alb` is internet-facing, three security groups allow `0.0.0.0/0` or `::/0` ingress, and five subnets route to an Internet Gateway.
- **Five inferred workloads were detected**, all named under a `cloudox-demo-atlas-*` pattern across `workload-prod` (122122642149) and `workload-dev` (105769365151), plus a single Lambda in the sandbox account (161388682021). No workloads were detected in the management, audit, log-archive, or platform accounts.
- **No explicit risk records were generated** (`risk_count: 0`), but 9 findings with high-confidence exposure classifications exist and warrant review — see [`07-findings-and-risks.md`](07-findings-and-risks.md).
- **Environment classification is incomplete**: the management account (110319895932), audit account (110019496666), and log-archive account (122980216815) carry `classification: unknown` with low confidence. The platform account (150982215529) is inferred as production with medium confidence only.

---

## Scope at a Glance

| Dimension | Value | Source |
|---|---|---|
| AWS Organization | `o-aaaapzvebq` | `organizations` block |
| Accounts | 7 | `graph_stats.account_count` |
| Environments (classified) | 7 total; 3 unknown classification | `environments` block |
| Total resources | 800 | `graph_stats.resource_count_total` |
| Total relationships | 264 | `graph_stats.relationship_count_total` |
| Findings | 9 (all exposure category) | `graph_stats.finding_count` |
| Explicit risk records | 0 | `graph_stats.risk_count` |
| Inferred workloads | 5 | `graph_stats.workload_count` |
| Primary active region | eu-central-1 | Workload anchor resources |

---

## Account and Environment Structure

| Account Alias | Account ID | OU Path | Environment Classification | Confidence |
|---|---|---|---|---|
| CloudoX (management) | 110319895932 | /Root | unknown | low |
| audit | 110019496666 | /Security | unknown | low |
| log-archive | 122980216815 | /Security | unknown | low |
| platform-prod | 150982215529 | /Platform | production (inferred) | medium |
| workload-prod | 122122642149 | /Workloads | production | high (explicit) |
| workload-dev | 105769365151 | /Workloads | development | high (explicit) |
| sandbox-ma | 161388682021 | /Sandbox | sandbox | high (explicit) |

The Security OU accounts (`audit`, `log-archive`) and the Platform OU account (`platform-prod`) carry no explicit environment tag. Their classification is either absent or inferred. See [`03-environment-structure.md`](03-environment-structure.md) for detail.

---

## Inferred Workloads

| Workload Name | Account | Region | Anchor Resource | Confidence | Internet-Facing (workload record) |
|---|---|---|---|---|---|
| cloudox-demo-atlas-prod | 122122642149 | eu-central-1 | `AWS::ECS::Cluster` `cloudox-demo-atlas-prod` | medium | No* |
| cloudox-demo-atlas-prod-api | 122122642149 | eu-central-1 | `AWS::Lambda::Function` `cloudox-demo-atlas-prod-api` | medium | No* |
| cloudox-demo-atlas-dev | 105769365151 | eu-central-1 | `AWS::ECS::Cluster` `cloudox-demo-atlas-dev` | medium | No |
| cloudox-demo-atlas-dev-api | 105769365151 | eu-central-1 | `AWS::Lambda::Function` `cloudox-demo-atlas-dev-api` | medium | No |
| cloudox-demo-sandbox-scratch | 161388682021 | eu-central-1 | `AWS::Lambda::Function` `cloudox-demo-sandbox-scratch` | low | No |

\* The workload record marks `internet_facing: false`, but the finding `exposure:AWS::ElasticLoadBalancingV2::LoadBalancer:cloudox-demo-atlas-prod-alb` confirms that `cloudox-demo-atlas-prod-alb` — a resource listed under the `cloudox-demo-atlas-prod` workload — is an internet-facing load balancer. The workload-level flag should be validated with the customer.

All workload names follow a `cloudox-demo-*` pattern. Whether this reflects a single application ("atlas"), multiple applications, or a demo/test deployment is not determinable from the discovery data alone.

---

## Internet Exposure Highlights

Nine findings were generated, all in the `exposure` category with high confidence:

- **ALB `cloudox-demo-atlas-prod-alb`** (account 122122642149): scheme is internet-facing.
- **Security groups** `sg-054446d655de1ee7f` (`cloudox-demo-sandbox-sg-permissive`), `sg-06f2b4190bf01d261` (`cloudox-demo-atlas-prod-sg-alb`), and `sg-0459201826f8de5b3` (`cloudox-demo-atlas-prod-sg-edge`): ingress rules allow `0.0.0.0/0` or `::/0`.
- **Five subnets** (`subnet-065f522206524ab12`, `subnet-029e2cceb3d0beff7`, `subnet-016c22941a019a137`, `subnet-013a24d318ed6f3d0`, `subnet-0f64d71c952a7898a`): route tables forward traffic to an Internet Gateway.

The permissive security group `sg-054446d655de1ee7f` is in the sandbox account (161388682021); the ALB and edge security groups are in the production workload account (122122642149). See [`07-findings-and-risks.md`](07-findings-and-risks.md) for the full finding list and [`04-networking.md`](04-networking.md) for network context.

---

## Monitoring and Security Tooling — Observed Presence

The following security and observability resources are present in the graph. Their operational status is not confirmed by the available data:

| Resource Type | Count | Note |
|---|---|---|
| `AWS::CloudTrail::Trail` | 12 | Resource presence confirmed; activation state unknown |
| `AWS::GuardDuty::Detector` | 1 | Resource presence confirmed; detector status unknown |
| `AWS::AccessAnalyzer::Analyzer` | 1 | Resource presence confirmed; analyzer status unknown |
| `AWS::CloudWatch::Alarm` | 7 | Resources present; alarm states unknown |
| `AWS::Logs::LogGroup` | 11 | Log groups present; retention and ingestion not confirmed |

See [`05-security-overview.md`](05-security-overview.md) and [`06-observability.md`](06-observability.md) for detail.

---

## Major Unknowns

- **Environment classification** for accounts 110319895932, 110019496666, and 122980216815 is absent from tags and unresolvable from the current data.
- **Operational status** of CloudTrail (12 trails), GuardDuty (1 detector), and Access Analyzer (1 analyzer) is not confirmed — only resource existence is known.
- **Workload purpose and ownership**: no owner tags, cost-centre tags, or application tags were surfaced in the discovery data for any workload resource.
- **The `cloudox-demo-*` naming pattern** may indicate demo or test infrastructure rather than production workloads. This cannot be determined from the data.
- **Platform account (150982215529)** contains no detected workloads. Its role (shared services, networking hub, CI/CD, etc.) is unknown.
- **No region count** was returned in `graph_stats.region_count` (value: 0). All workload anchors reference `eu-central-1`, but whether other regions are in use is not confirmed.

---

## Recommended Discovery Questions

1. **Workload identity**: Are the `cloudox-demo-atlas-*` resources a live production application or a demonstration/test deployment? Who owns them, and what is the intended audience?
2. **Security tooling activation**: Are CloudTrail, GuardDuty, and Access Analyzer actively enabled and centrally aggregated into the `audit` account? What is the current Security Hub posture?
3. **Platform account purpose**: What services or shared infrastructure does account 150982215529 (`platform-prod`) provide? Is it a networking hub, CI/CD platform, or shared services account?
4. **Environment tagging**: Why do the management, audit, and log-archive accounts lack explicit environment classification tags? Is there a tagging policy in place?
5. **Internet exposure intent**: Is the internet-facing ALB (`cloudox-demo-atlas-prod-alb`) intentionally public? What controls sit in front of it (WAF, CloudFront, etc.)?
6. **Sandbox governance**: The permissive security group `sg-054446d655de1ee7f` (`cloudox-demo-sandbox-sg-permissive`) is in the sandbox account. Are there guardrails (SCPs, permission boundaries) limiting blast radius from sandbox accounts?
7. **Ownership and on-call**: Is there a defined owner or team responsible for each account? No owner or contact tags were detected in the discovery data.