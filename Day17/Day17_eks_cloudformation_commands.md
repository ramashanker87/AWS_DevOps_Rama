# Day 17 EKS CloudFormation Lab Commands

## 1. Prerequisites

Required tools:

```bash
aws --version
kubectl version --client
```

Verify AWS access:

```bash
aws sts get-caller-identity
```

You need IAM permissions for:

- CloudFormation
- EKS
- EC2
- IAM
- CloudWatch Logs

---

## 2. Deploy the EKS Lab Stack

```bash
aws cloudformation deploy \
  --template-file Day17_eks_lab.yml \
  --stack-name day17-eks-lab \
  --capabilities CAPABILITY_NAMED_IAM \
  --region eu-north-1 \
  --parameter-overrides \
    ProjectName=day17-eks \
    KubernetesVersion=1.29 \
    NodeInstanceType=t3.medium \
    DesiredNodeCount=2 \
    MinNodeCount=1 \
    MaxNodeCount=3
```

EKS cluster creation can take several minutes.

---

## 3. Configure kubectl

```bash
aws eks update-kubeconfig \
  --region eu-north-1 \
  --name day17-eks-cluster
```

Verify nodes:

```bash
kubectl get nodes
```

---

## 4. Create Namespace

```bash
kubectl create namespace training
```

---

## 5. Create Deployment

```bash
kubectl create deployment hello-app \
  --image=nginx \
  --replicas=2 \
  -n training
```

Verify:

```bash
kubectl get deployments -n training
kubectl get pods -n training
```

---

## 6. Expose Deployment as LoadBalancer Service

```bash
kubectl expose deployment hello-app \
  --type=LoadBalancer \
  --port=80 \
  --target-port=80 \
  -n training
```

Verify:

```bash
kubectl get svc -n training
```

Wait until the EXTERNAL-IP or DNS name is available.

---

## 7. Scale Deployment

```bash
kubectl scale deployment hello-app \
  --replicas=4 \
  -n training
```

Verify:

```bash
kubectl get pods -n training
```

---

## 8. View Logs

```bash
kubectl get pods -n training
kubectl logs <pod-name> -n training
```

---

## 9. Cleanup Kubernetes Resources

```bash
kubectl delete namespace training
```

---

## 10. Delete CloudFormation Stack

```bash
aws cloudformation delete-stack \
  --stack-name day17-eks-lab \
  --region eu-north-1
```

Check delete status:

```bash
aws cloudformation describe-stacks \
  --stack-name day17-eks-lab \
  --region eu-north-1
```

Note: Delete the Kubernetes LoadBalancer service before deleting the stack to avoid leftover ELB resources.
