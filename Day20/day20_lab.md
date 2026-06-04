# Day 20 – 12-Jun-2026 (Friday)
# Module 4 – Kubernetes with Amazon EKS

# Lab: Helm Deployments & Autoscaling

## AWS Resources Used

- Amazon EKS
- Helm
- Kubernetes Metrics Server
- Auto Scaling

---

# Lab Objectives

Participants will:

- Install Helm.
- Create Helm repositories.
- Deploy applications using Helm.
- Manage Helm releases.
- Install Metrics Server.
- Configure HPA.
- Test autoscaling.
- Verify scaling operations.

---

# Prerequisites

Verify Cluster Access:

```bash
kubectl get nodes
```

Verify Helm:

```bash
helm version
```

---

# Lab 1: Install Helm Repository

Add Bitnami Repository:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Update Repository:

```bash
helm repo update
```

Search Charts:

```bash
helm search repo nginx
```

---

# Lab 2: Deploy NGINX Using Helm

```bash
helm install webserver bitnami/nginx
```

Verify:

```bash
helm list
```

```bash
kubectl get pods
```

---

# Lab 3: Review Helm Release

```bash
helm status webserver
```

Review:

- Release Name
- Namespace
- Resources

---

# Lab 4: Upgrade Helm Release

```bash
helm upgrade webserver bitnami/nginx
```

Verify:

```bash
helm history webserver
```

---

# Lab 5: Rollback Release

```bash
helm rollback webserver 1
```

Verify:

```bash
helm history webserver
```

---

# Lab 6: Install Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Verify:

```bash
kubectl get pods -n kube-system
```

---

# Lab 7: Verify Metrics Collection

```bash
kubectl top nodes
```

```bash
kubectl top pods
```

---

# Lab 8: Create Deployment

```bash
kubectl create deployment autoscale-app --image=nginx
```

Expose Deployment:

```bash
kubectl expose deployment autoscale-app --port=80 --type=ClusterIP
```

---

# Lab 9: Configure HPA

```bash
kubectl autoscale deployment autoscale-app --cpu-percent=70 --min=2 --max=5
```

Verify:

```bash
kubectl get hpa
```

---

# Lab 10: Generate Load

Run Load Generator:

```bash
kubectl run load-generator --rm -it --image=busybox -- sh
```

Inside Pod:

```bash
while true; do wget -q -O- http://autoscale-app; done
```

---

# Lab 11: Observe Autoscaling

Monitor:

```bash
kubectl get hpa -w
```

```bash
kubectl get pods -w
```

Observe:

- CPU increase
- Replica increase

---

# Lab 12: Verify Cluster Scaling

AWS Console:

EKS → Node Groups

Observe:

- Additional nodes (if Cluster Autoscaler enabled)

---

# Lab 13: Cleanup

Delete Resources:

```bash
kubectl delete deployment autoscale-app
```

```bash
kubectl delete hpa autoscale-app
```

```bash
helm uninstall webserver
```

---

# Challenge Exercise

Implement:

1. Helm Deployment
2. Custom Values File
3. HPA Configuration
4. Load Testing
5. Autoscaling Validation

Document:

- Commands
- Screenshots
- Results

---

# Lab Deliverables

Submit:

- Helm Deployment Screenshot
- Helm Release Output
- Metrics Server Screenshot
- HPA Screenshot
- Autoscaling Screenshot
- Scaling Metrics Screenshot

---

# Expected Learning Outcomes

✓ Deploy Applications with Helm

✓ Manage Helm Releases

✓ Upgrade and Rollback Releases

✓ Install Metrics Server

✓ Configure HPA

✓ Generate Load Tests

✓ Validate Autoscaling in Amazon EKS
