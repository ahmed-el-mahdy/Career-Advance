# ☸️ CKA — Certified Kubernetes Administrator Study Plan

> Target: Pass the CKA exam (CNCF/Linux Foundation)
> Duration: 4–6 weeks of focused daily practice
> Exam Format: Performance-based (CLI tasks in real clusters), 2 hours, 66% to pass

---

## 📚 Free Resources

### Video Courses (Free)
| Resource | Type | Link |
|----------|------|------|
| KillerCoda CKA Scenarios | Interactive Labs | https://killercoda.com/cka |
| TechWorld with Nana — K8s Full Course | YouTube (3.5hr) | https://www.youtube.com/watch?v=X48VuDVv0do |
| Kubernetes Tutorial for Beginners | YouTube (4hr) | https://www.youtube.com/watch?v=s_o8dwzRlu4 |
| John Savill's Kubernetes Master Class | YouTube (4hr) | https://www.youtube.com/watch?v=jW_-KZCjsm0 |
| That DevOps Guy — CKA Series | YouTube Playlist | https://www.youtube.com/playlist?list=PLHq1uqvAteVvUEdqaBeMK2awVThNujwMd |

### Official Documentation (PRIMARY EXAM RESOURCE)
| Resource | Link |
|----------|------|
| Kubernetes Official Docs | https://kubernetes.io/docs |
| Kubectl Reference | https://kubernetes.io/docs/reference/kubectl/ |
| CKA Exam Curriculum | https://github.com/cncf/curriculum |
| Kubernetes API Reference | https://kubernetes.io/docs/reference/kubernetes-api/ |

### Practice Platforms (Free)
| Platform | Description | Link |
|----------|-------------|------|
| KillerCoda | Browser-based K8s labs, CKA-specific | https://killercoda.com |
| Play with Kubernetes | 4-hour free cluster sessions | https://labs.play-with-k8s.com |
| kind (local) | Lightweight local clusters | https://kind.sigs.k8s.io |
| Minikube (local) | Single-node local K8s | https://minikube.sigs.k8s.io |

### GitHub Practice Repos (Free)
| Repo | Description | Link |
|------|-------------|------|
| dgkanatsios/CKAD-exercises | Exercises (works for CKA too) | https://github.com/dgkanatsios/CKAD-exercises |
| walidshaari/Kubernetes-Certified-Administrator | CKA study resources | https://github.com/walidshaari/Kubernetes-Certified-Administrator |
| alijahnas/CKA-practice-exercises | Task-based practice | https://github.com/alijahnas/CKA-practice-exercises |
| kodekloudhub/certified-kubernetes-administrator-course | KodeKloud labs companion | https://github.com/kodekloudhub/certified-kubernetes-administrator-course |

---

## 📋 Exam Domains & Weight

| Domain | Weight | Topics |
|--------|--------|--------|
| **Cluster Architecture, Installation & Configuration** | 25% | RBAC, kubeadm upgrade, etcd backup/restore, HA clusters |
| **Workloads & Scheduling** | 15% | Deployments, Jobs, CronJobs, DaemonSets, resource limits, node affinity |
| **Services & Networking** | 20% | Services, Ingress, NetworkPolicy, DNS, CNI |
| **Storage** | 10% | PV, PVC, StorageClass, volume types |
| **Troubleshooting** | 30% | Node failures, pod crashes, network issues, logs, events |

---

## 🗓️ 4-Week Study Plan

### Week 1: Cluster Architecture (25%)
| Day | Topic | Practice |
|-----|-------|----------|
| 1 | RBAC: Roles, ClusterRoles, Bindings | Create roles, test with `kubectl auth can-i` |
| 2 | ServiceAccounts + RBAC integration | SA → RoleBinding → pod access |
| 3 | kubeadm cluster upgrade (minor version) | KillerCoda: kubeadm upgrade lab |
| 4 | etcd backup & restore | `etcdctl snapshot save/restore` |
| 5 | Cluster installation with kubeadm | Build cluster from scratch (kind or KillerCoda) |
| 6 | Network plugins (CNI), kube-proxy | Understand Calico/Flannel differences |
| 7 | Review + mock scenarios | KillerCoda CKA scenarios |

### Week 2: Workloads + Storage (25%)
| Day | Topic | Practice |
|-----|-------|----------|
| 1 | Deployments: create, scale, rollout | `kubectl rollout status/history/undo` |
| 2 | Jobs & CronJobs | Create jobs with completions/parallelism |
| 3 | DaemonSets & StatefulSets | When to use each, headless services |
| 4 | Resource requests/limits, LimitRange, ResourceQuota | Enforce limits on namespaces |
| 5 | Node affinity, taints, tolerations | Schedule pods on specific nodes |
| 6 | PV, PVC, StorageClass | Dynamic provisioning, access modes |
| 7 | Volume types: hostPath, emptyDir, configMap, secret | Mount all types in pods |

### Week 3: Services & Networking (20%)
| Day | Topic | Practice |
|-----|-------|----------|
| 1 | Services: ClusterIP, NodePort, LoadBalancer | Expose deployments all 3 ways |
| 2 | Ingress controllers + Ingress resources | Create ingress rules with path-based routing |
| 3 | NetworkPolicy | Default deny, allow specific pods/namespaces |
| 4 | CoreDNS, service discovery | `nslookup` from pods, debug DNS |
| 5 | Pod-to-pod networking, CNI | Understand cluster networking model |
| 6 | Endpoint slices, headless services | StatefulSet DNS entries |
| 7 | Review + full networking mock | KillerCoda networking scenarios |

### Week 4: Troubleshooting (30%) + Full Mocks
| Day | Topic | Practice |
|-----|-------|----------|
| 1 | Troubleshoot broken pods: CrashLoopBackOff, ImagePull | `kubectl describe`, `kubectl logs` |
| 2 | Troubleshoot broken nodes: NotReady, kubelet | `systemctl status kubelet`, journalctl |
| 3 | Troubleshoot networking: service unreachable | Check endpoints, selectors, NetworkPolicy |
| 4 | Troubleshoot scheduling: Pending pods | Resource constraints, taints, node selector |
| 5 | Application lifecycle: probes, init containers | Liveness, readiness, startup probes |
| 6 | Full mock exam #1 | KillerCoda or killer.sh (if available) |
| 7 | Full mock exam #2 + review weak areas | Time yourself: 2 hours, 17 questions |

---

## ⚡ Exam Tips

1. **Use imperative commands** — faster than writing YAML:
   ```bash
   kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
   kubectl create deployment web --image=nginx --replicas=3
   kubectl expose deployment web --port=80 --type=NodePort
   ```

2. **Set up aliases at exam start**:
   ```bash
   alias k=kubectl
   export do="--dry-run=client -o yaml"
   export now="--force --grace-period=0"
   ```

3. **Bookmark these docs** (allowed during exam):
   - kubectl cheat sheet
   - API reference for resources
   - kubeadm upgrade docs
   - etcd operations guide

4. **Time management**: 17 questions in 120 min = ~7 min each. Skip hard ones, come back later.

5. **Context switching**: Each question specifies a cluster context. Always run `kubectl config use-context` first.

---

## 💰 Exam Cost Strategies

| Strategy | Savings | When |
|----------|---------|------|
| Linux Foundation Cyber Monday sale | 40-50% off | November |
| KodeKloud CKA bundle (course + exam) | ~$100 off | Periodic sales |
| Employer reimbursement | 100% | Ask HR/manager |
| Linux Foundation student discount | 25% off | If applicable |
| Kubecon attendee discount | Varies | At events |

---

*Remember: You already completed the KodeKloud CKA course. You just need hands-on repetition. Do 30-45 min of `kubectl` drills daily and you'll be ready.*
