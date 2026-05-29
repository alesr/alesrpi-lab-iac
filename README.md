# alesrpi-lab-iac

Provision K8s, Tailscale and ~~ArgoCD~~ to Raspberry Pi.

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

### 3. Install K3s & Tailscale (network bridge)

```bash
ansible-playbook site.yml --vault-password-file .vault_pass_file

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

Note: cluster metrics are disable to save memory.

### 5. Tailscale / GH Actions

To allow GitHub Actions to join Tailnet:

1. **generate Auth Keys:** go to [Tailscale Admin Console](https://login.tailscale.com/admin/settings/keys), create an **OAuth Client**, and generate an **Auth Key**.

2. **add secrets:** add the following to your GitHub Repository **Settings > Secrets and variables > Actions**:
* `TS_OAUTH_CLIENT_ID`: foo
* `TS_OAUTH_SECRET`: bar


3. **Update GitHub Secret:**
Paste the content of your `kubeconfig.yaml` into a GitHub Action Secret `KUBECONFIG`.

4. GitHub test workflow:

```yaml
name: Deploy to K3s

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Connect to Tailscale
        uses: tailscale/github-action@v2
        with:
          oauth-client-id: ${{ secrets.TS_OAUTH_CLIENT_ID }}
          oauth-secret: ${{ secrets.TS_OAUTH_SECRET }}
          tags: tag:ci

      - name: Configure Kubeconfig
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBECONFIG }}" > ~/.kube/config
          chmod 600 ~/.kube/config

      - name: Test Cluster Connectivity
        run: |
          kubectl get nodes
```
