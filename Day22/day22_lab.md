# Day 22 – 16-Jun-2026 (Tuesday)
# Module 5 & 6

# Lab: Flux CD Configuration & Secure Deployment

## AWS Resources Used

- Flux CD
- Amazon EKS
- AWS KMS
- AWS Secrets Manager

---

# Lab Objectives

Participants will:

- Install Flux CD.
- Connect Flux to Git repository.
- Configure GitOps deployments.
- Create AWS Secrets Manager secrets.
- Use KMS encryption.
- Deploy secure applications.
- Validate GitOps synchronization.

---

# Prerequisites

Verify Cluster Access:

```bash
kubectl get nodes
```

Verify AWS Access:

```bash
aws sts get-caller-identity
```

---

# Lab 1: Install Flux CLI

Linux:

```bash
curl -s https://fluxcd.io/install.sh | sudo bash
```

Verify:

```bash
flux --version
```

---

# Lab 2: Bootstrap Flux CD

GitHub Example:

```bash
flux bootstrap github --owner=my-org --repository=flux-demo --branch=main --path=clusters/dev
```

Verify:

```bash
kubectl get pods -n flux-system
```

---

# Lab 3: Verify Flux Components

```bash
kubectl get deployments -n flux-system
```

Review:

- source-controller
- kustomize-controller
- helm-controller
- notification-controller

---

# Lab 4: Verify Git Synchronization

```bash
flux get sources git
```

Verify:

Repository status = Ready

---

# Lab 5: Create Sample Deployment

deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx
```

Commit to Git repository.

---

# Lab 6: Validate Deployment

```bash
kubectl get deployments
```

Verify:

nginx-app deployed automatically.

---

# Lab 7: Create AWS KMS Key

```bash
aws kms create-key
```

Record:

Key ARN

---

# Lab 8: Create Secret in Secrets Manager

```bash
aws secretsmanager create-secret --name training-db-password --secret-string "MySecurePassword123"
```

Verify:

```bash
aws secretsmanager list-secrets
```

---

# Lab 9: Configure IAM Access

Verify IRSA configuration.

Review IAM Policies.

Confirm Pod access permissions.

---

# Lab 10: Create Kubernetes Secret Reference

Create Secret Provider configuration.

Review integration with Secrets Manager.

---

# Lab 11: Deploy Secure Application

Deploy application using secret references.

Verify Pod startup.

---

# Lab 12: Progressive Delivery Simulation

Update deployment image version.

Commit changes.

Observe:

Flux synchronization.

---

# Lab 13: Canary Deployment Exercise

Create:

Version 1 = Stable

Version 2 = Canary

Traffic:

90% → Stable

10% → Canary

Discuss implementation options.

---

# Lab 14: Review Flux Status

```bash
flux get all
```

Review:

- Sources
- Kustomizations
- Helm Releases

---

# Lab 15: Cleanup

Delete:

```bash
kubectl delete deployment nginx-app
```

Remove Flux resources if required.

Delete test secrets.

---

# Challenge Exercise

Implement:

1. Flux Bootstrap
2. GitOps Deployment
3. KMS Key Creation
4. Secrets Manager Integration
5. Secure Application Deployment
6. Canary Deployment Design

Document:

- Commands
- Architecture
- Screenshots

---

# Lab Deliverables

Submit:

- Flux Dashboard Screenshot
- Git Repository Screenshot
- Flux Sync Screenshot
- KMS Key Screenshot
- Secrets Manager Screenshot
- Deployment Screenshot

---

# Expected Learning Outcomes

✓ Install Flux CD

✓ Configure GitOps Deployments

✓ Manage Secrets Securely

✓ Use AWS KMS

✓ Integrate Secrets Manager

✓ Implement Secure Deployments

✓ Understand Progressive Delivery Concepts
