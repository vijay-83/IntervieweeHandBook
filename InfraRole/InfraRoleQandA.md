# Senior Engineer – Infra-as-Code (Woolworths Group India) – Interview Q&A Pack

Sections: 1 Terraform Core · 2 Terraform Enterprise/HCP · 3 Azure Landing Zones · 4 GCP Landing Zones · 5 Policy as Code · 6 Module Registry · 7 Tenancy & FinOps · 8 IAM & Networking · 9 Security & Patching · 10 Migration · 11 SRE/RCA · 12 Scenario-Based · 13 Agile/Squad · 14 Leadership & Behavioral · 15 Rapid-Fire · 16 Questions to Ask Them

---

## 1. Terraform Core

**Q1. How does Terraform state work, and how do you manage it at scale?**
State maps config resources to real-world objects and stores metadata/dependencies. At scale: remote backend (HCP/TFE, Azure Storage, GCS) with locking and encryption, one state per workspace/blast-radius boundary, no secrets in plain outputs, restricted access, versioned state with rollback, and `terraform_remote_state` or data sources/outputs for cross-stack sharing.

**Q2. `count` vs `for_each`?**
`count` indexes by position; removing a middle item shifts and recreates resources. `for_each` keys by a stable string/map key, so changes are isolated. Prefer `for_each` for anything not a simple on/off toggle (`count = var.enabled ? 1 : 0`).

**Q3. How do you handle drift?**
Scheduled speculative plans / health assessments (HCP drift detection), alert on non-empty plans, decide per case: reconcile via apply, or `import`/update code if the change was legitimate. Prevent drift with policy, restricted portal access and break-glass processes.

**Q4. Explain `terraform import`, `import` blocks and `moved` blocks.**
`import` brings existing resources into state. Config-driven `import` blocks (1.5+) are reviewable in a plan and can generate config (`-generate-config-out`). `moved` blocks refactor addresses (rename, module move) without destroy/recreate.

**Q5. What are lifecycle meta-arguments?**
`create_before_destroy`, `prevent_destroy`, `ignore_changes`, `replace_triggered_by`, and `precondition`/`postcondition`. Use sparingly; `ignore_changes` can hide drift.

**Q6. How do you manage secrets in Terraform?**
Never hardcode. Use Key Vault / Secret Manager via data sources or, better, ephemeral resources/write-only attributes (1.10+/1.11+) so values stay out of state; workload identity/OIDC for provider auth; mark variables `sensitive`; remember state still may contain secrets, so protect it.

**Q7. How do you structure a large Terraform codebase?**
Layered: foundation (org/management groups, landing zones) → platform (network, shared services) → workloads (tenant stacks). Reusable versioned modules, thin root modules, environment config via variables/workspaces/tfvars, separate state per layer.

**Q8. Workspaces (CLI) vs HCP/TFE workspaces vs directories per environment?**
CLI workspaces share one config/backend with separate state – fine for simple cases but poor isolation. HCP/TFE workspaces are full units with their own variables, state, RBAC and runs. For prod/non-prod isolation, separate workspaces/backends/accounts are safer than CLI workspaces.

**Q9. What is the dependency graph? When do you use `depends_on`?**
Terraform builds a DAG from references. Use `depends_on` only for hidden dependencies (e.g., IAM propagation); overuse causes needless replacements/deferred data reads.

**Q10. How do you test Terraform?**
`terraform validate`, `fmt`, `tflint`, `checkov`/`trivy`, native `terraform test` (plan/apply modes with mocks), Terratest for integration, ephemeral sandbox subscriptions/projects, and policy tests in CI.

**Q11. How do you upgrade providers/Terraform versions safely?**
Pin with `required_providers` version constraints and commit `.terraform.lock.hcl`; upgrade in lower environments first; read changelogs; run plans for all workspaces to detect diffs; stage rollout via a version-bump pipeline (Renovate/Dependabot).

**Q12. How do you handle a corrupted or locked state?**
Locked: verify no active run, then `force-unlock`. Corrupted: restore a previous state version from backend versioning, `state pull/push` carefully, `state rm`/`import` to reconcile; always back up first.

**Q13. Dynamic blocks, `for` expressions, `locals` — when to use?**
Dynamic blocks for variable-length nested blocks; `for` expressions to transform collections; locals for derived values. Avoid over-abstraction that harms readability.

**Q14. Terraform vs OpenTofu?**
OpenTofu is the open-source (Linux Foundation) fork post-BSL license change; largely compatible with some divergent features (state encryption, early variable evaluation). Decision factors: licensing, vendor support, ecosystem tooling, enterprise platform integration.

---

## 2. Terraform Enterprise / HCP Terraform Platform

**Q15. How would you design workspace governance for hundreds of teams?**
Organization → projects per business unit/product; workspaces per app/env/region; RBAC with teams mapped to IdP groups (SSO/SCIM); variable sets for shared config; run tasks and policy sets scoped by project; naming/tagging conventions; automated workspace creation via `tfe` provider (workspace-as-code).

**Q16. How do you manage the platform itself with code?**
Use the `tfe` provider to define organizations, projects, teams, workspaces, variable sets, policy sets and registry modules in a bootstrap repo; keep that bootstrap in its own protected workspace.

**Q17. Run workflows: VCS-driven, CLI-driven, API-driven?**
VCS: PR triggers speculative plans, merge triggers apply. CLI: dev/testing. API-driven: platform automation (tenancy vending, pipelines). Choose by control needs; enforce VCS-driven for production.

**Q18. How do agents work and why use them?**
HCP Terraform agents run in your network to reach private resources (private AKS/GKE, on-prem, private endpoints) without opening inbound access. Scale pools, secure with least-privilege identities, patch regularly, and isolate by environment.

**Q19. How do you authenticate TFE/HCP to Azure and GCP securely?**
Dynamic provider credentials using OIDC/workload identity federation: Azure federated credentials on an app registration/managed identity; GCP Workload Identity Federation. No long-lived keys; scope per workspace/project.

**Q20. How do you scale and keep a self-hosted TFE reliable?**
Active/active on Kubernetes, external services (managed PostgreSQL, object storage, Redis), backups and DR tested, monitoring of queue depth/run times, scale agents, upgrade cadence with staging, capacity limits per project.

**Q21. How do you do cost/quota/time management of runs?**
Concurrency limits, run timeouts, auto-destroy for ephemeral workspaces, run triggers to avoid cascades, cost estimation + policy on cost thresholds.

**Q22. What are run tasks and how would you use them?**
Third-party integrations at pre-plan/post-plan/pre-apply stages (security scanners, cost tools, ITSM approval). Use to enforce scans (Wiz, Snyk, Checkov), Infracost and change-management checks.

---

## 3. Azure Landing Zones

**Q23. What is an Azure Landing Zone and its key design areas?**
A scalable, secure, governed environment aligned to Cloud Adoption Framework: management group hierarchy, subscription organization, identity & access, network topology, security, management/monitoring, governance (policy), billing, platform automation.

**Q24. Describe a typical management group hierarchy.**
Tenant Root → Org root → Platform (Identity, Connectivity, Management) and Landing Zones (Corp, Online) → Sandbox and Decommissioned. Policies and RBAC assigned at MG level inherit downward.

**Q25. Hub-and-spoke vs Virtual WAN?**
Hub-and-spoke: more control, custom NVAs/firewalls, you manage routing. vWAN: managed global transit, simpler at multi-region/branch scale, some routing limitations. Choose by scale, regions, SD-WAN needs and operational skill.

**Q26. How do you implement subscription vending?**
Automated pipeline taking inputs (owner, cost centre, env, network size) that creates subscription, places it in an MG, applies RBAC, budgets, tags, networking peering, diagnostics, and policy — via Terraform (e.g., Azure Verified Modules `lz-vending`) triggered from a request/API.

**Q27. Azure Verified Modules (AVM) / CAF Terraform modules — would you use them?**
Yes as a baseline, wrapped/pinned to org standards; reduces effort and aligns with Microsoft guidance. Wrap when extra guardrails or interface simplification is needed; track upstream upgrades.

**Q28. Azure Policy effects and how do you roll out safely?**
Audit, Deny, Modify, DeployIfNotExists, AuditIfNotExists, Disabled. Roll out Audit-first, review compliance, exemptions with expiry, then enforce; use initiatives; remediation tasks for existing resources.

**Q29. Azure RBAC vs Entra ID roles vs PIM?**
RBAC controls Azure resources; Entra roles control directory. PIM provides JIT elevation with approval/MFA/audit. Use groups, custom roles only when needed, no standing Owner.

**Q30. Private networking for PaaS?**
Private Endpoints + Private DNS zones (centralized in hub), disable public network access, service endpoints only when private link is unavailable, Azure Firewall egress control, DNS forwarding for hybrid.

---

## 4. GCP Landing Zones

**Q31. Describe a GCP landing zone.**
Organization → folders (by env/BU) → projects; Cloud Identity/Workspace groups; org policies; shared VPC host/service projects; centralized logging sink and security (SCC); billing; Terraform-driven bootstrap (e.g., Cloud Foundation Toolkit/Fabric FAST).

**Q32. Shared VPC vs VPC peering?**
Shared VPC: central network project with subnets shared to service projects; central control, IAM-based. Peering: non-transitive, separate admin domains, CIDR limits. Prefer Shared VPC for enterprise with central network team.

**Q33. What are Organization Policies and examples?**
Constraints applied at org/folder/project: restrict resource locations, disable service account key creation, require OS Login, restrict public IPs, uniform bucket access, restrict VPC peering/external sharing.

**Q34. GCP IAM best practices?**
Groups not users, predefined/custom roles least-privilege, no primitive roles, IAM Conditions, deny policies, workload identity (no SA keys), service account impersonation, Policy Intelligence/Recommender to prune.

**Q35. How do you set up centralized logging and monitoring?**
Aggregated org-level log sinks to BigQuery/Cloud Storage/Pub-Sub (SIEM), log buckets with retention, Cloud Monitoring scoping projects, alert policies as code.

**Q36. Project factory pattern?**
Terraform module creating projects with APIs enabled, labels, billing link, shared VPC attachment, default service accounts removed, budgets and IAM baseline — invoked per tenant request.

---

## 5. Policy as Code (OPA / Sentinel)

**Q37. Sentinel vs OPA?**
Sentinel is HashiCorp-native, tightly integrated with TFE/HCP (enforcement levels: advisory, soft-mandatory, hard-mandatory). OPA (Rego) is open, general-purpose, usable across Terraform plans, Kubernetes (Gatekeeper), CI; integrates in HCP via policy sets. Choose Sentinel for tight TFC integration, OPA for cross-platform consistency.

**Q38. Write an example policy: deny public storage / require tags.**
*Sentinel (concept):* iterate `tfplan/v2` resource_changes for `azurerm_storage_account`, assert `public_network_access_enabled == false`, and `tags` contains required keys.
*OPA/Rego (concept):*
```rego
deny[msg] {
  rc := input.resource_changes[_]
  rc.type == "azurerm_storage_account"
  rc.change.after.public_network_access_enabled == true
  msg := sprintf("%s must not allow public access", [rc.address])
}
```

**Q39. How do you manage the policy lifecycle?**
Policies in Git, PR-reviewed with unit tests (`sentinel test`, `opa test`), versioned, staged rollout (advisory → soft → hard), exception process with expiry, metrics on violations, owners per policy, and regular review against threat model/CIS benchmarks.

**Q40. How do you avoid policy blocking delivery?**
Fast clear error messages with remediation links, advisory mode first, self-service dry-run, paved-road modules already compliant, exception SLA, and feedback loop with squads.

**Q41. Policy at multiple layers?**
Shift-left (pre-commit/IDE/CI scans), provisioning gate (Sentinel/OPA), and runtime (Azure Policy/Org Policy/SCC/Defender). Defence in depth — each layer catches different things.

**Q42. What data do policies evaluate?**
Plan JSON (`tfplan`), configuration, state, and run metadata; cost estimates; mocks for testing.

---

## 6. Central Module Registry

**Q43. How do you design a good Terraform module?**
Single responsibility, opinionated secure defaults, minimal required inputs, validated variables, clear outputs, no provider config inside, docs (terraform-docs), examples, semantic versioning, tests, changelog.

**Q44. How do you version and publish modules?**
Git tags in SemVer (`v1.2.0`) → private registry (HCP/TFE) via VCS integration or API; breaking changes = major; deprecate with notice; consumers pin `~>` ranges.

**Q45. How do you drive adoption and avoid fork sprawl?**
Great docs and examples, inner-source contribution model, backwards compatibility, office hours, scorecards, policy requiring registry modules for key resource types, no-code provisioning for simple cases.

**Q46. How do you roll a security fix to 200 consumers?**
Publish patch version; Renovate/Dependabot PRs to consumers; dashboard of module versions in use; policy blocking vulnerable versions after grace period; communicate; track burn-down.

**Q47. Composition: module of modules?**
Prefer flat, composable modules over deep nesting; use facade/"pattern" modules for landing zones with well-defined interfaces.

**Q48. Module testing strategy?**
Unit via `terraform test` with mocks, integration deployment into sandbox with Terratest, security scan in CI, example directories used as test fixtures.

---

## 7. Tenancy Lifecycle & FinOps

**Q49. Design a tenant vending/lifecycle process.**
Request (portal/ServiceNow/API) → approval → pipeline creates subscription/project, MG/folder placement, network, IAM groups, policies, tagging, budgets, monitoring, TFE workspace/project, repo template → handover; lifecycle: change, suspend, decommission with data retention and cleanup.

**Q50. Tagging/labelling strategy?**
Mandatory keys: cost-centre, owner, application, environment, data-classification, lifecycle/expiry. Enforce via Azure Policy `Modify`/Deny, Org Policy/labels, Sentinel checks; inherit tags from subscription/RG.

**Q51. How do you control cloud cost with code?**
Budgets + alerts per tenant, Infracost in PRs, policy on SKU/region allowlists, auto-shutdown for non-prod, rightsizing and reservations/CUDs, orphan resource cleanup automation, showback/chargeback reporting.

**Q52. How do you decommission environments safely?**
Expiry tag → notification → soft-disable → backup/snapshot → destroy via Terraform → verify cost reduction → archive audit trail. Guard with `prevent_destroy` on critical data until approved.

**Q53. Which FinOps KPIs?**
Tagging compliance %, unallocated spend, cost per tenant/unit, commitment coverage/utilisation, waste %, forecast accuracy.

---

## 8. IAM & Networking Guardrails

**Q54. Global IAM guardrails — what do you enforce?**
No standing privileged access (PIM/JIT), MFA/conditional access, group-based RBAC, deny dangerous actions, block SA key creation, restricted role assignments, break-glass accounts monitored, regular access reviews.

**Q55. Baseline network design principles?**
Non-overlapping CIDR plan (IPAM), segmentation, central egress via firewall, private connectivity, DNS strategy, NSG/firewall-rule-as-code, DDoS protection, flow logs, hybrid connectivity redundancy.

**Q56. How do you do CIDR/IPAM automation for tenants?**
Central IPAM (Azure IPAM, Infoblox, or a Terraform-managed allocator) that hands out non-overlapping ranges during vending; allocation stored and validated.

---

## 9. Secure Engineering & Patch/Vulnerability Management

**Q57. How do you harden IaC pipelines?**
Least-privilege OIDC identities, protected branches, mandatory review, signed commits, scan IaC (Checkov/Trivy/tfsec), secret scanning, pinned/verified providers & modules, isolated runners/agents, audit logs, supply-chain controls (SBOM, provenance).

**Q58. Describe a rapid vulnerability remediation campaign you led (log4j-style).**
Structure: (1) triage/scope via asset inventory & scanners, (2) prioritise by exploitability/exposure, (3) mitigation (WAF rule, config) while patching, (4) automate patch via IaC/image pipeline, (5) staged rollout canary→prod, (6) verify with rescan, (7) track metrics (MTTR, coverage), (8) post-incident improvements. Tailor with your own real example and numbers.

**Q59. Patch management for VMs and containers using code?**
Golden images (Packer) rebuilt on schedule/CVE trigger, immutable replacement via scale sets/MIGs, Azure Update Manager / OS Config for patch windows, container base-image rebuild pipelines, Kubernetes node image auto-upgrade, maintenance windows.

**Q60. CIS benchmarks & compliance as code?**
Map controls to Azure Policy/Org Policy/Sentinel; continuous assessment (Defender for Cloud, SCC); evidence collection automated; exceptions logged.

**Q61. How do you secure Terraform state?**
Encrypted at rest with CMK, private access, RBAC, audit logging, versioning, no broad read access, avoid secrets in state, restrict `terraform show`/state API.

**Q62. Supply-chain risk with public modules/providers?**
Mirror/approve through private registry, pin versions and checksums, review code, scan, provider allowlist (network mirror), monitor advisories.

---

## 10. Legacy to Modern Migration

**Q63. How do you bring manually created infrastructure under Terraform?**
Discover (inventory/Azure Resource Graph/Asset Inventory), generate config (`import` blocks + `-generate-config-out`, aztfexport, `gcloud` export/Terraformer), refactor into modules, import in stages by dependency order, validate zero-diff plan, lock down manual changes, then iterate.

**Q64. Migration strategy for a legacy estate with minimal downtime?**
Assess → prioritise by risk/value → build target landing zone → pilot → migrate waves with rollback plan → parallel run/cutover → decommission → measure. Use blue-green/canary, data sync, runbooks, communication.

**Q65. Migrating from Terraform OSS state/CI to HCP/TFE?**
Move state via backend migration (`terraform init -migrate-state`), create workspaces, configure variables/OIDC, connect VCS, enforce policies in advisory, parallel verify with no-op plans, cut over pipelines, retire old backends.

**Q66. How do you convince teams to give up manual access?**
Make the paved road faster and easier than the manual path, provide self-service, read-only portal access, break-glass with audit, then progressively restrict with data on benefits.

---

## 11. Incident Ownership & SRE

**Q67. Walk through an RCA you led.**
Use STAR: timeline, impact, detection, mitigation, root cause (5 Whys/fault tree), contributing factors, permanent fixes (code/automation/guardrails), action tracking, blameless review. Example theme: a bad module upgrade caused mass replacement → fixed with plan-review policy on destroys, staged rollouts and `prevent_destroy`.

**Q68. How do you reduce toil?**
Measure toil (tickets, manual steps), rank by frequency × time, automate via self-service/IaC/event-driven runbooks, set toil budget, verify elimination.

**Q69. SLIs/SLOs for a platform team?**
Provisioning success rate, lead time for tenant vending, plan/apply duration, policy false-positive rate, drift %, module adoption, change failure rate, MTTR.

**Q70. A Terraform apply failed halfway in prod. What do you do?**
Stop further runs/lock the workspace, assess state vs reality via plan, identify partial resources, fix cause (quota, permission, API error), re-apply idempotently or targeted apply only if necessary, document; avoid destructive manual edits; post-incident review.

**Q71. Incident severity & comms?**
Define severities, on-call roles (incident commander, comms, ops), status updates cadence, escalation, handover, post-incident review within a set time.

**Q72. Observability for IaC platform?**
Metrics on queue time, failures, drift, agent health; alerts to on-call; dashboards; audit logs forwarded to SIEM.

---

## 12. Scenario-Based Questions

**S1. A squad needs a new Azure subscription with private networking, a Postgres DB and cost tagging in 1 day. Design it.**
Vending pipeline → subscription in Corp MG → spoke VNet via IPAM, peering/vWAN connection, private DNS; Postgres Flexible Server module with private endpoint, CMK, diagnostics; policy-enforced tags/budget; TFE workspace + repo template; output handover docs.

**S2. A policy you introduced is blocking 40 pipelines on Monday morning. What do you do?**
Switch to advisory/soft-mandatory immediately (rollback), communicate, analyse false positives, fix and test policy, re-introduce gradually, improve rollout process (canary scope, preview mode).

**S3. A developer used `terraform apply -target` and manually changed prod. Drift detected. How do you handle?**
Review plan, decide reconcile direction, restrict permissions (only the platform identity can write), remove local apply capability, add drift alerts, coach, update process.

**S4. Two teams want conflicting changes to a shared module.**
Review via inner-source RFC, backwards-compatible options with feature flags, versioning (major if breaking), allow temporary pinning, document, arbitrate with metrics/standards.

**S5. Cloud bill spiked 40% in one tenant overnight.**
Check cost analysis by resource/tag, identify change (deployment, runaway autoscale, data egress), contain (scale down/budget action), fix in code, add guardrails (SKU policy, quotas, alerts), report.

**S6. You must manage 500 workspaces needing a provider version upgrade.**
Central version policy, automated PRs via bot, group workspaces by risk, run speculative plans in bulk via API, fix diffs, staged apply, track dashboard.

**S7. Prod landing zone design for two clouds with shared governance.**
Common principles (OPA-based policies, tagging, naming, identity federation), cloud-specific implementations, one orchestration platform (TFE), shared module standards, unified logging/SIEM and FinOps view.

**S8. Security reports a critical CVE in a base image used by 300 VMs.**
Rebuild golden image, publish, trigger rolling replacement through IaC with maintenance windows, canary first, verify via scanner, track completion, set policy to block old image IDs.

**S9. TFE queue is backed up with long waits.**
Check concurrency, agent capacity, large monolithic workspaces; scale agents, split state, optimise provider calls (`-parallelism`, refresh settings), stagger scheduled runs, add monitoring.

**S10. You discover secrets in state files.**
Rotate secrets immediately, restrict state access, refactor to ephemeral/Key Vault references, purge state history if needed, add scanning, review who accessed.

---

## 13. Agile / Squad-Based Delivery

**Q73. How do you work with Product Owners and Scrum Masters as a platform engineer?**
Treat platform as a product: backlog with user-facing features, roadmap, user research with squads, refined stories with acceptance criteria, demos, shared metrics.

**Q74. How do you balance feature work vs operational work?**
Capacity allocation (e.g., 60/20/20 features/toil-reduction/tech-debt), visible KPIs, interrupt rota, SLO-driven prioritisation.

**Q75. Definition of Done for an IaC change?**
Peer-reviewed, tested, scanned, policy-passing plan, documented, versioned, monitored, runbook updated, applied via pipeline.

**Q76. How do you estimate infrastructure work?**
Break into slices, spikes for unknowns, use historical cycle time, include testing/security/review, communicate uncertainty.

---

## 14. Leadership & Behavioral

**Q77. How do you set technical standards without being a bottleneck?**
Document ADRs, automate enforcement (linters, policy), create paved roads, delegate via guilds/champions, review by exception.

**Q78. Tell me about mentoring a junior engineer.**
Situation, goals set, pairing, gradual ownership, feedback, result (promotion/independence). Use a real story.

**Q79. Describe a time you disagreed with an architectural decision.**
Present data, understand constraints, propose alternatives/PoC, disagree-and-commit if overruled, follow up on outcomes.

**Q80. Tell me about a major failure.**
Own it, impact, immediate action, root cause, lasting fix, what you changed in your practice.

**Q81. Why Woolworths Group India, and why this role?**
Join an early team building a new capability centre, chance to shape standards, scale of a major retailer, multi-cloud IaC ownership, mentoring culture; link to the purpose "a little good to everyone, every day".

**Q82. How do you stay current?**
HashiConf/Google Next/Ignite, release notes, community modules, side labs, blogs, internal tech talks.

**Q83. First 90 days in this role?**
30: learn estate, tooling, pain points, meet squads; 60: quick wins (policy gaps, module backlog, drift report), define roadmap/metrics; 90: deliver first improvements (e.g., vending automation, policy lifecycle), share standards.

**Q84. How do you influence stakeholders who resist change?**
Understand motives, show value with data/pilot, reduce friction, involve early, celebrate wins.

---

## 15. Rapid-Fire (one-liners)

1. What file stores provider hashes? → `.terraform.lock.hcl`
2. Command to show plan without applying? → `terraform plan`
3. Mark a resource for recreation? → `terraform apply -replace=ADDR` (old: `taint`)
4. Remove from state without destroy? → `terraform state rm`
5. Sentinel enforcement levels? → advisory, soft-mandatory, hard-mandatory
6. Terraform's default parallelism? → 10
7. Which Azure policy effect auto-fixes tags? → Modify
8. GCP policy to block SA keys? → `iam.disableServiceAccountKeyCreation`
9. Azure top-level container for subscriptions governance? → Management Groups
10. Shared VPC project types? → Host and service projects
11. Where do you store OPA policies for TFE? → Policy sets (VCS-linked) with OPA runtime
12. `sensitive = true` hides state values? → No, only from CLI output; state still contains it
13. Immutable infrastructure example? → Replace VMs from new golden image rather than patch in place
14. What is a blast radius? → Scope of impact if a change/failure occurs
15. FinOps three phases? → Inform, Optimise, Operate
16. Module versioning scheme? → Semantic Versioning
17. Dynamic credentials method? → OIDC / workload identity federation
18. How to prevent destroy of critical resource? → `lifecycle { prevent_destroy = true }` plus policy
19. Terraform refresh-only? → `terraform apply -refresh-only` updates state to match reality
20. Alternative IaC tools to know? → Bicep, Pulumi, Crossplane, OpenTofu, Config Connector

---

## 16. Questions You Should Ask the Interviewer

- What does the current Terraform/TFE estate look like (workspaces, versions, self-hosted vs HCP)?
- What's the split of Azure vs GCP workloads today and the target?
- How mature are policies and the module registry; what is the biggest gap?
- How is tenancy provisioned today and what's the target lead time?
- How will the India team collaborate with the Australia team (time zones, ownership boundaries)?
- What does success look like at 6 and 12 months?
- What is the on-call expectation?
- What are the career paths and training support for this role?

---

## Prep Tips
- Prepare 3–4 STAR stories: migration, RCA, vulnerability campaign, standard-setting/mentoring.
- Keep numbers ready: workspaces managed, modules published, MTTR improvements, cost savings, provisioning time reduced.
- Be ready to whiteboard: landing zone hierarchy, tenant vending flow, policy-as-code pipeline.
- Know both clouds: if stronger in one, show a concrete mapping to the other (Azure MG ↔ GCP folders, Azure Policy ↔ Org Policy, PIM ↔ IAM Conditions/PAM).
