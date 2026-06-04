# Day 21 – 15-Jun-2026 (Monday)
# Module 5 – GitOps & Advanced Deployment

# Lab: ArgoCD Setup & Application Deployment

## AWS Resources Used

- Amazon EKS
- IAM
- ArgoCD

---

# Lab Objectives

Participants will:

- Install ArgoCD.
- Configure access.
- Connect Git repositories.
- Deploy applications using GitOps.
- Configure synchronization.
- Validate deployments.
- Observe drift detection.

---

# Prerequisites

Verify Cluster Access:

```bash
kubectl get nodes
```

---

# Lab 1: Create ArgoCD Namespace

```bash
kubectl create namespace argocd
```

Verify:

```bash
kubectl get ns
```

---

# Lab 2: Install ArgoCD

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verify:

```bash
kubectl get pods -n argocd
```

Wait until all pods are Running.

---

# Lab 3: Access ArgoCD UI

Expose Service:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open:

https://localhost:8080

---

# Lab 4: Retrieve Admin Password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Login:

Username:
admin

Password:
Retrieved Password

---

# Lab 5: Install ArgoCD CLI

Linux:

```bash
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
```

```bash
chmod +x argocd
sudo mv argocd /usr/local/bin/
```

Verify:

```bash
argocd version
```

---

# Lab 6: Login Using CLI

```bash
argocd login localhost:8080
```

Verify:

```bash
argocd account get-user-info
```

---

# Lab 7: Create GitOps Repository

Repository Structure:

app-config/
├── deployment.yaml
├── service.yaml
└── namespace.yaml

Push Repository to GitHub.

---

# Lab 8: Create ArgoCD Application

```bash
argocd app create nginx-app --repo https://github.com/example/app-config.git --path . --dest-server https://kubernetes.default.svc --dest-namespace default
```

Verify:

```bash
argocd app list
```

---

# Lab 9: Synchronize Application

```bash
argocd app sync nginx-app
```

Verify:

```bash
argocd app get nginx-app
```

---

# Lab 10: Validate Deployment

```bash
kubectl get deployments
```

```bash
kubectl get pods
```

Verify application is running.

---

# Lab 11: Enable Auto Sync

```bash
argocd app set nginx-app --sync-policy automated
```

Verify:

```bash
argocd app get nginx-app
```

---

# Lab 12: Test GitOps Workflow

Modify Deployment YAML.

Commit Changes:

```bash
git add .
git commit -m "Scale application"
git push
```

Observe:

ArgoCD automatically deploys changes.

---

# Lab 13: Test Drift Detection

Manually scale deployment:

```bash
kubectl scale deployment nginx --replicas=5
```

Observe:

ArgoCD reports OutOfSync.

Synchronize:

```bash
argocd app sync nginx-app
```

---

# Lab 14: Review Application History

```bash
argocd app history nginx-app
```

Review deployment history.

---

# Lab 15: Cleanup

Delete Application:

```bash
argocd app delete nginx-app
```

Delete Namespace:

```bash
kubectl delete namespace argocd
```

---

# Challenge Exercise

Implement:

1. ArgoCD Installation
2. Git Repository Integration
3. Automated Sync
4. Drift Detection
5. Rollback Validation

Document:

- Repository Structure
- Screenshots
- Deployment Results

---

# Lab Deliverables

Submit:

- ArgoCD Dashboard Screenshot
- Repository Screenshot
- Application Screenshot
- Sync Status Screenshot
- Drift Detection Screenshot
- Deployment History Screenshot

---

# Expected Learning Outcomes

✓ Install ArgoCD

✓ Configure GitOps

✓ Deploy Applications from Git

✓ Enable Automated Sync

✓ Detect Configuration Drift

✓ Manage Continuous Delivery Using ArgoCD
