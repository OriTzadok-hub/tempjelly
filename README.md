# 🏃‍♂️ Self-Hosted GitHub Actions Runner on AKS

This guide walks you through setting up [actions-runner-controller](https://github.com/actions/actions-runner-controller) in an Azure Kubernetes Service (AKS) cluster using a **GitHub Personal Access Token (PAT)**.

---

## 📋 Prerequisites

- Azure subscription
- AKS cluster (already created and `kubectl` configured)
- Helm installed
- GitHub PAT with `repo` and `admin:repo_hook` scopes

---

## 🔐 Step 1: Install `cert-manager`

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true
```

Wait for cert-manager pods to become ready:

```bash
kubectl get pods -n cert-manager
```

---

## 📦 Step 2: Install Actions Runner Controller

```bash
helm repo add actions-runner-controller https://actions-runner-controller.github.io/actions-runner-controller
helm repo update

helm install actions-runner-controller actions-runner-controller/actions-runner-controller \
  --namespace actions-runner-system \
  --create-namespace
```

---

## 🔑 Step 3: Create Secret with GitHub PAT

```bash
kubectl create secret generic controller-manager -n actions-runner-system \
  --from-literal=github_token=<YOUR_GITHUB_PAT>
```

Replace `<YOUR_GITHUB_PAT>` with your token.

---

## 🤖 Step 4: Deploy the Runner

```bash
kubectl apply -f runner.yaml
```

---

## ✅ Step 5: Verify Setup

Check pod status:

```bash
kubectl get pods -n actions-runner-system
```

Check runner registration:

```bash
kubectl get runners -n actions-runner-system
```

---

## 🧹 Cleanup

To clean up everything:

```bash
terraform destroy
```

This removes the AKS cluster and all associated resources, including the runner controller and runner pods.

---

## 📝 Notes

- The webhook errors (e.g., `mutate.runnerdeployment.actions.summerwind.dev`) may persist until `cert-manager` and the webhook services are fully operational.
- If you re-install the runner controller, you might need to delete stuck webhook configurations:

```bash
kubectl delete validatingwebhookconfiguration arc-actions-runner-controller-validating-webhook-configuration
```