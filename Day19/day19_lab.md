# Day 19 – 11-Jun-2026 (Thursday)
# Module 4 – Kubernetes with Amazon EKS

# Lab: Persistent Storage & Ingress Configuration

## AWS Resources Used

- Amazon EKS
- Amazon EBS
- Amazon EFS
- Amazon Route53

---

# Lab Objectives

Participants will:

- Create Persistent Volumes.
- Create Persistent Volume Claims.
- Use Amazon EBS storage.
- Configure Amazon EFS shared storage.
- Deploy applications with persistent storage.
- Configure Ingress.
- Integrate Route53.
- Verify application access.

---

# Prerequisites

Existing EKS Cluster

Verify:

```bash
kubectl get nodes
```

---

# Lab 1: Verify Storage Classes

```bash
kubectl get storageclass
```

Expected:

gp2 or gp3 storage class.

---

# Lab 2: Create Persistent Volume Claim

pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

Deploy:

```bash
kubectl apply -f pvc.yaml
```

Verify:

```bash
kubectl get pvc
```

---

# Lab 3: Deploy Application with PVC

deployment-pvc.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-storage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-storage
  template:
    metadata:
      labels:
        app: nginx-storage
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: app-storage
      volumes:
      - name: app-storage
        persistentVolumeClaim:
          claimName: app-pvc
```

Deploy:

```bash
kubectl apply -f deployment-pvc.yaml
```

Verify:

```bash
kubectl get pods
```

---

# Lab 4: Verify EBS Volume

AWS Console:

EBS → Volumes

Verify:

- New volume created
- Attached to worker node

---

# Lab 5: Create Amazon EFS File System

AWS Console:

EFS → Create File System

Configuration:

- Regional
- General Purpose

Record:

File System ID

---

# Lab 6: Install EFS CSI Driver

Verify installation:

```bash
kubectl get pods -n kube-system
```

Locate EFS CSI Driver Pods.

---

# Lab 7: Create EFS Storage Class

Review EFS Storage Class example.

Verify dynamic provisioning.

---

# Lab 8: Create EFS PVC

Deploy EFS PVC.

Verify:

```bash
kubectl get pvc
```

---

# Lab 9: Deploy Shared Storage Application

Deploy two Pods sharing EFS.

Verify shared content visibility.

---

# Lab 10: Create Service

```bash
kubectl expose deployment nginx-storage --type=ClusterIP --port=80
```

Verify:

```bash
kubectl get svc
```

---

# Lab 11: Install AWS Load Balancer Controller

Verify:

```bash
kubectl get deployment -n kube-system
```

Locate AWS Load Balancer Controller.

---

# Lab 12: Create Ingress

ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  ingressClassName: alb
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-storage
            port:
              number: 80
```

Deploy:

```bash
kubectl apply -f ingress.yaml
```

Verify:

```bash
kubectl get ingress
```

---

# Lab 13: Configure Route53

Create:

A Record Alias

Point DNS to ALB endpoint.

Example:

app.training.example.com

---

# Lab 14: Validate Access

Open:

http://app.training.example.com

Verify application availability.

---

# Lab 15: Review Security Configuration

Verify:

- IAM Roles
- Security Groups
- Ingress Rules

---

# Lab 16: Cleanup

Delete:

```bash
kubectl delete ingress app-ingress
```

```bash
kubectl delete deployment nginx-storage
```

```bash
kubectl delete pvc app-pvc
```

Verify resource removal.

---

# Challenge Exercise

Implement:

1. EBS Persistent Storage
2. EFS Shared Storage
3. Ingress Controller
4. Route53 DNS Entry
5. Secure Application Access

Document:

- Architecture
- Commands
- Screenshots

---

# Lab Deliverables

Submit:

- PVC Screenshot
- EBS Volume Screenshot
- EFS Screenshot
- Ingress Screenshot
- Route53 Record Screenshot
- Application Access Screenshot

---

# Expected Learning Outcomes

✓ Configure Persistent Storage

✓ Use Amazon EBS

✓ Use Amazon EFS

✓ Configure Ingress

✓ Integrate Route53

✓ Implement Secure Access

✓ Manage Stateful Kubernetes Applications
