# alesrpi-lab-iac

## Setup

### 1. Prerequisites

* **Ansible** (`brew install ansible`)
* **Kubectl** (`brew install kubernetes-cli`)

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

### 4. Swap rbpi local loopback address

```fish
# extract IP and update config file
set PI_IP (ssh -G rbpi | awk '/^hostname / {print $2}')
sed -i '' "s/127.0.0.1/$PI_IP/g" kubeconfig.yaml
set -Ux KUBECONFIG $PWD/kubeconfig.yaml
```

### 5. Check

```bash
kubectl get nodes
```
