# Day 18 – 10-Jun-2026 (Wednesday)
# Module 4 – Kubernetes with Amazon EKS

# Lab: EKS Provisioning & Networking Setup

## AWS Resources Used

- Amazon EKS
- Amazon VPC
- IAM
- CloudFormation

---

# Lab Objectives

Participants will:

- Provision EKS using CloudFormation.
- Configure networking resources.
- Verify cluster creation.
- Configure kubectl.
- Validate worker nodes.
- Test Kubernetes networking.
- Verify IAM integration.

---

# Prerequisites

Install Tools:

```bash
aws --version
kubectl version --client
```

Verify Login:

```bash
aws sts get-caller-identity
```

---

# Lab 1: Review CloudFormation Template

Resources Included:

- VPC
- Public Subnets
- Private Subnets
- NAT Gateway
- IAM Roles
- EKS Cluster
- Managed Node Group

Review Template:

Day17_eks_lab.yml

---

# Lab 2: Deploy CloudFormation Stack

```bash
aws cloudformation deploy --template-file Day17_eks_lab.yml --stack-name day18-eks-lab --capabilities CAPABILITY_NAMED_IAM --region eu-north-1
```

Monitor:

CloudFormation Console

Expected:

CREATE_COMPLETE

---

# Lab 3: Verify VPC Resources

Navigate:

VPC Console

Verify:

- VPC
- Public Subnets
- Private Subnets
- NAT Gateway
- Route Tables

Document IDs.

---

# Lab 4: Verify IAM Roles

IAM Console

Verify:

- EKS Cluster Role
- EKS Node Role

Check attached policies.

---

# Lab 5: Verify EKS Cluster

List Clusters:

```bash
aws eks list-clusters
```

Describe Cluster:

```bash
aws eks describe-cluster --name day17-eks-cluster
```

Verify Status:

ACTIVE

---

# Lab 6: Configure kubectl

```bash
aws eks update-kubeconfig --region eu-north-1 --name day17-eks-cluster
```

Verify:

```bash
kubectl get nodes
```

Expected:

Worker Nodes Ready

---

# Lab 7: Explore Cluster Resources

```bash
kubectl get nodes
```

```bash
kubectl get namespaces
```

```bash
kubectl get pods -A
```

---

# Lab 8: Validate Networking

Deploy Test Pod:

```bash
kubectl run nginx-test --image=nginx
```

Verify:

```bash
kubectl get pods
```

Check Pod IP:

```bash
kubectl get pod nginx-test -o wide
```

Record IP Address.

---

# Lab 9: Verify Service Networking

Create Service:

```bash
kubectl expose pod nginx-test --type=ClusterIP --port=80
```

Verify:

```bash
kubectl get svc
```

Record Service IP.

---

# Lab 10: Test Internal Connectivity

Launch Debug Pod:

```bash
kubectl run test-shell -it --rm --image=busybox -- sh
```

Inside Pod:

```bash
wget -qO- http://nginx-test
```

Verify response.

---

# Lab 11: Verify Node Group

AWS Console:

EKS → Node Groups

Verify:

- Desired Nodes
- Running Nodes
- Instance Type

---

# Lab 12: Scale Node Group

Increase Desired Nodes:

2 → 3

Verify:

```bash
kubectl get nodes
```

Observe additional node.

---

# Lab 13: CloudWatch Monitoring

Review:

- Cluster Logs
- Node Metrics
- Control Plane Logs

Verify:

- CPU Metrics
- Memory Metrics

---

# Lab 14: Cleanup Resources

Delete Test Resources:

```bash
kubectl delete pod nginx-test
```

Delete Stack:

```bash
aws cloudformation delete-stack --stack-name day18-eks-lab
```

Verify deletion.

---

# Challenge Exercise

Provision:

1. New EKS Cluster
2. New VPC
3. Managed Node Group
4. Configure kubectl
5. Deploy test application
6. Validate networking

Document:

- Commands
- Screenshots
- Outputs

---

# Lab Deliverables

Submit:

- CloudFormation Stack Screenshot
- VPC Architecture Screenshot
- EKS Cluster Screenshot
- Node Group Screenshot
- kubectl Output
- Networking Validation Screenshot

---

# Expected Learning Outcomes

✓ Provision EKS Clusters

✓ Configure VPC Networking

✓ Validate IAM Roles

✓ Configure kubectl

✓ Verify Cluster Health

✓ Test Kubernetes Networking

✓ Manage EKS Infrastructure Using CloudFormation
