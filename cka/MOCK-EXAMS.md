# ☸️ CKA Mock Exams & Practice Tests

---

## 🆓 Free Mock Exams & Scenarios

### KillerCoda (Best Free Option)
- **CKA Scenarios**: https://killercoda.com/cka
- **K8s Playgrounds**: https://killercoda.com/playgrounds/scenario/kubernetes
- Browser-based, no setup needed
- Multiple CKA-specific exam scenarios
- New scenarios added regularly

### GitHub Practice Sets
| Repo | Questions | Link |
|------|-----------|------|
| dgkanatsios/CKAD-exercises | 80+ tasks (applicable to CKA) | https://github.com/dgkanatsios/CKAD-exercises |
| alijahnas/CKA-practice-exercises | Full CKA curriculum coverage | https://github.com/alijahnas/CKA-practice-exercises |
| kodekloudhub/certified-kubernetes-administrator-course | Lab companion | https://github.com/kodekloudhub/certified-kubernetes-administrator-course |
| walidshaari/Kubernetes-Certified-Administrator | Resources collection | https://github.com/walidshaari/Kubernetes-Certified-Administrator |
| StenlyTU/K8s-training-official | Official docs exercises | https://github.com/StenlyTU/K8s-training-official |

### KodeKloud (Your Existing Access)
- You already completed the CKA course
- Go back and redo the **mock exams** (Lightning Labs + Mock Exams 1-3)
- Focus on timing: 7 min per question max

---

## 💰 Paid Mock Exams (When Ready)

| Platform | Cost | Notes |
|----------|------|-------|
| killer.sh | Free (2 sessions with exam purchase) | Most realistic — same environment as real exam |
| KodeKloud Ultimate CKA Mock | Included if you have subscription | Good difficulty level |

---

## 📝 Self-Made Mock Exam (Do This Weekly)

### Mock Exam #1 — Cluster Setup & RBAC
Set a timer for 30 minutes and complete:

1. Create namespace `exam-ns`
2. Create ServiceAccount `app-sa` in `exam-ns`
3. Create Role `pod-manager` allowing get, list, create, delete on pods in `exam-ns`
4. Bind `pod-manager` to `app-sa`
5. Verify: `kubectl auth can-i get pods --as=system:serviceaccount:exam-ns:app-sa -n exam-ns`
6. Create a pod `test-pod` using image nginx running as serviceAccount `app-sa`
7. Backup etcd to `/tmp/etcd-backup.db`

### Mock Exam #2 — Workloads & Storage
Timer: 30 minutes

1. Create deployment `web` with nginx:1.24, 3 replicas in namespace `production`
2. Update the image to nginx:1.25 using rolling update
3. Verify rollout history shows 2 revisions
4. Create a PV with 2Gi, hostPath `/data/pv`, accessMode ReadWriteOnce
5. Create a PVC requesting 1Gi
6. Mount PVC into a pod at `/usr/share/nginx/html`
7. Create a CronJob that runs `echo "backup done"` every 5 minutes

### Mock Exam #3 — Networking
Timer: 30 minutes

1. Create deployment `frontend` (nginx, 2 replicas) and `backend` (httpd, 2 replicas)
2. Expose both as ClusterIP services
3. Create a NetworkPolicy: `backend` pods only accept traffic from `frontend` pods
4. Verify: exec into frontend pod → curl backend (should work)
5. Create another pod `attacker` → curl backend (should be blocked)
6. Create an Ingress routing `/app` → frontend and `/api` → backend
7. Fix DNS: run nslookup from a pod to verify service discovery works

### Mock Exam #4 — Troubleshooting
Timer: 30 minutes

1. A pod is in CrashLoopBackOff — find and fix the issue (wrong command)
2. A deployment has 0/3 ready replicas — diagnose (wrong image name)
3. A service has no endpoints — fix it (label mismatch)
4. Node `worker01` shows NotReady — identify cause (kubelet stopped)
5. A pod can't resolve DNS — troubleshoot (CoreDNS pod not running)
6. A pod is Pending — find why (insufficient resources / taint)
7. Recover cluster from etcd backup at `/tmp/etcd-backup.db`

---

## ⏱️ Exam Day Strategy

```
Minutes 0-5:    Setup aliases, test cluster access
Minutes 5-60:   Easy/medium questions first (skip hard ones)
Minutes 60-90:  Return to skipped questions
Minutes 90-110: Double-check answers with `kubectl get`
Minutes 110-120: Final review of flagged questions
```

### Red Flags During Exam:
- Spending >10 min on one question → flag and move on
- Forgot to switch context → always check `kubectl config current-context`
- Writing YAML from scratch → use `--dry-run=client -o yaml` instead
- Pod stuck in Pending → check: resources, nodeSelector, taints, scheduler

---

*Do one mock exam per week (timed). Track your score. When you consistently score 80%+, you're ready.*
