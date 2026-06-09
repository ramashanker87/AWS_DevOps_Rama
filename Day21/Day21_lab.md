# Day 21 Lab - GitOps Principles and ArgoCD on Amazon EKS

## Module: Kubernetes with Amazon EKS

## Topics Covered

- GitOps principles
- ArgoCD architecture
- ArgoCD setup on Amazon EKS
- Application deployment using ArgoCD
- Git repository based deployment workflow
- IAM and AWS CLI validation

## AWS Resources Used

- Amazon EKS
- ArgoCD
- IAM
- Kubernetes Deployments
- Kubernetes Services
- Kubernetes Namespaces
- AWS CLI
- kubectl

---

# Lab Objectives

By the end of this lab, participants will be able to:

- Understand GitOps principles.
- Understand ArgoCD architecture and components.
- Install ArgoCD on Amazon EKS.
- Access the ArgoCD web UI.
- Retrieve the initial ArgoCD admin password.
- Configure ArgoCD CLI.
- Deploy an application using ArgoCD.
- Sync Kubernetes manifests from Git to EKS.
- Verify application deployment.
- Understand reconciliation and drift detection.
- Clean up ArgoCD and application resources.

---

# Lab Environment

Use the existing EKS cluster from previous labs.

```bash
export AWS_REGION=us-east-1
export AWS_PROFILE=devops
export CLUSTER_NAME=rama-eks-cluster
export ARGOCD_NAMESPACE=argocd
export APP_NAMESPACE=day21
export APP_NAME=day21-nginx
```

Explanation:

- `AWS_REGION` is the AWS region where the EKS cluster is running.
- `AWS_PROFILE` is the AWS CLI profile used for authentication.
- `CLUSTER_NAME` is the EKS cluster name.
- `ARGOCD_NAMESPACE` is where ArgoCD will be installed.
- `APP_NAMESPACE` is where the sample application will run.
- `APP_NAME` is the name of the sample application deployed by ArgoCD.

---

# GitOps Principles

GitOps is a deployment and operations model where Git is the source of truth for infrastructure and application configuration.

## Key GitOps Principles

1. **Declarative Configuration**

   Kubernetes manifests, Helm values, and infrastructure definitions are stored as code.

2. **Git as Source of Truth**

   Desired state is stored in Git.

3. **Automated Reconciliation**

   A GitOps tool continuously compares the live cluster state with the Git state.

4. **Pull-Based Deployment**

   The cluster pulls configuration from Git instead of external systems pushing changes into the cluster.

5. **Auditability**

   Every change is tracked through Git commits, pull requests, and history.

6. **Rollback**

   Previous application states can be restored by reverting Git commits.

---

# ArgoCD Architecture

ArgoCD is a declarative GitOps continuous delivery tool for Kubernetes.

## Main ArgoCD Components

- **argocd-server**

  Provides the API server and web UI.

- **argocd-repo-server**

  Connects to Git repositories and generates Kubernetes manifests.

- **argocd-application-controller**

  Compares the desired state in Git with the live state in the cluster and performs sync operations.

- **argocd-dex-server**

  Provides authentication integration with identity providers.

- **argocd-redis**

  Provides caching for ArgoCD components.

## ArgoCD Workflow

1. User commits Kubernetes manifests to Git.
2. ArgoCD watches the Git repository.
3. ArgoCD compares Git desired state with the live EKS cluster state.
4. If drift is detected, ArgoCD marks the application as `OutOfSync`.
5. User or automated policy syncs the application.
6. ArgoCD applies the manifests to Kubernetes.
7. Application becomes `Synced` and `Healthy`.

---

# Prerequisites

## 1. Verify AWS CLI Identity

```bash
aws sts get-caller-identity \
  --profile $AWS_PROFILE
```

What this does:

This verifies that AWS CLI is configured and using the correct AWS account.

Expected output includes:

```text
Account
Arn
UserId
```

---

## 2. Verify EKS Cluster

```bash
aws eks describe-cluster \
  --name $CLUSTER_NAME \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query 'cluster.{Name:name,Status:status,Version:version,Endpoint:endpoint}' \
  --output table
```

What this does:

This checks that the EKS cluster exists and is active.

Expected:

```text
Status: ACTIVE
```

---

## 3. Configure kubectl

```bash
aws eks update-kubeconfig \
  --region $AWS_REGION \
  --name $CLUSTER_NAME \
  --profile $AWS_PROFILE
```

What this does:

This updates kubeconfig so `kubectl` can connect to the EKS cluster.

---

## 4. Verify Worker Nodes

```bash
kubectl get nodes
```

What this does:

This confirms worker nodes are available.

Expected:

```text
STATUS: Ready
```

---

## 5. Verify Current Kubernetes Context

```bash
kubectl config current-context
```

What this does:

This shows the Kubernetes cluster context currently used by kubectl.

---

# Lab 1: Create Namespaces

## 1. Create ArgoCD Namespace

```bash
kubectl create namespace $ARGOCD_NAMESPACE
```

What this does:

This creates a namespace where ArgoCD components will be installed.

Verify:

```bash
kubectl get namespace $ARGOCD_NAMESPACE
```

---

## 2. Create Application Namespace

```bash
kubectl create namespace $APP_NAMESPACE
```

What this does:

This creates a namespace where the sample application will be deployed.

Verify:

```bash
kubectl get namespace $APP_NAMESPACE
```

---

# Lab 2: Install ArgoCD on EKS

## 1. Install ArgoCD Manifests

```bash
kubectl apply -n $ARGOCD_NAMESPACE \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

What this does:

This installs ArgoCD core components into the `argocd` namespace.

Resources created include:

- Deployments
- Services
- ConfigMaps
- Secrets
- ServiceAccounts
- RBAC permissions

---

## 2. Verify ArgoCD Pods

```bash
kubectl get pods -n $ARGOCD_NAMESPACE
```

What this does:

This checks whether ArgoCD pods are running.

Expected pods:

```text
argocd-application-controller
argocd-applicationset-controller
argocd-dex-server
argocd-notifications-controller
argocd-redis
argocd-repo-server
argocd-server
```

Expected status:

```text
Running
```

---

## 3. Watch ArgoCD Pods Until Ready

```bash
kubectl get pods -n $ARGOCD_NAMESPACE -w
```

Stop watching with:

```text
CTRL + C
```

---

## 4. Verify ArgoCD Services

```bash
kubectl get svc -n $ARGOCD_NAMESPACE
```

What this does:

This lists services created by ArgoCD.

Important service:

```text
argocd-server
```

---

# Lab 3: Access ArgoCD UI

There are multiple ways to access ArgoCD.

For lab training, the simplest method is port-forwarding.

## Option A: Port Forward ArgoCD Server

```bash
kubectl port-forward svc/argocd-server \
  -n $ARGOCD_NAMESPACE \
  8080:443
```

What this does:

This forwards local port `8080` to the ArgoCD server service on port `443`.

Open in browser:

```text
https://localhost:8080
```

Because ArgoCD uses a self-signed certificate in this setup, your browser may show a certificate warning. Accept the warning for lab use.

---

## Option B: Change ArgoCD Server Service to LoadBalancer

```bash
kubectl patch svc argocd-server \
  -n $ARGOCD_NAMESPACE \
  -p '{"spec": {"type": "LoadBalancer"}}'
```

What this does:

This exposes ArgoCD using an AWS Load Balancer.

Verify:

```bash
kubectl get svc argocd-server -n $ARGOCD_NAMESPACE
```

Wait until the `EXTERNAL-IP` or hostname appears.

Get URL:

```bash
kubectl get svc argocd-server \
  -n $ARGOCD_NAMESPACE \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

Use HTTPS:

```text
https://<load-balancer-hostname>
```

Important:

For production environments, use Ingress, TLS certificates, SSO, and restricted security groups instead of exposing ArgoCD openly.

---

# Lab 4: Get ArgoCD Admin Password

## 1. Get Initial Admin Password

```bash
kubectl -n $ARGOCD_NAMESPACE get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
echo
```

What this does:

This retrieves the initial password for the `admin` user.

Login username:

```text
admin
```

---

## 2. Login to ArgoCD UI

Open:

```text
https://localhost:8080
```

or, if using LoadBalancer:

```text
https://<argocd-load-balancer-hostname>
```

Credentials:

```text
Username: admin
Password: <password from previous command>
```

---

# Lab 5: Install ArgoCD CLI

## 1. Download ArgoCD CLI on Linux

```bash
curl -sSL -o argocd-linux-amd64 \
  https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
```

What this does:

This downloads the latest Linux AMD64 ArgoCD CLI binary.

---

## 2. Install ArgoCD CLI

```bash
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

What this does:

This installs the ArgoCD CLI into `/usr/local/bin`.

---

## 3. Verify ArgoCD CLI

```bash
argocd version --client
```

What this does:

This confirms the ArgoCD CLI is installed.

---

# Lab 6: Login to ArgoCD CLI

## If Using Port Forward

Keep this command running in another terminal:

```bash
kubectl port-forward svc/argocd-server \
  -n $ARGOCD_NAMESPACE \
  8080:443
```

Login:

```bash
argocd login localhost:8080 \
  --username admin \
  --password <ARGOCD_ADMIN_PASSWORD> \
  --insecure
```

What this does:

This logs the ArgoCD CLI into the local ArgoCD API server.

---

## If Using LoadBalancer

```bash
argocd login <argocd-load-balancer-hostname> \
  --username admin \
  --password <ARGOCD_ADMIN_PASSWORD> \
  --insecure
```

---

# Lab 7: Prepare Sample Git Repository

ArgoCD deploys applications from Git.

You can use your own GitHub repository or a sample public repository.

## Option A: Use a Public Example Repository

Example repository:

```text
https://github.com/argoproj/argocd-example-apps.git
```

Example application path:

```text
guestbook
```

This is useful for quick testing.

---

## Option B: Create Your Own Git Repository

Create local folder:

```bash
mkdir day21-gitops-app
cd day21-gitops-app
```

Create Kubernetes manifest folder:

```bash
mkdir manifests
```

Create deployment:

```bash
cat > manifests/deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: day21-nginx
  namespace: day21
spec:
  replicas: 2
  selector:
    matchLabels:
      app: day21-nginx
  template:
    metadata:
      labels:
        app: day21-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
EOF
```

Create service:

```bash
cat > manifests/service.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: day21-nginx
  namespace: day21
spec:
  type: ClusterIP
  selector:
    app: day21-nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
EOF
```

Initialize Git:

```bash
git init
git add .
git commit -m "Add Day21 GitOps nginx app"
```

Push to GitHub:

```bash
git remote add origin <your-git-repository-url>
git branch -M main
git push -u origin main
```

What this does:

This stores Kubernetes manifests in Git so ArgoCD can deploy them.

---

# Lab 8: Deploy Application with ArgoCD CLI

## Option A: Deploy Public Guestbook App

```bash
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace $APP_NAMESPACE
```

What this does:

This creates an ArgoCD Application named `guestbook`.

Explanation:

- `--repo` points to the Git repository.
- `--path` points to the manifest folder inside the repository.
- `--dest-server` points to the Kubernetes API server in the same cluster.
- `--dest-namespace` tells ArgoCD where to deploy the app.

---

## 2. Sync Application

```bash
argocd app sync guestbook
```

What this does:

This applies the Git manifests to the EKS cluster.

---

## 3. Check Application Status

```bash
argocd app get guestbook
```

Expected:

```text
Sync Status: Synced
Health Status: Healthy
```

---

# Lab 9: Deploy Application Using ArgoCD YAML

Instead of using CLI flags, create an ArgoCD Application manifest.

## 1. Create Application Manifest

```bash
cat > day21-argocd-app.yaml <<'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: day21-nginx
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: day21
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF
```

What this does:

This creates an ArgoCD Application using Kubernetes YAML.

Explanation:

- `repoURL` is the Git repository.
- `targetRevision` is the Git branch or commit.
- `path` is the folder containing manifests.
- `destination.namespace` is the Kubernetes namespace.
- `automated.prune` removes resources deleted from Git.
- `automated.selfHeal` corrects manual drift in the cluster.

---

## 2. Apply Application Manifest

```bash
kubectl apply -f day21-argocd-app.yaml
```

What this does:

This creates an ArgoCD Application custom resource.

---

## 3. Verify Application

```bash
kubectl get applications -n $ARGOCD_NAMESPACE
```

or:

```bash
argocd app list
```

---

# Lab 10: Verify Application Deployment

## 1. Check Pods

```bash
kubectl get pods -n $APP_NAMESPACE
```

Expected:

```text
Running
```

---

## 2. Check Services

```bash
kubectl get svc -n $APP_NAMESPACE
```

---

## 3. Check All Resources

```bash
kubectl get all -n $APP_NAMESPACE
```

---

## 4. Describe Application

```bash
argocd app get day21-nginx
```

or, for guestbook:

```bash
argocd app get guestbook
```

---

# Lab 11: Access the Deployed Application

If the sample app creates a ClusterIP service, use port-forwarding.

## 1. Check Service Name

```bash
kubectl get svc -n $APP_NAMESPACE
```

---

## 2. Port Forward Service

For guestbook:

```bash
kubectl port-forward svc/guestbook-ui \
  -n $APP_NAMESPACE \
  8081:80
```

Open:

```text
http://localhost:8081
```

For nginx app:

```bash
kubectl port-forward svc/day21-nginx \
  -n $APP_NAMESPACE \
  8081:80
```

Open:

```text
http://localhost:8081
```

---

# Lab 12: Test GitOps Drift Detection

## 1. Manually Scale Deployment

For guestbook:

```bash
kubectl scale deployment guestbook-ui \
  --replicas=1 \
  -n $APP_NAMESPACE
```

For nginx:

```bash
kubectl scale deployment day21-nginx \
  --replicas=1 \
  -n $APP_NAMESPACE
```

What this does:

This changes the live cluster state manually.

---

## 2. Check ArgoCD Status

```bash
argocd app list
```

or:

```bash
argocd app get day21-nginx
```

What this does:

This shows whether ArgoCD detects the difference between Git and the live cluster.

Expected:

```text
OutOfSync
```

If automated self-healing is enabled, ArgoCD may automatically restore the Git state.

---

## 3. Sync Again

```bash
argocd app sync day21-nginx
```

or:

```bash
argocd app sync guestbook
```

What this does:

This restores the cluster state from Git.

---

# Lab 13: Update Application Through Git

## 1. Change Deployment Manifest

In your Git repository, update replicas or image version.

Example:

```yaml
replicas: 3
```

or:

```yaml
image: nginx:1.26
```

---

## 2. Commit and Push

```bash
git add .
git commit -m "Update Day21 application"
git push
```

What this does:

This updates the desired application state in Git.

---

## 3. Refresh ArgoCD Application

```bash
argocd app refresh day21-nginx
```

---

## 4. Sync Application

```bash
argocd app sync day21-nginx
```

---

## 5. Verify Update

```bash
kubectl get deployment -n $APP_NAMESPACE
kubectl describe deployment day21-nginx -n $APP_NAMESPACE
```

What this does:

This confirms that the Git change was applied to the cluster.

---

# Lab 14: IAM Review for ArgoCD and EKS

ArgoCD itself manages Kubernetes resources inside the EKS cluster using Kubernetes RBAC.

## 1. Review ArgoCD Service Accounts

```bash
kubectl get serviceaccount -n $ARGOCD_NAMESPACE
```

What this does:

This lists Kubernetes service accounts used by ArgoCD components.

---

## 2. Review ArgoCD RBAC

```bash
kubectl get clusterrole | grep argocd
kubectl get clusterrolebinding | grep argocd
```

What this does:

This shows Kubernetes RBAC permissions assigned to ArgoCD.

---

## 3. Verify AWS Caller Identity

```bash
aws sts get-caller-identity \
  --profile $AWS_PROFILE
```

What this does:

This validates the IAM identity used for EKS cluster access.

---

## 4. Review EKS Cluster IAM Role

```bash
aws eks describe-cluster \
  --name $CLUSTER_NAME \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query 'cluster.roleArn' \
  --output text
```

What this does:

This shows the IAM role used by the EKS control plane.

---

# Lab 15: ArgoCD Application Health and Sync Policies

## 1. View Application Details

```bash
argocd app get day21-nginx
```

Review:

- Sync Status
- Health Status
- Repository
- Target Revision
- Destination Namespace
- Kubernetes Resources

---

## 2. View Application Events

```bash
kubectl describe application day21-nginx -n $ARGOCD_NAMESPACE
```

What this does:

This displays ArgoCD Application events and sync history.

---

## 3. Enable Auto-Sync if Not Enabled

```bash
argocd app set day21-nginx \
  --sync-policy automated \
  --auto-prune \
  --self-heal
```

What this does:

This enables automatic synchronization, automatic pruning, and self-healing.

---

# Lab 16: Troubleshooting

## ArgoCD Pods Not Running

```bash
kubectl get pods -n $ARGOCD_NAMESPACE
kubectl describe pod <pod-name> -n $ARGOCD_NAMESPACE
kubectl logs <pod-name> -n $ARGOCD_NAMESPACE
```

Common causes:

- Image pull issue
- Insufficient node capacity
- Kubernetes API issue

---

## Cannot Access ArgoCD UI

Check service:

```bash
kubectl get svc -n $ARGOCD_NAMESPACE
```

If using port-forward:

```bash
kubectl port-forward svc/argocd-server \
  -n $ARGOCD_NAMESPACE \
  8080:443
```

If using LoadBalancer:

```bash
kubectl describe svc argocd-server -n $ARGOCD_NAMESPACE
```

---

## ArgoCD CLI Login Failed

Verify password:

```bash
kubectl -n $ARGOCD_NAMESPACE get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
echo
```

Login with insecure flag for lab:

```bash
argocd login localhost:8080 \
  --username admin \
  --password <ARGOCD_ADMIN_PASSWORD> \
  --insecure
```

---

## Application OutOfSync

```bash
argocd app get day21-nginx
argocd app diff day21-nginx
argocd app sync day21-nginx
```

Common causes:

- Manual changes in cluster
- Git repository changed
- Failed sync
- Missing namespace
- Invalid Kubernetes manifest

---

## Application Not Healthy

```bash
kubectl get all -n $APP_NAMESPACE
kubectl describe pod <pod-name> -n $APP_NAMESPACE
kubectl logs <pod-name> -n $APP_NAMESPACE
```

Common causes:

- Container crash
- Wrong image
- Service selector mismatch
- Resource limits too low

---

# Lab 17: Cleanup

## 1. Delete ArgoCD Application

If using CLI-created app:

```bash
argocd app delete guestbook
```

Confirm deletion when prompted.

If using YAML-created app:

```bash
kubectl delete application day21-nginx -n $ARGOCD_NAMESPACE
```

---

## 2. Delete Application Namespace

```bash
kubectl delete namespace $APP_NAMESPACE
```

---

## 3. Delete ArgoCD Namespace

```bash
kubectl delete namespace $ARGOCD_NAMESPACE
```

What this does:

This removes ArgoCD and its components from the cluster.

---

## 4. Verify Cleanup

```bash
kubectl get namespaces
kubectl get pods -A | grep argocd
kubectl get pods -A | grep day21
```

Expected:

No ArgoCD or Day21 application pods should remain.

---

# Challenge Exercise

Implement a complete GitOps workflow:

1. Create your own Git repository.
2. Add Kubernetes manifests for an nginx application.
3. Install ArgoCD on EKS.
4. Create an ArgoCD Application pointing to your Git repository.
5. Enable automatic sync.
6. Update the app through Git.
7. Verify ArgoCD syncs the change.
8. Manually change the live deployment.
9. Verify ArgoCD detects and corrects drift.
10. Document the full workflow.

---

# Lab Deliverables

Submit:

- Git repository screenshot
- ArgoCD pods screenshot
- ArgoCD UI screenshot
- ArgoCD Application screenshot
- `argocd app list` output
- `kubectl get pods -n day21` output
- Drift detection screenshot
- Git commit history screenshot
- Application access screenshot
- Cleanup verification screenshot

---

# Expected Learning Outcomes

After completing this lab, participants should be able to:

- Explain GitOps principles.
- Explain ArgoCD architecture.
- Install ArgoCD on Amazon EKS.
- Access ArgoCD UI and CLI.
- Deploy Kubernetes applications from Git.
- Sync applications using ArgoCD.
- Detect and correct configuration drift.
- Understand automated sync, prune, and self-heal.
- Review IAM and Kubernetes RBAC used in the lab.
- Clean up ArgoCD and application resources safely.
