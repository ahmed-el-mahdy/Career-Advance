# ☁️ AZ-104 — Azure Administrator Associate Study Plan

> Target: Pass the AZ-104 exam (Microsoft)
> Duration: 4–6 weeks of focused study
> Exam Format: Multiple choice + case studies, 40-60 questions, ~120 min, 700/1000 to pass

---

## 📚 Free Resources

### Video Courses (Free)
| Resource | Duration | Link |
|----------|----------|------|
| **John Savill's AZ-104 Study Cram** | 4 hours | https://www.youtube.com/watch?v=VOod_VNgdJk |
| John Savill's AZ-104 Full Course | 18 hours | https://www.youtube.com/playlist?list=PLlVtbbG169nGlGPWs9xaLKT1KfwqREHbs |
| freeCodeCamp AZ-104 Course | 11 hours | https://www.youtube.com/watch?v=10PbGbTUSAg |
| Adam Marczak — Azure for Everyone | Playlist | https://www.youtube.com/c/intabornet |

### Microsoft Learn (Official & Free)
| Path | Link |
|------|------|
| **AZ-104 Learning Path (Complete)** | https://learn.microsoft.com/en-us/training/paths/az-104-administrator/ |
| Prerequisites: Azure Fundamentals | https://learn.microsoft.com/en-us/training/paths/az-900-describe-cloud-concepts/ |
| Manage identities and governance | https://learn.microsoft.com/en-us/training/paths/az-104-manage-identities-governance/ |
| Implement and manage storage | https://learn.microsoft.com/en-us/training/paths/az-104-manage-storage/ |
| Deploy and manage compute | https://learn.microsoft.com/en-us/training/paths/az-104-manage-compute-resources/ |
| Configure and manage virtual networking | https://learn.microsoft.com/en-us/training/paths/az-104-manage-virtual-networks/ |
| Monitor and maintain Azure resources | https://learn.microsoft.com/en-us/training/paths/az-104-monitor-backup-resources/ |

### Free Lab Environments
| Platform | Description | Link |
|----------|-------------|------|
| **Microsoft Learn Sandboxes** | Free Azure environments in each module (no credit card!) | Built into Learn modules |
| **Azure Free Account** | $200 credit for 30 days + always-free services | https://azure.microsoft.com/free |
| **Your Mosadad Subscription** | You already have production hands-on experience | Daily work! |

### Practice Tests (Free)
| Resource | Link |
|----------|------|
| Microsoft Practice Assessment | https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/practice/assessment?assessment-type=practice&assessmentId=21 |
| ExamTopics (community answers) | https://www.examtopics.com/exams/microsoft/az-104/ |
| John Savill's Study Questions | Included in his YouTube playlist |

---

## 📋 Exam Domains & Weight

| Domain | Weight | Topics |
|--------|--------|--------|
| **Manage Azure Identities & Governance** | 20-25% | Entra ID, RBAC, subscriptions, policies, management groups |
| **Implement & Manage Storage** | 15-20% | Storage accounts, blob, files, redundancy, access tiers, AzCopy |
| **Deploy & Manage Azure Compute** | 20-25% | VMs, VMSS, App Service, containers, ARM templates |
| **Configure & Manage Virtual Networking** | 20-25% | VNets, NSG, peering, VPN, Load Balancer, DNS, App Gateway |
| **Monitor & Maintain Azure Resources** | 10-15% | Azure Monitor, alerts, Log Analytics, backup, recovery |

---

## 🗓️ 4-Week Study Plan

### Week 1: Identity & Governance (20-25%)
| Day | Topic | Practice |
|-----|-------|----------|
| 1 | Entra ID: Users, groups, licenses | Create users/groups in portal, Entra admin center |
| 2 | RBAC: Built-in roles, custom roles, scope | Assign roles at subscription/RG/resource level |
| 3 | Azure Policy & Initiatives | Create a policy that denies public IPs |
| 4 | Management Groups & Subscriptions | Understand hierarchy, cost management |
| 5 | Entra ID: MFA, SSPR, conditional access | Configure MFA for test users |
| 6 | Azure AD Connect, B2B/B2C basics | Understand hybrid identity scenarios |
| 7 | Review + Microsoft Learn Practice Assessment | Domain 1 questions |

### Week 2: Storage & Compute (35-45%)
| Day | Topic | Practice |
|-----|-------|----------|
| 1 | Storage accounts: types, redundancy (LRS/GRS/ZRS) | Create accounts with different tiers |
| 2 | Blob storage: access tiers, lifecycle management | Upload blobs, move between hot/cool/archive |
| 3 | Azure Files, File Sync, AzCopy | Create file shares, use AzCopy to transfer |
| 4 | VMs: Create, resize, availability sets/zones | Deploy VMs with different SKUs |
| 5 | VMSS: Autoscale rules, upgrade policies | Create scale set with custom autoscale |
| 6 | App Service: Plans, deployment slots, scaling | Deploy a web app with staging slot |
| 7 | ARM Templates & Bicep basics, Azure Container Instances | Deploy from template |

### Week 3: Virtual Networking (20-25%)
| Day | Topic | Practice |
|-----|-------|----------|
| 1 | VNets, subnets, IP addressing | Design address space, create subnets |
| 2 | NSGs and ASGs | Create rules, associate with subnets/NICs |
| 3 | VNet peering (local & global) | Peer two VNets, test connectivity |
| 4 | VPN Gateway (point-to-site, site-to-site) | Understand gateway SKUs, connection types |
| 5 | Azure Load Balancer (public & internal) | Create LB with health probes, rules |
| 6 | Application Gateway + WAF | Path-based routing, SSL termination |
| 7 | Azure DNS, Private DNS, service endpoints, private endpoints | Configure DNS zones |

### Week 4: Monitoring + Full Review
| Day | Topic | Practice |
|-----|-------|----------|
| 1 | Azure Monitor: Metrics, logs, dashboards | Create metric alerts for a VM |
| 2 | Log Analytics workspace, KQL basics | Write simple KQL queries |
| 3 | Azure Backup: VM backup, recovery vault | Configure backup policy, test restore |
| 4 | Network Watcher: connection troubleshoot, NSG flow logs | Diagnose connectivity |
| 5 | Full practice exam #1 | Microsoft Practice Assessment |
| 6 | Review weak areas + redo labs | Focus on lowest-scoring domains |
| 7 | Full practice exam #2 | ExamTopics or John Savill questions |

---

## 💡 Your Advantage

You already use these daily at Mosadad & Julfar:
- ✅ Entra ID (SC-300 certified!)
- ✅ RBAC & Policy
- ✅ VNets, NSGs, peering
- ✅ App Gateway, WAF
- ✅ Azure Monitor, App Insights
- ✅ Key Vault, storage accounts
- ✅ ARM/Bicep (through Terraform knowledge)

**Your gaps to focus on**: VMSS autoscale rules, Azure Backup/Recovery, VPN Gateway config details, AzCopy commands, storage lifecycle management, Log Analytics KQL syntax.

---

## 💰 Free Exam Voucher Strategies

| Method | Details |
|--------|---------|
| **Microsoft Cloud Skills Challenge** | Free challenges → free voucher. Check: https://www.microsoft.com/en-us/cloudskillschallenge |
| **Microsoft Ignite / Build events** | Attend virtual → sometimes get free voucher |
| **Microsoft Virtual Training Days** | Register for free events, occasionally include voucher |
| **Employer sponsorship** | Ask Julfar/Mosadad to cover the $165 |
| **Microsoft ESI** (if available) | Enterprise Skills Initiative — ask your company |

---

## ⚡ Exam Day Tips

1. **You CAN'T go back** on some questions (case study sections) — read carefully
2. **Flag questions** you're unsure about — return if time allows
3. **Eliminate wrong answers** — usually 2 are obviously wrong
4. **Azure CLI vs PowerShell** — know both syntax for common operations
5. **Read the FULL question** — "most cost-effective" vs "least administrative effort" changes the answer
6. **Time**: ~2 min per question average, case studies take longer

---

*You have SC-300 + production Azure experience. AZ-104 will validate what you already do daily. Focus on the gaps listed above.*
