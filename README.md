
# Camunda 8 Kubernetes Installation Guide (via Helm & Kind)

## 🕒 Total Time Taken

**\~3 hours**

---

## 🎯 Objective

Install the **Camunda 8 (C8)** stack on a local Kubernetes cluster using **Kind** and **Helm**, and document all necessary steps, issues, and their resolutions encountered throughout the process.

---

## 🧰 Prerequisites

* Windows 10/11 with **Docker Desktop** installed and running
* Basic PowerShell command-line knowledge
* Internet connectivity
* Admin access on your machine

---

## ✅ Step-by-Step Installation

### 1. Docker Setup

* Download and install **Docker Desktop** for Windows.
* Validate the installation:

```powershell
docker version
```

---

### 2. Configure WSL for Resource Allocation

To avoid resource constraints (CPU/memory) when deploying Camunda pods:

1. Edit (or create) a `.wslconfig` file at `C:\Users\<YourUsername>\.wslconfig` with:

```ini
[wsl2]
memory=8GB
processors=4
```

2. Apply changes:

```powershell
wsl --shutdown
# Then restart Docker Desktop manually
```

---

### 3. Install Chocolatey (Windows Package Manager)

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; `
[System.Net.ServicePointManager]::SecurityProtocol = `
[System.Net.ServicePointManager]::SecurityProtocol -bor 3072; `
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

---

### 4. Install Kind (Kubernetes-in-Docker)

```powershell
choco install kind
```

---

### 5. Install Helm (Kubernetes Package Manager)

```powershell
choco install kubernetes-helm
```

---

### 6. Create a Kind Cluster

1. Create a cluster configuration file `kind-config.yaml`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 8080
        hostPort: 8080
```

2. Create the cluster:

```powershell
kind create cluster --config kind-config.yaml
```

---

### 7. Add Camunda Helm Repository

```bash
helm repo add camunda https://helm.camunda.io
helm repo update
```

---

### 8. Create Namespace for Camunda

```bash
kubectl create namespace camunda
```

---

### 9. Install Camunda Platform

Ensure `values.yaml` contains the correct minimal/simplified configuration. Then run:

```bash
helm install camunda-platform camunda/camunda-platform `
  --namespace camunda `
  -f values.yaml
```

---

### 10. Verify Deployment

Check pod status:

```bash
kubectl get pods -n camunda
```

All pods should eventually move to `Running` or `Completed`.

---

## 🐞 Common Issues & Fixes

### 🚫 `Insufficient CPU/Memory`

**Error**:
`0/1 nodes are available: Insufficient cpu, Insufficient memory`

**Fix**:
Update `.wslconfig` to allocate more resources and restart Docker Desktop.

---

### 🚫 `Command not recognized` (Helm/Kind/Kubectl)

**Fix**:
Install missing tools with Chocolatey. Restart PowerShell or your terminal session to refresh the environment variables.

---

### 🚫 Pods stuck in `Pending` or `Init`

**Fix**:
Delete the cluster:

```bash
kind delete cluster
```

Then recreate it after increasing WSL/Docker Desktop memory.

---

### 🚫 `Helm install` fails with invalid `apiVersion`

**Fix**:
Use the correct `apiVersion: kind.x-k8s.io/v1alpha4` in your `kind-config.yaml`.

---

## ✅ Final Result

All Camunda services are deployed and running in the `camunda` namespace:

```bash
kubectl get pods -n camunda
```
