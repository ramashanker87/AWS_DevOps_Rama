# Day 20 Lab - Helm Charts and Scaling Strategies on Amazon EKS

## Module: Kubernetes with Amazon EKS

## Concepts Covered

- Helm charts
- Helm deployments
- Kubernetes scaling strategies
- Horizontal Pod Autoscaler (HPA)
- Cluster scaling with Amazon EKS managed node groups
- Application validation and cleanup

## AWS Resources Used

- Amazon EKS
- Helm
- Kubernetes Deployments
- Kubernetes Services
- Kubernetes Horizontal Pod Autoscaler
- Amazon EC2 Auto Scaling through EKS Managed Node Groups
- Amazon CloudWatch metrics
- AWS CLI

---

# Lab Objectives

By the end of this lab, participants will be able to:

- Install and verify Helm.
- Add and update Helm chart repositories.
- Deploy an application on Amazon EKS using Helm.
- Customize Helm values.
- Upgrade and rollback Helm releases.
- Scale Kubernetes workloads manually.
- Configure Horizontal Pod Autoscaler.
- Generate load to test autoscaling.
- Scale EKS managed node groups using AWS CLI.
- Validate application, pod, and node scaling.
- Clean up all lab resources.

---

# Lab Environment

Use the same EKS cluster created in previous labs.

```bash
export AWS_REGION=us-east-1
export AWS_PROFILE=devops
export CLUSTER_NAME=rama-eks-cluster
export NAMESPACE=day20
```

Explanation:

- `AWS_REGION` sets the AWS region where the EKS cluster exists.
- `AWS_PROFILE` tells AWS CLI which local credentials profile to use.
- `CLUSTER_NAME` is the EKS cluster name.
- `NAMESPACE` separates Day 20 resources from other Kubernetes resources.

---

# Prerequisites

## 1. Verify AWS CLI Identity

```bash
aws sts get-caller-identity \
  --profile $AWS_PROFILE
```

What this does:

This confirms that your AWS CLI is authenticated and using the correct AWS account.

Expected:

```text
Account: 386757865964
```

---

## 2. Verify EKS Cluster Status

```bash
aws eks describe-cluster \
  --name $CLUSTER_NAME \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query 'cluster.{Name:name,Status:status,Version:version,Endpoint:endpoint}' \
  --output table
```

What this does:

This checks whether the EKS cluster exists and is active.

Expected:

```text
Status: ACTIVE
```

---

## 3. Configure kubectl Access

```bash
aws eks update-kubeconfig \
  --region $AWS_REGION \
  --name $CLUSTER_NAME \
  --profile $AWS_PROFILE
```

What this does:

This updates your local kubeconfig file so `kubectl` can connect to the EKS cluster.

---

## 4. Verify Worker Nodes

```bash
kubectl get nodes
```

What this does:

This confirms that worker nodes are registered with the Kubernetes control plane.

Expected:

```text
STATUS: Ready
```

---

## 5. Create Namespace

```bash
kubectl create namespace $NAMESPACE
```

What this does:

This creates a separate namespace for all Day 20 lab resources.

If the namespace already exists, use:

```bash
kubectl get namespace $NAMESPACE
```

---

# Lab 1: Install and Verify Helm

## 1. Check Helm Version

```bash
helm version
```

What this does:

This verifies whether Helm is installed on your machine or CloudShell.

Expected:

```text
version.BuildInfo
```

---

## 2. Install Helm if Missing

For Linux or AWS CloudShell:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

What this does:

This downloads and installs Helm 3.

Verify again:

```bash
helm version
```

---

# Lab 2: Add Helm Repository

## 1. Add Bitnami Helm Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

What this does:

This adds the Bitnami Helm chart repository to your local Helm configuration.

---

## 2. Update Helm Repositories

```bash
helm repo update
```

What this does:

This downloads the latest chart index from all configured Helm repositories.

---

## 3. Search for nginx Chart

```bash
helm search repo nginx
```

What this does:

This searches available Helm repositories for charts related to nginx.

---

# Lab 3: Deploy Application Using Helm

## 1. Deploy nginx using Helm

```bash
helm install day20-nginx bitnami/nginx \
  --namespace $NAMESPACE
```

What this does:

This installs the Bitnami nginx Helm chart as a release named `day20-nginx` in the `day20` namespace.

---

## 2. Verify Helm Release

```bash
helm list -n $NAMESPACE
```

What this does:

This lists Helm releases in the Day 20 namespace.

Expected:

```text
day20-nginx
```

---

## 3. Verify Kubernetes Resources

```bash
kubectl get all -n $NAMESPACE
```

What this does:

This shows all Kubernetes resources created by the Helm chart, such as pods, services, deployments, and replica sets.

---

## 4. Verify Pods

```bash
kubectl get pods -n $NAMESPACE -o wide
```

What this does:

This shows nginx pods, their status, IP addresses, and the worker nodes where they are running.

Expected:

```text
STATUS: Running
READY: 1/1
```

---

# Lab 4: Inspect Helm Release

## 1. Get Helm Release Status

```bash
helm status day20-nginx -n $NAMESPACE
```

What this does:

This displays release status, chart details, namespace, and service information.

---

## 2. Get Helm Values

```bash
helm get values day20-nginx -n $NAMESPACE
```

What this does:

This shows custom values used during installation.

If no custom values were provided, it may return:

```text
USER-SUPPLIED VALUES:
null
```

---

## 3. Get Full Manifest

```bash
helm get manifest day20-nginx -n $NAMESPACE
```

What this does:

This shows the Kubernetes YAML generated by the Helm chart.

This is useful for understanding what Helm created behind the scenes.

---

# Lab 5: Customize Helm Deployment

## 1. Create Custom Values File

```bash
cat > day20-values.yaml <<'EOF'
replicaCount: 2

service:
  type: LoadBalancer
  ports:
    http: 80

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi
EOF
```

What this does:

This creates a values file to customize the nginx Helm deployment.

Explanation:

- `replicaCount: 2` creates two nginx pods.
- `service.type: LoadBalancer` exposes the application externally.
- `resources.requests` defines minimum CPU and memory required.
- `resources.limits` defines maximum CPU and memory allowed.

---

## 2. Upgrade Helm Release with Custom Values

```bash
helm upgrade day20-nginx bitnami/nginx \
  --namespace $NAMESPACE \
  -f day20-values.yaml
```

What this does:

This updates the existing Helm release using the new values.

---

## 3. Verify Upgrade

```bash
helm list -n $NAMESPACE
kubectl get pods -n $NAMESPACE
kubectl get svc -n $NAMESPACE
```

What this does:

These commands confirm the release is upgraded, pods are running, and a LoadBalancer service is created.

---

# Lab 6: Access Helm-Deployed Application

## 1. Get Load Balancer URL

```bash
export APP_URL=$(kubectl get svc day20-nginx \
  -n $NAMESPACE \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
```

What this does:

This extracts the external LoadBalancer DNS name created for the nginx service.

---

## 2. Print Application URL

```bash
echo http://$APP_URL
```

---

## 3. Test Application

```bash
curl http://$APP_URL
```

What this does:

This sends an HTTP request to the nginx application through the AWS Load Balancer.

Expected:

```text
Welcome to nginx
```

---

# Lab 7: Manual Pod Scaling

## 1. Check Current Replica Count

```bash
kubectl get deployment -n $NAMESPACE
```

What this does:

This shows the current number of desired and available replicas.

---

## 2. Scale Deployment to 4 Replicas

```bash
kubectl scale deployment day20-nginx \
  --replicas=4 \
  -n $NAMESPACE
```

What this does:

This manually changes the number of nginx pods to 4.

---

## 3. Verify Scaling

```bash
kubectl get pods -n $NAMESPACE -o wide
```

Expected:

```text
4 nginx pods Running
```

---

## 4. Reset Replicas using Helm

```bash
helm upgrade day20-nginx bitnami/nginx \
  --namespace $NAMESPACE \
  -f day20-values.yaml
```

What this does:

This restores the replica count from the Helm values file.

Important:

Manual `kubectl scale` changes can be overwritten by Helm upgrades.

---

# Lab 8: Helm Upgrade and Rollback

## 1. Update Values to 3 Replicas

```bash
cat > day20-values-v2.yaml <<'EOF'
replicaCount: 3

service:
  type: LoadBalancer
  ports:
    http: 80

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi
EOF
```

---

## 2. Upgrade Release

```bash
helm upgrade day20-nginx bitnami/nginx \
  --namespace $NAMESPACE \
  -f day20-values-v2.yaml
```

What this does:

This creates a new Helm release revision with 3 replicas.

---

## 3. Check Helm History

```bash
helm history day20-nginx -n $NAMESPACE
```

What this does:

This displays all revisions of the Helm release.

---

## 4. Roll Back to Previous Revision

```bash
helm rollback day20-nginx 1 -n $NAMESPACE
```

What this does:

This rolls the release back to revision 1.

Verify:

```bash
helm history day20-nginx -n $NAMESPACE
kubectl get pods -n $NAMESPACE
```

---

# Lab 9: Install Metrics Server

Horizontal Pod Autoscaler requires metrics.

## 1. Check if Metrics Server Exists

```bash
kubectl get deployment metrics-server -n kube-system
```

What this does:

This checks whether Metrics Server is already installed.

---

## 2. Install Metrics Server if Missing

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

What this does:

This installs Metrics Server, which collects CPU and memory usage metrics from Kubernetes nodes and pods.

---

## 3. Verify Metrics Server

```bash
kubectl get pods -n kube-system | grep metrics-server
```

---

## 4. Check Node Metrics

```bash
kubectl top nodes
```

---

## 5. Check Pod Metrics

```bash
kubectl top pods -n $NAMESPACE
```

What this does:

This verifies that Metrics Server is collecting metrics successfully.

---

# Lab 10: Configure Horizontal Pod Autoscaler

## 1. Create HPA for nginx Deployment

```bash
kubectl autoscale deployment day20-nginx \
  --cpu-percent=50 \
  --min=2 \
  --max=6 \
  -n $NAMESPACE
```

What this does:

This creates an HPA that keeps average CPU usage around 50%.

Explanation:

- `--min=2` means minimum 2 pods.
- `--max=6` means maximum 6 pods.
- `--cpu-percent=50` means scale up if CPU usage goes above 50%.

---

## 2. Verify HPA

```bash
kubectl get hpa -n $NAMESPACE
```

Expected:

```text
NAME          REFERENCE                MINPODS   MAXPODS
day20-nginx   Deployment/day20-nginx   2         6
```

---

## 3. Describe HPA

```bash
kubectl describe hpa day20-nginx -n $NAMESPACE
```

What this does:

This shows scaling rules, current metrics, and recent HPA events.

---

# Lab 11: Generate Load for Autoscaling

## 1. Start Load Generator Pod

```bash
kubectl run load-generator \
  --image=busybox \
  --restart=Never \
  -n $NAMESPACE \
  -- /bin/sh -c "while true; do wget -q -O- http://day20-nginx; done"
```

What this does:

This creates a pod that continuously sends traffic to the nginx service.

---

## 2. Watch HPA

```bash
kubectl get hpa -n $NAMESPACE -w
```

What this does:

This watches the autoscaler and shows whether the replica count changes.

Stop watching with:

```text
CTRL + C
```

---

## 3. Watch Pods

```bash
kubectl get pods -n $NAMESPACE -w
```

Expected:

Pods may increase up to 6 if CPU utilization crosses the threshold.

---

## 4. Stop Load Generator

```bash
kubectl delete pod load-generator -n $NAMESPACE
```

What this does:

This stops the artificial traffic generator.

---

# Lab 12: EKS Managed Node Group Scaling

Pod autoscaling increases pods, but node group scaling controls worker node capacity.

## 1. List Node Groups

```bash
aws eks list-nodegroups \
  --cluster-name $CLUSTER_NAME \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This lists EKS managed node groups for the cluster.

---

## 2. Describe Node Group

```bash
aws eks describe-nodegroup \
  --cluster-name $CLUSTER_NAME \
  --nodegroup-name rama-eks-nodegroup \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query 'nodegroup.{Name:nodegroupName,Status:status,Scaling:scalingConfig,InstanceTypes:instanceTypes}' \
  --output table
```

What this does:

This displays node group status, desired capacity, min/max size, and instance type.

---

## 3. Scale Node Group to 3 Nodes

```bash
aws eks update-nodegroup-config \
  --cluster-name $CLUSTER_NAME \
  --nodegroup-name rama-eks-nodegroup \
  --scaling-config minSize=1,maxSize=4,desiredSize=3 \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This asks EKS to update the managed node group desired size to 3 nodes.

---

## 4. Verify Nodes

```bash
kubectl get nodes
```

Expected:

A new node joins the cluster.

---

## 5. Scale Node Group Back to 2 Nodes

```bash
aws eks update-nodegroup-config \
  --cluster-name $CLUSTER_NAME \
  --nodegroup-name rama-eks-nodegroup \
  --scaling-config minSize=1,maxSize=4,desiredSize=2 \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This reduces desired worker nodes back to 2.

---

# Lab 13: Scaling Strategy Review

## Manual Scaling

Manual scaling is useful for planned events or testing.

Example:

```bash
kubectl scale deployment day20-nginx \
  --replicas=3 \
  -n $NAMESPACE
```

## Horizontal Pod Autoscaler

HPA automatically scales pods based on CPU or memory metrics.

Example:

```bash
kubectl get hpa -n $NAMESPACE
```

## Node Group Scaling

Node group scaling changes worker node capacity.

Example:

```bash
aws eks update-nodegroup-config \
  --cluster-name $CLUSTER_NAME \
  --nodegroup-name rama-eks-nodegroup \
  --scaling-config minSize=1,maxSize=4,desiredSize=3 \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

## Cluster Autoscaler

Cluster Autoscaler automatically adjusts node count when pods cannot be scheduled due to insufficient capacity.

This lab demonstrates node group scaling manually with AWS CLI. In production, Cluster Autoscaler or Karpenter is commonly used for automatic node scaling.

---

# Lab 14: Validate Final Resources

## Helm Releases

```bash
helm list -n $NAMESPACE
```

## Deployments

```bash
kubectl get deployments -n $NAMESPACE
```

## Pods

```bash
kubectl get pods -n $NAMESPACE -o wide
```

## Services

```bash
kubectl get svc -n $NAMESPACE
```

## HPA

```bash
kubectl get hpa -n $NAMESPACE
```

## Nodes

```bash
kubectl get nodes
```

---

# Lab 15: Troubleshooting

## Helm Release Not Found

```bash
helm list -A
```

Check whether the release was installed in a different namespace.

---

## Pods Pending

```bash
kubectl describe pod <pod-name> -n $NAMESPACE
kubectl get events -n $NAMESPACE --sort-by=.metadata.creationTimestamp
```

Common causes:

- Not enough node capacity
- Image pull issue
- Resource requests too high

---

## HPA Shows Unknown

```bash
kubectl get pods -n kube-system | grep metrics-server
kubectl top nodes
kubectl top pods -n $NAMESPACE
```

Common cause:

Metrics Server is not installed or not ready.

---

## LoadBalancer Pending

```bash
kubectl describe svc day20-nginx -n $NAMESPACE
```

Common causes:

- Subnet tags missing
- AWS cloud provider issue
- Security group restrictions

---

# Lab 16: Cleanup

## 1. Delete Load Generator

```bash
kubectl delete pod load-generator -n $NAMESPACE --ignore-not-found=true
```

## 2. Delete HPA

```bash
kubectl delete hpa day20-nginx -n $NAMESPACE --ignore-not-found=true
```

## 3. Uninstall Helm Release

```bash
helm uninstall day20-nginx -n $NAMESPACE
```

What this does:

This removes all Kubernetes resources managed by the Helm release.

---

## 4. Delete Namespace

```bash
kubectl delete namespace $NAMESPACE
```

---

## 5. Reset Node Group Size

```bash
aws eks update-nodegroup-config \
  --cluster-name $CLUSTER_NAME \
  --nodegroup-name rama-eks-nodegroup \
  --scaling-config minSize=1,maxSize=3,desiredSize=2 \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This returns the node group to the original lab size.

---

# Challenge Exercise

Implement the following:

1. Deploy an application using Helm.
2. Customize the Helm chart using a values file.
3. Upgrade the Helm release.
4. Roll back the Helm release.
5. Configure Horizontal Pod Autoscaler.
6. Generate load and observe scaling.
7. Scale EKS managed node group using AWS CLI.
8. Document scaling behavior.

---

# Lab Deliverables

Submit:

- Helm version output
- Helm repo list output
- Helm release screenshot
- `kubectl get pods -n day20` output
- LoadBalancer application access screenshot
- HPA screenshot
- Node group scaling screenshot
- Helm rollback screenshot
- Cleanup verification screenshot

---

# Expected Learning Outcomes

After completing this lab, participants should be able to:

- Understand Helm chart structure and purpose.
- Deploy applications on EKS using Helm.
- Customize Helm deployments using values files.
- Perform Helm upgrades and rollbacks.
- Understand manual pod scaling.
- Configure Kubernetes Horizontal Pod Autoscaler.
- Generate load to test autoscaling.
- Scale Amazon EKS managed node groups.
- Understand the difference between pod scaling and node scaling.
- Clean up Helm and autoscaling lab resources.
