# Day 19 – 11-Jun-2026 (Thursday)

# Module 4 – Kubernetes with Amazon EKS

# Lab: Persistent Storage & Ingress Configuration

## AWS Resources Used

* Amazon EKS
* Amazon EBS
* Amazon EFS
* Amazon Route53
* AWS Load Balancer Controller
* IAM
* EC2 Security Groups

---

# Lab Objectives

Participants will:

* Create Persistent Volumes (PV).
* Create Persistent Volume Claims (PVC).
* Use Amazon EBS storage.
* Configure Amazon EFS shared storage.
* Deploy applications with persistent storage.
* Install and configure AWS Load Balancer Controller.
* Configure Kubernetes Ingress.
* Integrate Route53 DNS.
* Verify application access.

---

# Lab Environment

```bash
export AWS_REGION=us-east-1
export CLUSTER_NAME=rama-eks-cluster
export AWS_PROFILE=devops
export NAMESPACE=day19
```

---

# Prerequisites

Existing EKS Cluster must be running.

Verify:

```bash
aws eks describe-cluster \
  --name $CLUSTER_NAME \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query 'cluster.status'
```

Expected:

```text
ACTIVE
```

Update kubeconfig:

```bash
aws eks update-kubeconfig \
  --region $AWS_REGION \
  --name $CLUSTER_NAME \
  --profile $AWS_PROFILE
```

Verify Worker Nodes:

```bash
kubectl get nodes
```

Expected:

```text
STATUS: Ready
```

Create Namespace:

```bash
kubectl create namespace $NAMESPACE
```

---

# Lab 1: Verify Storage Classes

List Available Storage Classes:

```bash
kubectl get storageclass
```

Expected:

```text
gp2
gp3
```

Identify Default Storage Class:

```bash
kubectl get storageclass
```

Look for:

```text
(default)
```

---

# Lab 2: Create Persistent Volume Claim (Amazon EBS)

Create File:

```bash
cat > pvc.yaml <<'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
  namespace: day19
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
EOF
```

Deploy PVC:

```bash
kubectl apply -f pvc.yaml
```

Verify:

```bash
kubectl get pvc -n day19
```

Expected:

```text
STATUS: Bound
```

Describe PVC:

```bash
kubectl describe pvc app-pvc -n day19
```

---

# Lab 3: Deploy Application with PVC

Create Deployment:

```bash
cat > deployment-pvc.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-storage
  namespace: day19
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
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: app-storage
      volumes:
      - name: app-storage
        persistentVolumeClaim:
          claimName: app-pvc
EOF
```

Deploy:

```bash
kubectl apply -f deployment-pvc.yaml
```

Verify:

```bash
kubectl get pods -n day19
```

---

# Lab 4: Verify Amazon EBS Volume

Find Persistent Volume:

```bash
kubectl get pv
```

Describe Volume:

```bash
kubectl describe pv
```

Get EBS Volume ID:

```bash
kubectl describe pv
```

Verify from AWS:

```bash
aws ec2 describe-volumes \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

Verify:

* New EBS volume created
* Volume status = in-use
* Attached to EKS worker node

---

# Lab 5: Create Amazon EFS File System

Create EFS:

```bash
aws efs create-file-system \
  --creation-token day19-efs \
  --performance-mode generalPurpose \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

Capture File System ID:

```bash
export FILE_SYSTEM_ID=<efs-id>
```

Verify:

```bash
aws efs describe-file-systems \
  --file-system-id $FILE_SYSTEM_ID \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

---

# Lab 6: Create EFS Mount Targets

Get VPC:

```bash
aws eks describe-cluster \
  --name $CLUSTER_NAME \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query "cluster.resourcesVpcConfig.vpcId"
```

Create Mount Targets in each subnet:

```bash
aws efs create-mount-target \
  --file-system-id $FILE_SYSTEM_ID \
  --subnet-id <subnet-id> \
  --security-groups <security-group-id> \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

Repeat for all EKS subnets.

Verify:

```bash
aws efs describe-mount-targets \
  --file-system-id $FILE_SYSTEM_ID \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

---

# Lab 7: Install Amazon EFS CSI Driver

Associate IAM OIDC Provider:

```bash
eksctl utils associate-iam-oidc-provider \
  --region=$AWS_REGION \
  --cluster=$CLUSTER_NAME \
  --approve
```

Install EFS CSI Driver Add-on:

```bash
aws eks create-addon \
  --cluster-name $CLUSTER_NAME \
  --addon-name aws-efs-csi-driver \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

Verify:

```bash
kubectl get pods -n kube-system
```

Expected:

```text
efs-csi-controller
efs-csi-node
```

---

# Lab 8: Create EFS Storage Class

Create Storage Class:

```bash
cat > efs-storageclass.yaml <<'EOF'
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: FILE_SYSTEM_ID
  directoryPerms: "700"
EOF
```

Replace:

```text
FILE_SYSTEM_ID
```

Deploy:

```bash
kubectl apply -f efs-storageclass.yaml
```

Verify:

```bash
kubectl get storageclass
```

---

# Lab 9: Create EFS Persistent Volume Claim

Create PVC:

```bash
cat > efs-pvc.yaml <<'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-pvc
  namespace: day19
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi
EOF
```

Deploy:

```bash
kubectl apply -f efs-pvc.yaml
```

Verify:

```bash
kubectl get pvc -n day19
```

Expected:

```text
STATUS: Bound
```

---

# Lab 10: Deploy Shared Storage Application

Create Deployment:

```bash
cat > shared-app.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shared-nginx
  namespace: day19
spec:
  replicas: 2
  selector:
    matchLabels:
      app: shared-nginx
  template:
    metadata:
      labels:
        app: shared-nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: shared-storage
          mountPath: /usr/share/nginx/html
      volumes:
      - name: shared-storage
        persistentVolumeClaim:
          claimName: efs-pvc
EOF
```

Deploy:

```bash
kubectl apply -f shared-app.yaml
```

Verify:

```bash
kubectl get pods -n day19
```

Confirm both pods share same storage.

---

# Lab 11: Create Service

Expose Deployment:

```bash
kubectl expose deployment nginx-storage \
  --type=ClusterIP \
  --port=80 \
  --namespace day19
```

Verify:

```bash
kubectl get svc -n day19
```

---

# Lab 12: Install AWS Load Balancer Controller

Download IAM Policy:

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

Create IAM Policy:

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

Install Controller using Helm:

```bash
helm repo add eks https://aws.github.io/eks-charts

helm repo update
```

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=$CLUSTER_NAME
```

Verify:

```bash
kubectl get deployment -n kube-system
```

Expected:

```text
aws-load-balancer-controller
```

---

# Lab 13: Create Ingress

Create Ingress:

```bash
cat > ingress.yaml <<'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: day19
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
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
EOF
```

Deploy:

```bash
kubectl apply -f ingress.yaml
```

Verify:

```bash
kubectl get ingress -n day19
```

Capture ALB hostname.

---

# Lab 14: Configure Route53

List Hosted Zones:

```bash
aws route53 list-hosted-zones \
  --profile $AWS_PROFILE
```

Create Alias Record:

Example:

```text
app.training.example.com
```

Point DNS to:

```text
ALB-DNS-NAME
```

Verify:

```bash
nslookup app.training.example.com
```

---

# Lab 15: Validate Access

Open Browser:

```text
http://app.training.example.com
```

Or test:

```bash
curl http://app.training.example.com
```

Verify application availability.

---

# Lab 16: Review Security Configuration

Verify IAM Roles:

```bash
aws iam list-roles \
  --profile $AWS_PROFILE
```

Verify Security Groups:

```bash
aws ec2 describe-security-groups \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

Verify Ingress:

```bash
kubectl describe ingress app-ingress -n day19
```

---

# Lab 17: Cleanup

Delete Ingress:

```bash
kubectl delete ingress app-ingress -n day19
```

Delete Services:

```bash
kubectl delete svc nginx-storage -n day19
```

Delete Deployments:

```bash
kubectl delete deployment nginx-storage -n day19
kubectl delete deployment shared-nginx -n day19
```

Delete PVCs:

```bash
kubectl delete pvc app-pvc -n day19
kubectl delete pvc efs-pvc -n day19
```

Delete EFS:

```bash
aws efs delete-file-system \
  --file-system-id $FILE_SYSTEM_ID \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

Verify Cleanup:

```bash
kubectl get all -n day19
```

---

# Challenge Exercise

Implement:

1. Amazon EBS Persistent Storage
2. Amazon EFS Shared Storage
3. AWS Load Balancer Controller
4. Kubernetes Ingress
5. Route53 DNS Integration
6. Secure Application Access

Document:

* Architecture Diagram
* AWS CLI Commands
* Screenshots
* Validation Outputs

---

# Lab Deliverables

Submit:

* PVC Screenshot
* EBS Volume Screenshot
* EFS File System Screenshot
* EFS CSI Driver Screenshot
* Ingress Screenshot
* Route53 Record Screenshot
* Application Access Screenshot
* Security Review Screenshot

---

# Expected Learning Outcomes

✓ Configure Persistent Storage with Amazon EBS

✓ Configure Shared Storage with Amazon EFS

✓ Deploy Stateful Applications on Amazon EKS

✓ Install AWS Load Balancer Controller

✓ Configure Kubernetes Ingress

✓ Integrate Route53 DNS

✓ Secure Application Access

✓ Manage Persistent Kubernetes Workloads on Amazon EKS
