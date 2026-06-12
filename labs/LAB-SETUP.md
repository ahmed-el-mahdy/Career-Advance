# 🖥️ Local Lab Environment Setup

> Set up your machine for daily CKA practice

---

## Prerequisites

- Windows with WSL2 or Git Bash
- Docker Desktop installed
- At least 8GB RAM free

---

## Option 1: kind (Recommended for CKA)

```bash
# Install kind (Kubernetes IN Docker)
# Windows (chocolatey):
choco install kind

# Or download binary:
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-windows-amd64
mv ./kind /usr/local/bin/kind

# Create a cluster
kind create cluster --name cka-lab

# Create multi-node cluster (realistic for CKA)
cat <<EOF > kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF
kind create cluster --name cka-multi --config kind-config.yaml

# Verify
kubectl get nodes
kubectl cluster-info

# Delete when done
kind delete cluster --name cka-lab
```

---

## Option 2: Minikube

```bash
# Install
choco install minikube

# Start with resources
minikube start --cpus=2 --memory=4096 --driver=docker

# Enable addons useful for CKA
minikube addons enable metrics-server
minikube addons enable ingress
minikube addons enable dashboard

# Access dashboard
minikube dashboard
```

---

## Option 3: KillerCoda (Zero Setup)

Just go to https://killercoda.com/playgrounds/scenario/kubernetes
- Browser-based
- Fresh cluster every time
- 60 min sessions (restart for more)
- Perfect for quick daily drills

---

## kubectl Autocomplete Setup

```bash
# Bash
echo 'source <(kubectl completion bash)' >> ~/.bashrc
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc
source ~/.bashrc

# Verify
k get no<TAB>  # should autocomplete to 'nodes'
```

---

## Useful kubectl Plugins (krew)

```bash
# Install krew (kubectl plugin manager)
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/aarch64/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)

# Useful plugins
kubectl krew install ctx     # switch contexts fast
kubectl krew install ns      # switch namespaces fast
kubectl krew install neat    # clean up yaml output
kubectl krew install tree    # visualize object relationships
```

---

## Daily CKA Warm-up Script

Save this as `~/cka-warmup.sh` and run daily:

```bash
#!/bin/bash
echo "=== CKA Daily Warm-up ==="
echo ""

# Setup
alias k=kubectl
export do="--dry-run=client -o yaml"

echo "1. Create namespace 'practice'"
k create ns practice

echo "2. Run a pod"
k run web --image=nginx -n practice

echo "3. Create deployment"
k create deployment api --image=httpd --replicas=2 -n practice

echo "4. Expose as service"
k expose deployment api --port=80 -n practice

echo "5. Create configmap"
k create configmap app-config --from-literal=env=dev -n practice

echo "6. Create secret"
k create secret generic db-pass --from-literal=password=s3cret -n practice

echo "7. Create role + binding"
k create role viewer --verb=get,list --resource=pods -n practice
k create rolebinding viewer-binding --role=viewer --user=dev -n practice

echo ""
echo "✅ Warm-up complete. Now practice troubleshooting!"
echo "Try: k get all -n practice"
```

---

## Azure CLI Setup (for AZ-104)

```bash
# Install Azure CLI
# Windows:
winget install Microsoft.AzureCLI

# Login
az login

# Set subscription
az account set --subscription "your-subscription-name"

# Verify
az account show --output table
```

---

*kind clusters spin up in ~30 seconds. No excuses. Practice daily.*
