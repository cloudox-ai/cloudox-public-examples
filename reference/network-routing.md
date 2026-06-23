> **Demo Report** — AWS account IDs and resource identifiers in this report have been
> replaced with synthetic equivalents for public safety. Architecture, workloads,
> findings, and relationships are based on a real AWS environment.

---

[Reference](./README.md)

# Network Routing Reference

Configured VPC route-table routes (control-plane config). Shows where a route table *directs* traffic for a destination — this is route-level connectivity only, not end-to-end reachability (which also depends on security groups, NACLs, and the target's state).

_Displaying all **39** row(s) (no rows omitted)._

_Sorted by account, then region, then route table (stable, not by risk)._

| Account | Region | Route Table | Source Scope | Destination | Target Type | Target Resource | Route State | Confidence | Evidence |
|---|---|---|---|---|---|---|---|---|---|
| 122980216815 | eu-central-1 | rtb-03044e73437cfeda0 | vpc-02298727f33db8908 (main route table) | 172.31.0.0/16 | local | local | active | Verified | `rtb-03044e73437cfeda0` |
| 122980216815 | eu-central-1 | rtb-03044e73437cfeda0 | vpc-02298727f33db8908 (main route table) | 0.0.0.0/0 | internet gateway | igw-0dd534cb15282cdcb | active | Verified | `rtb-03044e73437cfeda0`, `igw-0dd534cb15282cdcb` |
| 122980216815 | us-east-1 | rtb-0ccd1f020917655dc | vpc-0819c77622c7a1e86 (main route table) | 172.31.0.0/16 | local | local | active | Verified | `rtb-0ccd1f020917655dc` |
| 122980216815 | us-east-1 | rtb-0ccd1f020917655dc | vpc-0819c77622c7a1e86 (main route table) | 0.0.0.0/0 | internet gateway | igw-0cff0d66b4fd90803 | active | Verified | `rtb-0ccd1f020917655dc`, `igw-0cff0d66b4fd90803` |
| 110019496666 | eu-central-1 | rtb-06b3cf16781db0d52 | vpc-0038de9730553c915 (main route table) | 172.31.0.0/16 | local | local | active | Verified | `rtb-06b3cf16781db0d52` |
| 110019496666 | eu-central-1 | rtb-06b3cf16781db0d52 | vpc-0038de9730553c915 (main route table) | 0.0.0.0/0 | internet gateway | igw-0d14f1dd4e54d5906 | active | Verified | `rtb-06b3cf16781db0d52`, `igw-0d14f1dd4e54d5906` |
| 110019496666 | us-east-1 | rtb-0a098578f63800228 | vpc-0563bd6a8386b6eb8 (main route table) | 172.31.0.0/16 | local | local | active | Verified | `rtb-0a098578f63800228` |
| 110019496666 | us-east-1 | rtb-0a098578f63800228 | vpc-0563bd6a8386b6eb8 (main route table) | 0.0.0.0/0 | internet gateway | igw-00ed21b9a0e6596a8 | active | Verified | `rtb-0a098578f63800228`, `igw-00ed21b9a0e6596a8` |
| 105769365151 | eu-central-1 | cloudox-demo-atlas-dev-private-rt (rtb-009215bffcfd9f9a0) | vpc-0a4d44a07f48d7ca0 · 2 subnet(s) | 10.30.0.0/16 | local | local | active | Verified | `rtb-009215bffcfd9f9a0` |
| 105769365151 | eu-central-1 | cloudox-demo-atlas-dev-private-rt (rtb-009215bffcfd9f9a0) | vpc-0a4d44a07f48d7ca0 · 2 subnet(s) | pl-66a5400f | gateway | vpce-0f376e51c370a0f95 | active | Verified | `rtb-009215bffcfd9f9a0`, `vpce-0f376e51c370a0f95` |
| 105769365151 | eu-central-1 | cloudox-demo-atlas-dev-private-rt (rtb-009215bffcfd9f9a0) | vpc-0a4d44a07f48d7ca0 · 2 subnet(s) | pl-6ea54007 | gateway | vpce-0e75224f50dddfb8d | active | Verified | `rtb-009215bffcfd9f9a0`, `vpce-0e75224f50dddfb8d` |
| 105769365151 | eu-central-1 | cloudox-demo-atlas-dev-public-rt (rtb-04732cf316d6c4917) | vpc-0a4d44a07f48d7ca0 · 2 subnet(s) | 10.30.0.0/16 | local | local | active | Verified | `rtb-04732cf316d6c4917` |
| 105769365151 | eu-central-1 | cloudox-demo-atlas-dev-public-rt (rtb-04732cf316d6c4917) | vpc-0a4d44a07f48d7ca0 · 2 subnet(s) | 0.0.0.0/0 | internet gateway | cloudox-demo-atlas-dev-igw (igw-0155958a0a5e9c500) | active | Verified | `rtb-04732cf316d6c4917`, `igw-0155958a0a5e9c500` |
| 105769365151 | eu-central-1 | rtb-04e1da71bbd8c26be | vpc-0a4d44a07f48d7ca0 (main route table) | 10.30.0.0/16 | local | local | active | Verified | `rtb-04e1da71bbd8c26be` |
| 105769365151 | eu-central-1 | rtb-0a43164e123e00eca | vpc-0fdf08606e046467e (main route table) | 172.31.0.0/16 | local | local | active | Verified | `rtb-0a43164e123e00eca` |
| 105769365151 | eu-central-1 | rtb-0a43164e123e00eca | vpc-0fdf08606e046467e (main route table) | 0.0.0.0/0 | internet gateway | igw-014b6ff537b3b21f6 | active | Verified | `rtb-0a43164e123e00eca`, `igw-014b6ff537b3b21f6` |
| 105769365151 | us-east-1 | rtb-07a9d2304da94784c | vpc-0b807fb10d159b43d (main route table) | 172.31.0.0/16 | local | local | active | Verified | `rtb-07a9d2304da94784c` |
| 105769365151 | us-east-1 | rtb-07a9d2304da94784c | vpc-0b807fb10d159b43d (main route table) | 0.0.0.0/0 | internet gateway | igw-0567575921f471548 | active | Verified | `rtb-07a9d2304da94784c`, `igw-0567575921f471548` |
| 122122642149 | eu-central-1 | cloudox-demo-atlas-prod-private-rt (rtb-0fb49391955972271) | vpc-0aaa6d4f2f981e945 · 2 subnet(s) | 10.40.0.0/16 | local | local | active | Verified | `rtb-0fb49391955972271` |
| 122122642149 | eu-central-1 | cloudox-demo-atlas-prod-private-rt (rtb-0fb49391955972271) | vpc-0aaa6d4f2f981e945 · 2 subnet(s) | pl-66a5400f | gateway | vpce-02ddbd8d63a6ab447 | active | Verified | `rtb-0fb49391955972271`, `vpce-02ddbd8d63a6ab447` |
| 122122642149 | eu-central-1 | cloudox-demo-atlas-prod-private-rt (rtb-0fb49391955972271) | vpc-0aaa6d4f2f981e945 · 2 subnet(s) | pl-6ea54007 | gateway | vpce-0232f3af5b06cd3cf | active | Verified | `rtb-0fb49391955972271`, `vpce-0232f3af5b06cd3cf` |
| 122122642149 | eu-central-1 | cloudox-demo-atlas-prod-public-rt (rtb-01181117bc9ca1e3e) | vpc-0aaa6d4f2f981e945 · 2 subnet(s) | 10.40.0.0/16 | local | local | active | Verified | `rtb-01181117bc9ca1e3e` |
| 122122642149 | eu-central-1 | cloudox-demo-atlas-prod-public-rt (rtb-01181117bc9ca1e3e) | vpc-0aaa6d4f2f981e945 · 2 subnet(s) | 0.0.0.0/0 | internet gateway | cloudox-demo-atlas-prod-igw (igw-01dd5625ec1ea7a54) | active | Verified | `rtb-01181117bc9ca1e3e`, `igw-01dd5625ec1ea7a54` |
| 122122642149 | eu-central-1 | rtb-0f15ecd0bd6466997 | vpc-0e0655b83d41e71e6 (main route table) | 172.31.0.0/16 | local | local | active | Verified | `rtb-0f15ecd0bd6466997` |
| 122122642149 | eu-central-1 | rtb-0f15ecd0bd6466997 | vpc-0e0655b83d41e71e6 (main route table) | 0.0.0.0/0 | internet gateway | igw-087b27f6cb0f7a239 | active | Verified | `rtb-0f15ecd0bd6466997`, `igw-087b27f6cb0f7a239` |
| 122122642149 | eu-central-1 | rtb-0a909ccce765ff9cf | vpc-0aaa6d4f2f981e945 (main route table) | 10.40.0.0/16 | local | local | active | Verified | `rtb-0a909ccce765ff9cf` |
| 122122642149 | us-east-1 | rtb-011c9a14b02d60cf5 | vpc-08bb10d6c7cf4425c (main route table) | 172.31.0.0/16 | local | local | active | Verified | `rtb-011c9a14b02d60cf5` |
| 122122642149 | us-east-1 | rtb-011c9a14b02d60cf5 | vpc-08bb10d6c7cf4425c (main route table) | 0.0.0.0/0 | internet gateway | igw-047ff562988a7b858 | active | Verified | `rtb-011c9a14b02d60cf5`, `igw-047ff562988a7b858` |
| 110319895932 | eu-central-1 | rtb-09d2553b02855b85e | vpc-082aa2987b732c239 (main route table) | 172.31.0.0/16 | local | local | active | Verified | `rtb-09d2553b02855b85e` |
| 110319895932 | eu-central-1 | rtb-09d2553b02855b85e | vpc-082aa2987b732c239 (main route table) | 0.0.0.0/0 | internet gateway | igw-075a796b9b2c1d488 | active | Verified | `rtb-09d2553b02855b85e`, `igw-075a796b9b2c1d488` |
| 110319895932 | us-east-1 | rtb-02f0e5b92601604ae | vpc-03887c51c92a0061d (main route table) | 172.31.0.0/16 | local | local | active | Verified | `rtb-02f0e5b92601604ae` |
| 110319895932 | us-east-1 | rtb-02f0e5b92601604ae | vpc-03887c51c92a0061d (main route table) | 0.0.0.0/0 | internet gateway | igw-0c139573cfad88991 | active | Verified | `rtb-02f0e5b92601604ae`, `igw-0c139573cfad88991` |
| 161388682021 | eu-central-1 | cloudox-demo-sandbox-public-rt (rtb-0ef271019f865470e) | vpc-0c97d53850c027a14 · 1 subnet(s) | 10.50.0.0/16 | local | local | active | Verified | `rtb-0ef271019f865470e` |
| 161388682021 | eu-central-1 | cloudox-demo-sandbox-public-rt (rtb-0ef271019f865470e) | vpc-0c97d53850c027a14 · 1 subnet(s) | 0.0.0.0/0 | internet gateway | cloudox-demo-sandbox-igw (igw-08017528fa36ccb4d) | active | Verified | `rtb-0ef271019f865470e`, `igw-08017528fa36ccb4d` |
| 161388682021 | eu-central-1 | rtb-0325323499896dfb2 | vpc-0c97d53850c027a14 (main route table) | 10.50.0.0/16 | local | local | active | Verified | `rtb-0325323499896dfb2` |
| 161388682021 | eu-central-1 | rtb-05510a0575f38a097 | vpc-06bc92d9927a2b9a6 (main route table) | 172.31.0.0/16 | local | local | active | Verified | `rtb-05510a0575f38a097` |
| 161388682021 | eu-central-1 | rtb-05510a0575f38a097 | vpc-06bc92d9927a2b9a6 (main route table) | 0.0.0.0/0 | internet gateway | igw-0df57fd5578ef6371 | active | Verified | `rtb-05510a0575f38a097`, `igw-0df57fd5578ef6371` |
| 161388682021 | us-east-1 | rtb-07680900fbed3780d | vpc-0fd81e106f24b4e85 (main route table) | 172.31.0.0/16 | local | local | active | Verified | `rtb-07680900fbed3780d` |
| 161388682021 | us-east-1 | rtb-07680900fbed3780d | vpc-0fd81e106f24b4e85 (main route table) | 0.0.0.0/0 | internet gateway | igw-0460eb3ac759de200 | active | Verified | `rtb-07680900fbed3780d`, `igw-0460eb3ac759de200` |

**Coverage notes**

- Route-level only: a route shows intended next-hop, not that traffic is permitted (security groups / NACLs) or that the path completes. See the Security Flow and Connectivity references for the rest.
- Transit Gateway route tables are not collected; only VPC route tables (and their Transit Gateway targets) are shown.
