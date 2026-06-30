# Impact Snapshot — Ahmed ElMahdy

> One-page summary for recruiters and interviews. Print or attach as PDF.

---

## Profile

**Platform / DevOps Engineer** | 5+ years Cloud & DevOps | Azure & AWS
Currently operating production Kubernetes platforms across two enterprise products.

---

## Platforms Under Management

| Platform | Stack | Scale |
|----------|-------|-------|
| Mosadad (Insurance Recovery) | Azure AKS, Terraform, .NET 8, Angular 19 | 7 microservices, 4 AKS clusters, 4 environments |
| ActionAI (AI Automation) | Azure AKS, Helm, Argo CD, Kafka, .NET 8 | 12 microservices, 3 environments |

---

## Key Impact Metrics

| Metric | Value | Context |
|--------|-------|---------|
| HPA Policies | **24** | 6 services × 4 environments with tiered scaling (dev:2, qa:3, staging/prod:4) |
| Staging IaC Drift | **0** | Full Terraform alignment across all environments |
| Alert Rules | **4** + **3** availability tests | Grafana/Prometheus + Application Insights APM |
| Microservices Operated | **19** total | 7 (Mosadad) + 12 (ActionAI) |
| AKS Clusters | **4** production | Standard_D4ds_v5 nodes, NGINX ingress |
| CI/CD Pipelines | **DRY reusable** | GitHub Actions → ACR → AKS → APIM auto-import |
| Workload Optimization | Claims tuned to **1 vCPU / 2Gi** | Improved PDF processing stability |

---

## Scope of Delivery

**Infrastructure:** Terraform modular IaC (AKS, APIM, App Service, Key Vault, networking, monitoring modules)

**Kubernetes:** Kustomize overlays (dev/qa/staging/prod), HPA per service, NGINX ingress path routing, CSI secret provider

**Security:** App Gateway WAF_v2, Azure Firewall, APIM rate limiting, Key Vault CSI, B2C/Entra ID authentication

**CI/CD:** GitHub Actions with DRY reusable workflows, Docker multi-stage builds, ACR, automated APIM import

**Monitoring:** Grafana + Prometheus dashboards, Application Insights APM, alert rules, availability tests

**GitOps (ActionAI):** Argo CD app-of-apps pattern, Helm umbrella chart, Kafka Strimzi operator

---

## Certifications

- AWS Solutions Architect – Associate
- AWS Certified Cloud Practitioner
- Microsoft SC-300 (Identity & Access Admin)
- McKinsey Forward Program

---

## Contact

- **Portfolio:** ahmed-el-mahdy.github.io
- **GitHub:** github.com/ahmed-el-mahdy
- **LinkedIn:** linkedin.com/in/ahmed-el-mahdy
