# Findings & Risks

> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

Nine findings were generated across the discovery data. No explicit risk records were generated (`risk_count: 0` in `graph_stats`). The findings below are drawn directly from the `findings` collection and are grouped by type for readability.

---

## Internet Exposure Findings

### 1. Internet-Facing Load Balancer

| Field | Value |
|---|---|
| Finding ID | `exposure:AWS::ElasticLoadBalancingV2::LoadBalancer:cloudox-demo-atlas-prod-alb` |
| Impacted resource | `cloudox-demo-atlas-prod-alb` |
| Account | `122122642149` (workload-prod, classified: production, confidence: high) |
| Category | Exposure |
| Confidence | High |
| Evidence | Load balancer scheme field is `internet-facing` |

The ALB `cloudox-demo-atlas-prod-alb` is directly reachable from the public internet. This resource sits in the production account (`122122642149`) and is associated with the `cloudox-demo-atlas-prod` workload. The workload itself is inferred as `internet_facing: false` — this contradiction should be validated with the customer.

**Follow-up priority:** High — internet-facing ALB in a high-confidence production account warrants confirmation that the exposure is intentional and that listener rules, WAF attachment, and TLS configuration are in place.

**Suggested action:** Confirm with the workload owner whether `cloudox-demo-atlas-prod-alb` is intentionally public. If so, verify that an AWS WAF WebACL is associated with the listener and that HTTPS (port 443) is the only active listener. Check whether the `internet_facing: false` flag on the workload record is a data gap.

---

### 2. Public Subnets (Consolidated)

Five subnets have route tables that forward traffic to an Internet Gateway, classifying them as public subnets.

| Subnet ID | Finding ID | Confidence |
|---|---|---|
| `subnet-065f522206524ab12` | `exposure:AWS::EC2::Subnet:subnet-065f522206524ab12` | High |
| `subnet-029e2cceb3d0beff7` | `exposure:AWS::EC2::Subnet:subnet-029e2cceb3d0beff7` | High |
| `subnet-016c22941a019a137` | `exposure:AWS::EC2::Subnet:subnet-016c22941a019a137` | High |
| `subnet-013a24d318ed6f3d0` | `exposure:AWS::EC2::Subnet:subnet-013a24d318ed6f3d0` | High |
| `subnet-0f64d71c952a7898a` | `exposure:AWS::EC2::Subnet:subnet-0f64d71c952a7898a` | High |

The account and VPC association for each subnet is not present in the filtered discovery data. The graph contains 63 subnets across 15 VPCs in the full graph — account attribution for these five should be confirmed at the workshop.

**Follow-up priority:** Medium — public subnets are a structural fact, not a vulnerability in isolation. Follow-up priority rises if resources with public IP auto-assignment or permissive security groups are placed in these subnets.

**Suggested action:** For each of the five subnet IDs, confirm the owning account and VPC, whether `MapPublicIpOnLaunch` is enabled, and which resource types (EC2, ECS tasks, RDS) are currently placed in them. Cross-reference with the security group findings below.

---

### 3. Permissive Security Groups (Consolidated)

Three security groups have ingress rules permitting traffic from `0.0.0.0/0` or `::/0`.

| Security Group ID | Name | Finding ID | Confidence |
|---|---|---|---|
| `sg-054446d655de1ee7f` | `cloudox-demo-sandbox-sg-permissive` | `exposure:AWS::EC2::SecurityGroup:sg-054446d655de1ee7f` | High |
| `sg-06f2b4190bf01d261` | `cloudox-demo-atlas-prod-sg-alb` | `exposure:AWS::EC2::SecurityGroup:sg-06f2b4190bf01d261` | High |
| `sg-0459201826f8de5b3` | `cloudox-demo-atlas-prod-sg-edge` | `exposure:AWS::EC2::SecurityGroup:sg-0459201826f8de5b3` | High |

**Context by group:**

- **`sg-054446d655de1ee7f` (`cloudox-demo-sandbox-sg-permissive`)** — The name contains "permissive" and "sandbox", suggesting this may be intentional for a sandbox environment (`161388682021`, classified: sandbox, confidence: high). However, the account association is not confirmed in the filtered data and should be validated.

- **`sg-06f2b4190bf01d261` (`cloudox-demo-atlas-prod-sg-alb`)** — Name pattern associates this with the production ALB. An ALB security group accepting `0.0.0.0/0` is expected for a public-facing load balancer, but the permitted ports and protocols are not visible in the provided data.

- **`sg-0459201826f8de5b3` (`cloudox-demo-atlas-prod-sg-edge`)** — Name pattern suggests an edge/perimeter role in the production workload. The specific ports permitted from `0.0.0.0/0` are not present in the discovery data.

**Follow-up priority:**

- `sg-054446d655de1ee7f`: Low follow-up priority if confirmed to be sandbox-only and not attached to production resources — validate account placement and attachment.
- `sg-06f2b4190bf01d261`: Medium — expected for an internet-facing ALB, but permitted ports should be confirmed (HTTP/HTTPS only vs. broader ranges).
- `sg-0459201826f8de5b3`: High follow-up priority — an "edge" security group in production with unrestricted ingress warrants port-level review; the specific ports open to `0.0.0.0/0` are unknown from the provided data.

**Suggested action:** For `sg-06f2b4190bf01d261` and `sg-0459201826f8de5b3`, retrieve the full ingress rule set (ports, protocols) and confirm that only 80/443 are open to `0.0.0.0/0`. For `sg-054446d655de1ee7f`, confirm the owning account and that no production resources are attached to this group.

---

## No Explicit Risk Records

`risk_count: 0` in the graph stats. No explicit risk records were generated by the discovery process. The findings above represent the full set of detected issues.

---

## Assumptions

| # | Assumption | Basis | How to validate |
|---|---|---|---|
| 1 | The five public subnets belong to the production or workload accounts | Naming convention of the ALB and security groups suggests a production VPC context; no account tag is present in the filtered data | Retrieve subnet details from the full graph or confirm at the workshop |
| 2 | `cloudox-demo-sandbox-sg-permissive` is attached to sandbox account `161388682021` | Name contains "sandbox"; a sandbox account exists in the org | Confirm via EC2 DescribeSecurityGroups in account `161388682021` |
| 3 | The `internet_facing: false` flag on the `cloudox-demo-atlas-prod` workload is a data gap, not a factual statement | The workload includes `cloudox-demo-atlas-prod-alb`, which is confirmed internet-facing by a high-confidence finding | Validate workload metadata with the customer and the discovery pipeline |

---

## Unknowns / Gaps

- **Ports and protocols** for the three permissive security groups are not present in the provided discovery data. The finding confirms `0.0.0.0/0` ingress exists but does not specify which ports are open.
- **Subnet-to-account mapping** for the five public subnets is not present in the filtered data. Account and VPC attribution should be confirmed before the workshop.
- **ALB listener configuration** (ports, TLS certificates, WAF association) for `cloudox-demo-atlas-prod-alb` is not present in the discovery data. One listener resource (`AWS::ElasticLoadBalancingV2::Listener: 1`) is recorded in the graph but its configuration is not visible here.
- **Security group attachment** — which EC2, ECS, or RDS resources are currently attached to `sg-0459201826f8de5b3` and `sg-06f2b4190bf01d261` is not determinable from the provided data.

---

## Follow-Up Items for the Workshop

| # | Item | Tied to | Rationale |
|---|---|---|---|
| 1 | Confirm whether `cloudox-demo-atlas-prod-alb` internet exposure is intentional and review listener/WAF configuration | `cloudox-demo-atlas-prod-alb`, account `122122642149` | High-confidence internet-facing ALB in production; WAF and TLS state unknown |
| 2 | Retrieve full ingress rules (ports/protocols) for `sg-0459201826f8de5b3` and `sg-06f2b4190bf01d261` | `sg-0459201826f8de5b3`, `sg-06f2b4190bf01d261` | `0.0.0.0/0` ingress confirmed; permitted ports unknown |
| 3 | Confirm account and resource attachment for `sg-054446d655de1ee7f` | `sg-054446d655de1ee7f` | Name suggests sandbox intent; needs confirmation that no production resources are attached |
| 4 | Map the five public subnets to their owning accounts and VPCs and check `MapPublicIpOnLaunch` | `subnet-065f522206524ab12`, `subnet-029e2cceb3d0beff7`, `subnet-016c22941a019a137`, `subnet-013a24d318ed6f3d0`, `subnet-0f64d71c952a7898a` | Public subnet presence is structural; risk depends on what is placed in them |
| 5 | Resolve the `internet_facing: false` discrepancy on the `cloudox-demo-atlas-prod` workload | Workload `cloudox-demo-atlas-prod`, finding `exposure:AWS::ElasticLoadBalancingV2::LoadBalancer:cloudox-demo-atlas-prod-alb` | Workload metadata contradicts a high-confidence exposure finding |