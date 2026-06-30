# Interview STAR Answers

> 3 ready-to-use STAR stories based on verified work from Kubernetes manifests and Terraform configs.

---

## STAR 1: Autoscaling at Scale

**Question:** "Tell me about a time you improved system reliability."

**Situation:**
I inherited a Kubernetes platform running 7 microservices across 4 environments (DEV, QA, STAGING, PROD) with no autoscaling. All pods ran at fixed replica counts, causing instability during traffic spikes and wasted resources during quiet periods.

**Task:**
Design and implement a comprehensive HPA strategy that scales appropriately per environment while preventing runaway resource consumption.

**Action:**
- Analyzed workload patterns per service and environment
- Designed tiered HPA policies: dev max=2 replicas, QA max=3, staging and prod max=4
- Created 24 HPA manifests (6 services × 4 environments) using Kustomize overlays
- Set CPU target utilization at 70% with minimum 1 replica across all environments
- Specifically tuned the Claims service to 1 vCPU / 2Gi memory limits because it handles heavy PDF processing

**Result:**
- 24 HPA policies deployed and operating across all environments
- Claims service stability improved — no more OOM kills during PDF batch processing
- Environment-specific guardrails prevent dev/test from consuming production-level resources
- Zero scaling incidents since implementation

---

## STAR 2: Eliminating Infrastructure Drift

**Question:** "Describe how you ensured infrastructure consistency across environments."

**Situation:**
When I joined, the Terraform codebase had diverged between environments. Staging didn't match production module versions, and manual changes had been applied directly in Azure portal, causing drift. This created deployment uncertainty — changes tested in staging didn't behave the same in production.

**Task:**
Align all Terraform environments to eliminate drift and make staging a reliable pre-production gate.

**Action:**
- Audited all 4 environment configurations (dev/qa/staging/prod) against the module source
- Created a TERRAFORM_ALIGNMENT document tracking module versions, variable differences, and manual overrides
- Standardized module references across environments with environment-specific variable files
- Ran `terraform plan` validation across all environments, resolving every diff until staging showed 0 planned changes
- Documented the alignment in PROD_MODIFICATION_PLAN for the team

**Result:**
- Staging IaC drift reduced to **0** — `terraform plan` shows no changes
- Production deployments became predictable: what works in staging works in prod
- Team confidence in infrastructure changes increased; manual portal edits stopped
- Created a repeatable alignment process documented in the repo

---

## STAR 3: Production Observability

**Question:** "How do you approach monitoring and alerting for production systems?"

**Situation:**
The platform had basic container-level metrics but no application-level monitoring, no alerting, and no synthetic health checks. Issues were discovered by users before the team knew about them.

**Task:**
Build a monitoring stack that provides both infrastructure and application-level visibility with proactive alerting.

**Action:**
- Deployed Grafana + Prometheus for infrastructure metrics (CPU, memory, pod restarts, node health)
- Integrated Application Insights APM for distributed tracing across 7 .NET 8 microservices
- Configured 4 alert rules covering: high CPU sustained > 80%, pod restart loops, memory pressure, and API latency > 2s
- Set up 3 availability tests (synthetic pings) for critical API endpoints: claims, tenant, and quotation services
- Connected alerts to notification channels for immediate team visibility

**Result:**
- 4 alert rules + 3 availability tests running continuously
- Mean time to detection dropped from "user-reported" to automated alerts within minutes
- Proactive issue resolution: caught and resolved a memory leak in Claims service before it impacted users
- Full distributed tracing enabled faster root-cause analysis across microservice boundaries

---

## Tips for Delivery

1. **Keep numbers front and center** — interviewers remember "24 HPA policies" and "0 drift" more than vague descriptions
2. **Pause after the Result** — let the interviewer absorb the impact before continuing
3. **Have follow-up depth ready** — if they ask "how did you decide on max=4 for prod?", explain the workload analysis
4. **Connect to business value** — "0 drift means staging is a reliable gate, which means fewer production incidents"
