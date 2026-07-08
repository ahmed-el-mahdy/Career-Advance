# Interview Prep — Technical Deep Dive

> Prepare for Sunday Jul 13 interview. Covers likely questions, system design scenarios, and quick-reference answers.

---

## Quick Reference: Your Numbers

| Metric | Value | One-liner |
|--------|-------|-----------|
| AKS Clusters | 4 | dev, qa, staging, prod |
| Microservices | 7 (.NET 8) + 2 Angular frontends | Full-stack platform |
| HPA Policies | 24 | 6 svc × 4 env, tiered max replicas |
| IaC Drift | 0 | terraform plan = no changes in staging |
| Alert Rules | 4 + 3 availability tests | Proactive, not reactive |
| CI/CD | DRY reusable workflows | 1 template → 9 repos |
| Secrets | 13 migrated to Key Vault | Zero plaintext in K8s |
| Environments | 4 | Kustomize overlays, promoted via branch |

---

## Technical Questions & Answers

### 1. "Design a CI/CD pipeline for microservices"

**Your answer (draw this on whiteboard):**

```
Developer → git push (branch)
    ↓
GitHub Actions trigger (branch-based: dev/qa/staging/prod)
    ↓
Job 1: Build & Test
  - dotnet build + test
  - Docker multi-stage build (SDK → runtime)
  - Tag: {branch}-{short-sha}
    ↓
Job 2: Push to ACR
  - az acr login (managed identity)
  - docker push recoveryacr{env}.azurecr.io/{service}:{tag}
    ↓
Job 3: Deploy to AKS
  - az aks get-credentials
  - kubectl set image deployment/{service} {service}={new-image}
  - kubectl rollout status (verify)
    ↓
Job 4: APIM Import
  - Wait for pod ready
  - az apim api import --specification-url http://{ip}/{service}/swagger/v1/swagger.json
```

**Key points to mention:**
- Branch strategy: dev → qa → staging → prod (merge promotes)
- Environment-scoped secrets (not repo-level)
- Image tag = commit SHA for traceability
- Rollback = kubectl rollout undo

---

### 2. "How would you implement GitOps for 4 environments?"

**Your answer:**

```
Repo structure:
├── base/                    ← shared deployment templates
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/                 ← patches for dev
    ├── qa/                  ← patches for qa
    ├── staging/             ← patches for staging (prod-like)
    └── prod/                ← patches for prod
```

**Promotion flow:**
1. Developer merges to `dev` → CI deploys to dev cluster
2. QA approves → merge dev → qa → CI deploys to qa
3. Stakeholder sign-off → merge qa → staging
4. Staging validated → merge staging → prod

**What differs per overlay:**
- Replicas (dev=1, qa=2, staging/prod=3-4)
- Resource limits (dev=256Mi, prod=2Gi)
- ConfigMap values (endpoints, feature flags)
- Secret references (different Key Vault per env)

---

### 3. "Explain your Terraform state management strategy"

**Your answer:**

- State stored in Azure Storage Account (remote backend)
- One state file per environment (dev.tfstate, qa.tfstate, etc.)
- State locking via Azure Blob lease
- Module-based architecture: shared modules, per-env variable files

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "recovery-rg-terraform"
    storage_account_name = "recoverytfstate"
    container_name       = "tfstate"
    key                  = "prod.tfstate"
  }
}
```

**Drift detection:** Run `terraform plan` in CI on schedule → alert if drift detected.

---

### 4. "How do you handle secrets in Kubernetes?"

**Your answer (the migration story):**

**Before (inherited):** Secrets in ConfigMaps (plaintext, kubectl-readable)

**After (implemented):**
1. Secrets stored in Azure Key Vault (per environment)
2. CSI Secrets Store Driver installed on AKS
3. SecretProviderClass per service → references Key Vault
4. Pod mounts secret as volume (not env var)
5. Auth via Managed Identity (no passwords anywhere)

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: claims-secrets
spec:
  provider: azure
  parameters:
    keyvaultName: "recovery-kv-prod"
    usePodIdentity: "false"
    useVMManagedIdentity: "true"
    userAssignedIdentityID: "<kubelet-identity>"
    objects: |
      array:
        - objectName: mongodb-connection-string
          objectType: secret
        - objectName: redis-connection-string
          objectType: secret
```

---

### 5. "Describe your monitoring/alerting approach"

**Your answer:**

**3 layers:**
1. **Infrastructure** — Prometheus + Grafana (CPU, memory, pod restarts, node health)
2. **Application** — Application Insights APM (request rate, latency, exceptions, dependencies)
3. **Availability** — Synthetic tests pinging critical endpoints every 5 min

**Alert rules (prod):**
| Alert | Condition | Severity |
|-------|-----------|----------|
| High CPU | > 80% for 10 min | Sev 2 |
| High Memory | > 80% for 10 min | Sev 2 |
| High Error Rate | > 5 failed requests / 5 min | Sev 1 |
| High Latency | avg response > 5s / 5 min | Sev 2 |

**Action group:** Email + Teams notification → on-call engineer.

---

### 6. "How would you do zero-downtime deployments?"

**Your answer:**

- **Rolling update** (default): `maxSurge=25%`, `maxUnavailable=25%`
- **Readiness probe**: New pod only receives traffic when `/health` returns 200
- **Liveness probe**: Pod restarted if health check fails
- **PodDisruptionBudget**: Ensures min 1 replica during voluntary disruptions

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
readinessProbe:
  httpGet:
    path: /health
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 5
```

**Rollback:** `kubectl rollout undo deployment/{service}` → instant revert to previous image.

---

### 7. "Explain AKS autoscaling (cluster + pod)"

**Your answer:**

**Two levels:**

1. **Pod-level: HPA (Horizontal Pod Autoscaler)**
   - Scales pod replicas based on CPU/Memory utilization
   - Our config: CPU target 70%, min=1, max varies by env
   - Example: Claims pod CPU hits 75% → HPA adds replica in ~30s

2. **Cluster-level: Cluster Autoscaler**
   - Adds/removes nodes when pods can't be scheduled
   - Our config: min=2, max=5 nodes (prod)
   - Trigger: Pod pending due to insufficient resources → new node in ~3-5 min

**Tiered approach:**
| Environment | HPA max | Cluster nodes |
|-------------|---------|---------------|
| Dev | 2 | 1-2 (B2s) |
| QA | 3 | 1-3 |
| Staging | 4 | 2-4 |
| Prod | 4 | 2-5 (D4ds_v5) |

---

### 8. "How do you handle drift detection in Terraform?"

**Your answer:**

1. **Scheduled `terraform plan`** in CI (weekly or on PR)
2. **Compare**: plan output should show `0 to add, 0 to change, 0 to destroy`
3. **If drift found**: 
   - Minor (tags, descriptions) → apply to align
   - Major (someone changed via portal) → investigate, import or revert
4. **Prevention**: 
   - Lock down Azure portal write access (RBAC reader for devs)
   - All changes must go through PR → terraform plan → apply

---

### 9. "Describe a production incident you resolved"

**Use STAR 1 (Claims OOM) or this one:**

**Situation:** Claims service started returning 503s during batch PDF processing.

**Root cause:** Pod OOM-killed because default memory limit (512Mi) was insufficient for IronPDF rendering.

**Resolution:**
1. Identified via `kubectl describe pod` → `OOMKilled` reason
2. Analyzed memory usage pattern with Prometheus
3. Tuned: increased to 2Gi memory limit, set CPU request to 1 vCPU
4. Added HPA to scale out during batch processing bursts
5. Result: Zero OOM kills since, stable PDF processing

---

### 10. "How would you design a multi-region DR strategy?"

**Your answer (conceptual — shows you think at scale):**

```
Primary: UAE North (Active)
├── AKS Cluster
├── MongoDB (replica set primary)
├── Redis Cache
└── APIM + App Gateway

Secondary: UAE Central (Passive/Warm)
├── AKS Cluster (scaled down, warm standby)
├── MongoDB (replica set secondary, auto-failover)
├── Redis Cache (geo-replication)
└── APIM (synced via Git)

Traffic Manager: DNS-based failover
RPO: ~0 (sync replication)
RTO: 5-15 min (cluster scale-up + DNS switch)
```

---

## Behavioral Questions (STAR Format)

### "Tell me about a time you stabilized a failing platform"

→ Use **STAR 2** (Terraform drift elimination) from STAR-ANSWERS.md

### "How did you handle inheriting undocumented infrastructure?"

**S:** Joined Mosadad, found 4 AKS clusters with no documentation, secrets in plaintext, no IaC alignment.

**T:** Make the platform production-ready and maintainable.

**A:** 
1. Mapped all resources across 4 subscriptions (dev/qa/staging/prod)
2. Documented everything in README + architecture diagrams
3. Aligned Terraform to actual state (import → plan → 0 drift)
4. Implemented Key Vault CSI, monitoring, HPA in systematic phases

**R:** Platform went from "scary to touch" to "confident to deploy" in 6 months. New team members can onboard in days, not weeks.

### "What's your approach to on-call and incident response?"

**Answer:**
1. Alert fires → acknowledge within 5 min
2. Triage: is it affecting users? (check availability tests)
3. Quick mitigation: rollback, scale up, or redirect traffic
4. Root cause analysis after mitigation
5. Post-mortem: document what happened, add missing alert/test
6. Prevention: fix the underlying issue, improve monitoring

---

## Questions to Ask THEM

1. "What does your current deployment pipeline look like?"
2. "How many environments do you manage, and how is promotion handled?"
3. "What's the biggest infrastructure challenge you're facing right now?"
4. "How is on-call structured? What's the incident frequency?"
5. "What does success look like in this role in the first 90 days?"

---

## Day-Before Checklist

- [ ] Review this document (30 min)
- [ ] Practice STAR stories out loud (15 min)
- [ ] Open `IMPACT-SNAPSHOT.md` — memorize the numbers
- [ ] Prepare screen share if asked to demonstrate anything
- [ ] Test audio/video setup
- [ ] Have a glass of water ready
- [ ] Arrive/join 5 minutes early
