# Day 17 – 09-Jun-2026 (Tuesday)
# Module 4 – Kubernetes with Amazon EKS

# Lab: Kubernetes Deployments & Services

## AWS Resources Used

- Amazon EKS
- Amazon EC2
- IAM
- kubectl

---

# Lab Objectives

Participants will:

- Configure kubectl.
- Connect to EKS cluster.
- Deploy applications.
- Scale deployments.
- Create Services.
- Verify Pod networking.
- Manage Kubernetes workloads.

---

# Prerequisites

Existing EKS Cluster

Required Tools:

```bash
aws --version
kubectl version --client
```

Verify AWS Login:

```bash
aws sts get-caller-identity
```

---

# Lab 1: Verify EKS Cluster

List Clusters:

```bash
aws eks list-clusters 
```

Describe Cluster:

```bash
aws eks describe-cluster --name training-eks
```

---

# Lab 2: Configure kubectl

Update kubeconfig:

```bash
aws eks update-kubeconfig --region eu-north-1 --name training-eks
```

Verify:

```bash
kubectl get nodes
```

Expected:

Worker nodes displayed.

---

# Lab 3: Explore Cluster

```bash
kubectl cluster-info
```

```bash
kubectl get nodes
```

```bash
kubectl get namespaces
```

---

# Lab 4: Create Namespace

```bash
kubectl create namespace training
```

Verify:

```bash
kubectl get namespaces
```

---

# Lab 5: Create Deployment

deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
  namespace: training
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - name: hello-app
        image: nginx
        ports:
        - containerPort: 80
```

Deploy:

```bash
kubectl apply -f deployment.yaml
```

Verify:

```bash
kubectl get deployments -n training
```

```bash
kubectl get pods -n training
```

---

# Lab 6: Scale Deployment

Scale to 4 Pods:

```bash
kubectl scale deployment hello-app --replicas=4 -n training
```

Verify:

```bash
kubectl get pods -n training
```

---

# Lab 7: Create ClusterIP Service

service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-service
  namespace: training
spec:
  selector:
    app: hello-app
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

Deploy:

```bash
kubectl apply -f service.yaml
```

Verify:

```bash
kubectl get svc -n training
```

---

# Lab 8: Create LoadBalancer Service

loadbalancer.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-lb
  namespace: training
spec:
  selector:
    app: hello-app
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

Deploy:

```bash
kubectl apply -f loadbalancer.yaml
```

Verify:

```bash
kubectl get svc -n training
```

Wait for EXTERNAL-IP assignment.

---

# Lab 9: Verify Application

```bash
kubectl get svc hello-lb -n training
```

Open:

EXTERNAL-IP

Verify NGINX page.

---

# Lab 10: Inspect Resources

```bash
kubectl describe deployment hello-app -n training
```

```bash
kubectl describe service hello-lb -n training
```

---

# Lab 11: View Logs

```bash
kubectl logs -pod-name -n training
```

---

# Lab 12: Delete Resources

```bash
kubectl delete service hello-lb -n training
```

```bash
kubectl delete service hello-service -n training
```

```bash
kubectl delete deployment hello-app -n training
```

---

# Challenge Exercise

Create:

1. Namespace
2. Deployment with 3 replicas
3. ClusterIP Service
4. LoadBalancer Service
5. Scale deployment to 5 replicas

Verify:

- Pods
- Services
- External access

Document:

- Commands
- Screenshots
- Observations

---

# Lab Deliverables

Submit:

- EKS Cluster Screenshot
- Node List Screenshot
- Deployment Screenshot
- Service Screenshot
- LoadBalancer Screenshot
- Scaling Screenshot

---

# Expected Learning Outcomes

✓ Connect to EKS Cluster

✓ Manage Kubernetes Resources

✓ Create Deployments

✓ Scale Applications

✓ Create Services

✓ Verify Networking

✓ Deploy Applications on Amazon EKS
