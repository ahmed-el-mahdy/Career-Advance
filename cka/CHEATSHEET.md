# ☸️ CKA kubectl Imperative Commands Cheatsheet

> These commands are FASTER than writing YAML during the exam.
> Practice them until they're muscle memory.

---

## 🛠️ Setup (Run first in exam)

```bash
alias k=kubectl
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
complete -o default -F __start_kubectl k
```

---

## 📦 Pods

```bash
# Create pod
k run nginx --image=nginx
k run nginx --image=nginx --port=80 --labels="app=web,tier=frontend"

# Generate YAML (don't create)
k run nginx --image=nginx $do > pod.yaml

# Pod with command
k run busybox --image=busybox --command -- sleep 3600
k run busybox --image=busybox -- /bin/sh -c "echo hello"

# Pod with environment variables
k run nginx --image=nginx --env="DB_HOST=mysql" --env="DB_PORT=3306"

# Delete pod immediately
k delete pod nginx $now

# Get pod logs
k logs nginx
k logs nginx -c sidecar          # specific container
k logs nginx --previous          # previous crashed container

# Execute into pod
k exec -it nginx -- /bin/bash
k exec nginx -- cat /etc/resolv.conf

# Pod with resource limits
k run nginx --image=nginx $do > pod.yaml
# Then edit YAML to add resources
```

---

## 🚀 Deployments

```bash
# Create deployment
k create deployment web --image=nginx --replicas=3

# Scale
k scale deployment web --replicas=5

# Update image
k set image deployment/web nginx=nginx:1.25

# Rollout management
k rollout status deployment/web
k rollout history deployment/web
k rollout undo deployment/web
k rollout undo deployment/web --to-revision=2

# Generate YAML
k create deployment web --image=nginx --replicas=3 $do > deploy.yaml
```

---

## 🌐 Services

```bash
# Expose deployment (ClusterIP)
k expose deployment web --port=80 --target-port=8080

# Expose as NodePort
k expose deployment web --port=80 --type=NodePort

# Expose pod directly
k expose pod nginx --port=80 --name=nginx-svc

# Create service without selector (then edit)
k create service clusterip my-svc --tcp=80:80 $do > svc.yaml
```

---

## 🔐 RBAC

```bash
# Create role
k create role pod-reader --verb=get,list,watch --resource=pods

# Create clusterrole
k create clusterrole node-reader --verb=get,list --resource=nodes

# Create rolebinding
k create rolebinding read-pods --role=pod-reader --user=dev-user
k create rolebinding read-pods --role=pod-reader --serviceaccount=default:mysa

# Create clusterrolebinding
k create clusterrolebinding admin-binding --clusterrole=cluster-admin --user=admin

# Check permissions
k auth can-i get pods --as=dev-user
k auth can-i get pods --as=system:serviceaccount:default:mysa
k auth can-i "*" "*" --as=admin  # check if cluster-admin
```

---

## 🗂️ ConfigMaps & Secrets

```bash
# ConfigMap from literal
k create configmap app-config --from-literal=DB_HOST=mysql --from-literal=DB_PORT=3306

# ConfigMap from file
k create configmap app-config --from-file=config.txt

# Secret from literal
k create secret generic db-creds --from-literal=username=admin --from-literal=password=secret

# Secret (docker registry)
k create secret docker-registry regcred --docker-server=acr.io --docker-username=user --docker-password=pass
```

---

## 💾 Storage

```bash
# PersistentVolume (must write YAML)
cat <<EOF | k apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/pv
EOF

# PersistentVolumeClaim
cat <<EOF | k apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
EOF
```

---

## 🏷️ Labels & Selectors

```bash
# Add label
k label pod nginx env=prod
k label node node01 disk=ssd

# Remove label
k label pod nginx env-

# Filter by label
k get pods -l app=web
k get pods -l 'app in (web,api)'
k get pods -l app=web,tier=frontend

# Show labels
k get pods --show-labels
```

---

## 🧱 Taints & Tolerations

```bash
# Taint a node
k taint nodes node01 key=value:NoSchedule
k taint nodes node01 key=value:NoExecute

# Remove taint
k taint nodes node01 key=value:NoSchedule-

# Drain node (for maintenance)
k drain node01 --ignore-daemonsets --delete-emptydir-data

# Uncordon
k uncordon node01

# Cordon (no new pods)
k cordon node01
```

---

## 🔍 Troubleshooting

```bash
# Pod status
k get pods -o wide
k describe pod nginx
k logs nginx --previous

# Node status
k get nodes
k describe node node01
ssh node01 "systemctl status kubelet"
ssh node01 "journalctl -u kubelet --no-pager | tail -50"

# Events (cluster-wide)
k get events --sort-by=.metadata.creationTimestamp
k get events -A | grep -i error

# DNS debug
k run dns-test --image=busybox:1.28 --rm -it -- nslookup kubernetes.default
k run dns-test --image=busybox:1.28 --rm -it -- nslookup my-svc.my-namespace.svc.cluster.local

# Check endpoints
k get endpoints my-svc

# Resource usage
k top nodes
k top pods -A
```

---

## 🔄 etcd Backup & Restore

```bash
# Backup
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify
ETCDCTL_API=3 etcdctl snapshot status /tmp/etcd-backup.db --write-table

# Restore
ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db \
  --data-dir=/var/lib/etcd-restored

# Then update etcd static pod manifest:
# /etc/kubernetes/manifests/etcd.yaml → volumes.hostPath.path → /var/lib/etcd-restored
```

---

## ⬆️ Kubeadm Upgrade

```bash
# Check available versions
apt list -a kubeadm

# Upgrade control plane
apt update
apt install -y kubeadm=1.29.x-*
kubeadm upgrade plan
kubeadm upgrade apply v1.29.x

# Upgrade kubelet & kubectl on control plane
apt install -y kubelet=1.29.x-* kubectl=1.29.x-*
systemctl daemon-reload
systemctl restart kubelet

# Upgrade worker node
k drain worker01 --ignore-daemonsets
# SSH to worker:
apt install -y kubeadm=1.29.x-*
kubeadm upgrade node
apt install -y kubelet=1.29.x-*
systemctl daemon-reload
systemctl restart kubelet
# Back on control plane:
k uncordon worker01
```

---

*Print this. Practice daily. Pass the exam.* 🎯
