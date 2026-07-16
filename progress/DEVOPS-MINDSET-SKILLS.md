# DevOps Engineering Mindset & Methodology Skills

> **Author**: Ahmed ElMahdy  
> **Built from**: Real production experience at Mosadad Recovery Claims Platform  
> **Purpose**: Reusable thinking frameworks, troubleshooting patterns, and decision-making methodology  
> **Last Updated**: July 16, 2026

---

## 1. Root Cause Analysis Framework

### The 5-Layer Diagnostic Model

When any issue is reported, systematically check these layers (top-down):

```
Layer 5: USER/CLIENT → What they see (404, timeout, error message)
Layer 4: GATEWAY/APIM → Routing, path matching, policies, subscription
Layer 3: NETWORKING → Ingress, DNS, NSG rules, IP/port connectivity  
Layer 2: APPLICATION → Pod health, env vars, config, code bugs
Layer 1: DATA → Database connectivity, auth, stale creds, query perf
```

**Rule**: Never assume — verify each layer before jumping to conclusions.

### Applied Examples

| Symptom | Initial Assumption | Actual Root Cause | Layer |
|---------|-------------------|-------------------|-------|
| "404 Not Found" on quotation | Backend is down | APIM path `api/quotations` (plural) vs client calls `api/quotation` | Layer 4 |
| "SCRAM-SHA-1 auth failed" on prod | Password wrong | K8s CSI secret stale — KV updated but secret not synced | Layer 1 |
| "Request timeout" on staging IntHub | MongoDB slow | PDF download (1-3s) + blob upload happening on first view | Layer 2 |
| "Workflow stuck 15 mins" | GitHub Actions issue | `workflow-trigger-guard.yaml` Python validation hanging | Layer 2 |

---

## 2. Change Impact Prediction (Blast Radius Thinking)

### Before Any Change, Ask:

1. **What depends on this?** (upstream consumers)
2. **What does this depend on?** (downstream dependencies)
3. **What environments does this affect?** (dev/qa/staging/prod isolation)
4. **Is this reversible?** (rollback plan)
5. **What's the worst case?** (blast radius)

### Decision Matrix

| Risk Level | Action | Example |
|-----------|--------|---------|
| **Low** (reversible, single env) | Do it | Patch configmap, restart pod |
| **Medium** (multi-env, reversible) | Do with `[skip ci]`, verify | Push workflow to all branches |
| **High** (prod-touching, hard to reverse) | Ask first, plan rollback | Delete K8s secret, change APIM serviceUrl |
| **Critical** (data-affecting, irreversible) | Get approval, backup first | MongoDB schema change, prod DB migration |

### Isolation Patterns Used

- **Environment variables per env**: GitHub Environments scope vars/secrets per branch
- **Concurrency groups**: `{service}-{env}` prevents parallel deploys to same env
- **Validation guardrails**: Fail fast if APIM name doesn't match target environment
- **`[skip ci]`**: Push config changes without triggering deployment

---

## 3. Troubleshooting Methodology (OODA Loop)

### Observe → Orient → Decide → Act

```
OBSERVE:  What are the symptoms? (logs, error codes, response times)
ORIENT:   What changed? (recent deploy, config change, infrastructure)
DECIDE:   What's the cheapest/safest verification step?
ACT:      Apply the fix with minimal blast radius
VERIFY:   Confirm the fix, check for secondary effects
```

### Troubleshooting Commands Toolkit

| Need | Command |
|------|---------|
| Pod status | `kubectl get pods -n {ns} -l app={svc} -o wide` |
| Pod logs | `kubectl logs -l app={svc} -n {ns} --tail=50` |
| Exec into pod | `kubectl exec -n {ns} {pod} -- env \| grep {key}` |
| Test TCP | `nc -zv -w5 {ip} {port}` |
| Test HTTP | `curl -s -o /dev/null -w 'HTTP %{http_code}' {url}` |
| MongoDB auth | `mongosh 'mongodb://{user}:{pass}@{ip}:27017/{db}?authSource=admin' --eval 'db.runCommand({ping:1})'` |
| APIM check | `az apim api list --resource-group {rg} --service-name {apim} -o table` |
| Secret check | `kubectl get secret {name} -n {ns} -o jsonpath='{.data.{key}}' \| base64 -d` |
| ConfigMap check | `kubectl get cm {name} -n {ns} -o jsonpath='{.data}'` |
| HPA status | `kubectl get hpa -n {ns}` |

---

## 4. CI/CD Workflow Design Principles

### Golden Standard Pattern

1. **One workflow file, all environments** — branch determines target env
2. **Environment-scoped secrets/vars** — GitHub Environments isolate config
3. **Guardrails first** — fail fast before expensive operations
4. **Validate Azure context** — subscription + tenant check before any deployment
5. **Cross-env contamination check** — validate APIM name contains expected env keyword
6. **Retry with backoff** — 3 attempts for transient failures (ACR, APIM import)
7. **Post-deploy verification** — rollout status, image check, endpoint validation

### Anti-Patterns Avoided

| Anti-Pattern | Why It's Bad | What We Do Instead |
|-------------|-------------|-------------------|
| Hardcoded IPs in workflow | Drift between environments | Use `vars.K8S_SERVICE_IP_{ENV}` per environment |
| Same secret name for all envs | Cross-env contamination risk | `AZURE_CREDENTIALS_{ENV}` scoped per environment |
| `pull_request` trigger on CI/CD | Deploys on PR open (dangerous) | Only `push` to protected branches |
| No concurrency control | Parallel deploys corrupt state | Concurrency group per service + env |
| Swagger health check on private clusters | Times out, blocks pipeline | Skip for prod/staging (firewall blocks) |

---

## 5. Kubernetes Operations Patterns

### ConfigMap Updates (The Correct Way)

```bash
# 1. Find the ACTUAL configmap name the deployment uses
kubectl get deployment {name} -n {ns} -o jsonpath='{.spec.template.spec.containers[0].envFrom[0].configMapRef.name}'

# 2. Patch the CORRECT configmap (not a new one!)
kubectl patch configmap {actual-name} -n {ns} --type merge -p '{"data":{"KEY":"VALUE"}}'

# 3. Restart the deployment to pick up changes
kubectl rollout restart deployment {name} -n {ns}

# 4. Verify
kubectl exec -n {ns} {new-pod} -- env | grep KEY
```

**Lesson learned**: Applying a YAML with `kubectl apply -f` creates a NEW configmap if the name differs from what the deployment references. Always verify the deployment's `envFrom` first.

### Secret Management (CSI Driver)

```
Key Vault → SecretProviderClass → K8s Secret object → Pod secretKeyRef
```

**Critical**: CSI mounts the file correctly, but the `secretObjects` K8s Secret can become stale. Fix: delete the K8s secret + restart pods.

### HPA + Replicas Interaction

- HPA overrides the deployment's `replicas` field
- Setting `replicas: 2` means nothing if HPA `minReplicas: 1`
- Always update BOTH the deployment AND the HPA together

---

## 6. MongoDB Diagnostic Patterns

### Performance Analysis Steps

1. **Get collection stats**: `db.collection.stats()` — size, count, index count
2. **Check indexes**: `db.collection.getIndexes()` — missing = COLLSCAN
3. **Find slow queries**: `db.currentOp({secs_running: {$gte: 2}})` 
4. **Explain a query**: `db.collection.find({field: value}).explain("executionStats")`
5. **Connection pool**: `db.serverStatus().connections` — current vs available

### Common Fixes

| Issue | Indicator | Fix |
|-------|-----------|-----|
| COLLSCAN | 47ms+ query, high `docsExamined` | Create compound index |
| Pool exhaustion | `connections.current` near max | Increase `maxPoolSize`, check for leaks |
| Auth failure | SCRAM-SHA-1 error | Check creds, CSI sync, connection string encoding |
| Timeout | `serverSelectionTimeoutMS` errors | Check NSG rules, VM reachability |

---

## 7. Multi-Environment Architecture Awareness

### Environment-Specific Knowledge Map

| Aspect | Dev | QA | Staging | Prod |
|--------|-----|-----|---------|------|
| **Networking** | Unified ingress (ClusterIP) | Unified ingress (ClusterIP) | Direct LoadBalancer per svc | Ingress internal + external |
| **APIM routing** | External ingress IP | External ingress IP | Direct LB IP per service | Internal VNet IP (10.0.1.62) |
| **Swagger** | Accessible externally | Accessible externally | NOT accessible externally | NOT accessible (firewall) |
| **Secrets** | Key Vault + CSI | Key Vault + CSI | Key Vault + CSI | Key Vault + CSI (stale risk!) |
| **DB** | VM (20.233.17.126) | VM (same VNet) | VM (20.74.220.193) | VM (10.0.2.4, private subnet) |

### Key Differences That Cause Issues

1. **Staging has no ingress** → Swagger unreachable from internet → Workflow must skip health check
2. **Prod APIM uses internal IP** → `K8S_SERVICE_IP_PROD` must be `10.0.1.62`, not external
3. **Dev APIM path inconsistency** → `api/quotations` vs `api/quotation` → always verify
4. **CSI secret sync** → K8s Secret objects can be stale even when KV has correct value

---

## 8. Communication & Documentation Patterns

### Stakeholder Communication

- **Brief first, detail if asked** — lead with impact/numbers
- **Risk eliminated > task completed** — frame work in terms of risk reduction
- **Before/After tables** — visual proof of improvement

### Documentation Locations

| What | Where | When to Update |
|------|-------|---------------|
| Architecture decisions | Confluence (MPM space) | After each major change |
| Operational runbooks | `D:\Mahdy\Mind\platform-docs/` | Weekly |
| Task tracking | Jira MDI board | After each task completes |
| K8s manifests | `kubernetes` repo | With every cluster change |
| CI/CD config | Each backend repo `.github/workflows/` | With every workflow change |

---

## 9. Predictive Thinking Checklist

Before applying ANY change, mentally walk through:

- [ ] Will this trigger a workflow? (Check branch triggers)
- [ ] Will the workflow succeed? (Check env vars, secrets, connectivity)
- [ ] Does this affect other services sharing the same resource? (shared configmaps, shared KV)
- [ ] Is there a timing dependency? (pod restart before new config available?)
- [ ] What will the developer see? (API response, swagger, error messages)
- [ ] Is the git file in sync with the live cluster? (drift detection)
- [ ] Does prod need a separate handling? (firewall, VNet, internal IPs)

---

## 10. Skills Demonstrated & Reusable

| Skill Category | Specific Skills |
|---------------|----------------|
| **Kubernetes** | Deployments, HPA, ConfigMaps, Secrets, CSI Driver, RBAC, Ingress, Health Probes, Jobs |
| **CI/CD** | GitHub Actions (multi-env), concurrency, artifacts, environment protection, workflow_dispatch |
| **Azure** | AKS, APIM, Key Vault, ACR, App Gateway, Firewall, VMs, Blob Storage, NSGs |
| **MongoDB** | Indexes, connection pooling, SCRAM auth, replica sets, performance analysis |
| **Networking** | Ingress routing, VNet integration, internal vs external IPs, NSG rules, SSH |
| **Troubleshooting** | Log analysis, `kubectl exec`, connectivity testing, APIM tracing, CSI debugging |
| **Security** | Secret rotation, CSI driver, RBAC, service principals, SSH key management |
| **Documentation** | Confluence API, Jira API, structured READMEs, runbooks |
| **Automation** | Python scripts for batch operations, shell scripting, bulk git operations |
