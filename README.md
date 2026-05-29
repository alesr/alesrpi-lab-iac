# alesrpi-lab-iac

Provision K8s and ~~ArgoCD~~ in Raspberry Pi.

## Setup

### 1. Prerequisites

```bash
brew install ansible kubernetes-cli helm

ansible-galaxy collection install kubernetes.core

python3 -m venv .venv
source .venv/bin/activate.fish
pip install kubernetes pyyaml
```

### 2. Enable passwordless sudo for Ansible

```bash
ssh rbpi
sudo current_user=alessandro bash -c 'echo "$current_user ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/90-ansible-users'
sudo chmod 440 /etc/sudoers.d/90-ansible-users
```

### 3. Install K3s

```bash
ansible-playbook site.yml

set -Ux KUBECONFIG $PWD/kubeconfig.yaml
```

### 4. Check

```bash
kubectl get nodes
# kubectl get pods -n argocd
```

<!--### 5. Access the ArgoCD Dashboard

```bash
# get password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl port-forward svc/argocd-server -n argocd 8080:443
```-->
