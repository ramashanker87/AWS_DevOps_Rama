# Day 22 Lab - Flux CD, Progressive Delivery, and DevSecOps Basics on Amazon EKS

## Module: Kubernetes with Amazon EKS

## Topics Covered

- Flux CD
- GitOps with Flux
- Progressive Delivery
- DevSecOps basics
- Secure Kubernetes deployment
- AWS KMS
- AWS Secrets Manager
- Amazon EKS

## Lab

- Flux CD configuration
- Secure deployment
- Secret management with AWS Secrets Manager
- Secret encryption with AWS KMS
- Progressive delivery concepts using Flux and Flagger

## AWS Resources Used

- Amazon EKS
- Flux CD
- AWS KMS
- AWS Secrets Manager
- IAM
- Kubernetes Secrets
- Kubernetes Deployments
- Kubernetes Services
- AWS CLI
- kubectl

---

# Lab Objectives

By the end of this lab, participants will be able to:

- Understand Flux CD and GitOps workflow.
- Install Flux CLI.
- Bootstrap Flux CD on Amazon EKS.
- Connect Flux CD with a Git repository.
- Deploy applications using Flux.
- Understand progressive delivery concepts.
- Configure a secure deployment workflow.
- Store application secrets in AWS Secrets Manager.
- Use AWS KMS for encryption.
- Review DevSecOps basics for Kubernetes workloads.
- Validate secure application deployment.
- Clean up Flux and AWS resources.

---

# Flux CD Overview

Flux CD is a GitOps continuous delivery tool for Kubernetes.

It continuously watches Git repositories and applies the desired state to Kubernetes clusters.

## Flux CD Core Components

- **source-controller**

  Watches Git, Helm, and OCI sources.

- **kustomize-controller**

  Applies Kubernetes manifests using Kustomize.

- **helm-controller**

  Manages Helm releases.

- **notification-controller**

  Sends alerts and events.

- **image-reflector-controller**

  Scans image repositories for new versions.

- **image-automation-controller**

  Updates Git manifests when new images are available.

## Flux CD Workflow

1. Kubernetes manifests are stored in Git.
2. Flux watches the Git repository.
3. Flux pulls changes from Git.
4. Flux applies manifests to the EKS cluster.
5. Flux reconciles cluster state with Git state.
6. If cluster drift occurs, Flux restores the desired state.

---

# Progressive Delivery Overview

Progressive delivery is a deployment strategy where changes are released gradually.

Common strategies:

- Blue/green deployment
- Canary deployment
- A/B testing
- Traffic shifting
- Automated rollback
- Metric-based promotion

Flux can work with tools like **Flagger** to implement progressive delivery.

Flagger can automate canary deployments by using metrics and service mesh or ingress controllers.

---

# DevSecOps Basics

DevSecOps means integrating security into the development and deployment lifecycle.

Important DevSecOps practices for EKS:

- Store secrets securely.
- Encrypt sensitive data.
- Use least-privilege IAM permissions.
- Scan container images.
- Avoid hardcoded credentials.
- Use Kubernetes RBAC.
- Use network policies where required.
- Use Git pull requests for infrastructure and application changes.
- Track all changes through Git history.
- Use KMS encryption for secrets.
- Use Secrets Manager for application secrets.

---

# Lab Environment

Use the existing EKS cluster from previous labs.

```bash
export AWS_REGION=us-east-1
export AWS_PROFILE=devops
export CLUSTER_NAME=rama-eks-cluster
export NAMESPACE=day22
export FLUX_NAMESPACE=flux-system
export GITHUB_USER=<your-github-username>
export GITHUB_REPO=day22-flux-gitops
export GITHUB_BRANCH=main
```

Explanation:

- `AWS_REGION` is the AWS region where EKS is running.
- `AWS_PROFILE` is the AWS CLI profile used for AWS access.
- `CLUSTER_NAME` is the EKS cluster name.
- `NAMESPACE` is the namespace for the sample app.
- `FLUX_NAMESPACE` is where Flux components are installed.
- `GITHUB_USER` is your GitHub username or organization.
- `GITHUB_REPO` is the Git repository used by Flux.
- `GITHUB_BRANCH` is the Git branch Flux will track.

---

# Prerequisites

## 1. Verify AWS CLI Identity

```bash
aws sts get-caller-identity \
  --profile $AWS_PROFILE
```

What this does:

This verifies that your terminal is authenticated to AWS.

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

This confirms that the EKS cluster exists and is active.

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

This updates the local kubeconfig so `kubectl` can connect to the EKS cluster.

---

## 4. Verify Worker Nodes

```bash
kubectl get nodes
```

What this does:

This checks that EKS worker nodes are registered and ready.

Expected:

```text
STATUS: Ready
```

---

## 5. Verify Git and GitHub Access

```bash
git --version
```

What this does:

This confirms Git is installed.

Flux bootstrap requires access to a Git provider such as GitHub.

Create a GitHub personal access token with repository permissions.

Set token:

```bash
export GITHUB_TOKEN=<your-github-token>
```

Important:

Do not commit this token into Git.

---

# Lab 1: Install Flux CLI

## 1. Install Flux CLI

For Linux or AWS CloudShell:

```bash
curl -s https://fluxcd.io/install.sh | sudo bash
```

What this does:

This downloads and installs the Flux CLI.

---

## 2. Verify Flux CLI

```bash
flux --version
```

What this does:

This confirms the Flux CLI is installed.

---

## 3. Run Flux Precheck

```bash
flux check --pre
```

What this does:

This checks whether the Kubernetes cluster meets Flux installation requirements.

Expected:

```text
checks passed
```

---

# Lab 2: Create Namespace for Application

```bash
kubectl create namespace $NAMESPACE
```

What this does:

This creates the namespace where the secure sample application will run.

Verify:

```bash
kubectl get namespace $NAMESPACE
```

---

# Lab 3: Bootstrap Flux CD on EKS

## 1. Bootstrap Flux with GitHub

```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=$GITHUB_REPO \
  --branch=$GITHUB_BRANCH \
  --path=clusters/$CLUSTER_NAME \
  --personal
```

What this does:

This installs Flux components into the EKS cluster and creates the GitOps repository structure.

Flux creates:

- `flux-system` namespace
- Flux controllers
- Git repository configuration
- Kustomization configuration
- Deploy key or token configuration
- GitOps manifests in the repository

---

## 2. Verify Flux Namespace

```bash
kubectl get namespace $FLUX_NAMESPACE
```

---

## 3. Verify Flux Pods

```bash
kubectl get pods -n $FLUX_NAMESPACE
```

Expected pods:

```text
source-controller
kustomize-controller
helm-controller
notification-controller
```

Expected status:

```text
Running
```

---

## 4. Verify Flux System

```bash
flux check
```

What this does:

This confirms that Flux is installed and all controllers are healthy.

---

# Lab 4: Inspect Flux GitOps Resources

## 1. View GitRepository Resource

```bash
kubectl get gitrepositories -n $FLUX_NAMESPACE
```

What this does:

This shows the Git repository source that Flux monitors.

---

## 2. Describe GitRepository

```bash
kubectl describe gitrepository flux-system -n $FLUX_NAMESPACE
```

What this does:

This displays the repository URL, branch, sync interval, and latest revision.

---

## 3. View Kustomization Resource

```bash
kubectl get kustomizations -n $FLUX_NAMESPACE
```

What this does:

This shows the Kustomization that applies manifests from Git.

---

## 4. Describe Kustomization

```bash
kubectl describe kustomization flux-system -n $FLUX_NAMESPACE
```

What this does:

This displays reconciliation status and errors if Flux cannot apply manifests.

---

# Lab 5: Create Sample Application Manifests

Clone the GitOps repository created by Flux:

```bash
git clone https://github.com/$GITHUB_USER/$GITHUB_REPO.git
cd $GITHUB_REPO
```

What this does:

This downloads the GitOps repository to your local environment.

---

## 1. Create Application Folder

```bash
mkdir -p apps/day22-secure-app
```

---

## 2. Create Deployment Manifest

```bash
cat > apps/day22-secure-app/deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-nginx
  namespace: day22
  labels:
    app: secure-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: secure-nginx
  template:
    metadata:
      labels:
        app: secure-nginx
    spec:
      securityContext:
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 8080
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: false
          capabilities:
            drop:
            - ALL
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

This creates a secure baseline Kubernetes deployment.

Security settings:

- `runAsNonRoot: true` avoids running the pod as root.
- `seccompProfile: RuntimeDefault` uses the default Linux syscall security profile.
- `allowPrivilegeEscalation: false` prevents privilege escalation.
- `capabilities.drop: ALL` removes Linux capabilities from the container.
- Resource requests and limits prevent uncontrolled resource usage.

Note:

The default nginx image may require additional configuration to run perfectly as non-root on port 8080. In production, use a non-root nginx image or custom Dockerfile.

---

## 3. Create Service Manifest

```bash
cat > apps/day22-secure-app/service.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: secure-nginx
  namespace: day22
spec:
  type: ClusterIP
  selector:
    app: secure-nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
EOF
```

What this does:

This creates a ClusterIP service for internal access to the application.

---

## 4. Create Kustomization File

```bash
cat > apps/day22-secure-app/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
EOF
```

What this does:

This tells Flux/Kustomize which manifests belong to the application.

---

# Lab 6: Add Application to Flux Sync Path

Create an application Kustomization resource under the cluster path.

```bash
mkdir -p clusters/$CLUSTER_NAME/apps
```

Create Flux Kustomization:

```bash
cat > clusters/$CLUSTER_NAME/apps/day22-secure-app.yaml <<EOF
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: day22-secure-app
  namespace: flux-system
spec:
  interval: 1m
  path: ./apps/day22-secure-app
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
  targetNamespace: day22
EOF
```

What this does:

This tells Flux to continuously reconcile the manifests from `apps/day22-secure-app`.

Explanation:

- `interval: 1m` means Flux checks every minute.
- `path` points to the application manifests in Git.
- `prune: true` removes cluster resources deleted from Git.
- `sourceRef` points to the Git repository Flux already tracks.
- `targetNamespace` sets the deployment namespace.

---

# Lab 7: Commit and Push GitOps Changes

```bash
git add .
git commit -m "Add Day22 secure app with Flux"
git push
```

What this does:

This pushes the desired application state into Git.

Flux will detect this commit and apply the application to EKS.

---

# Lab 8: Force Flux Reconciliation

```bash
flux reconcile source git flux-system \
  -n $FLUX_NAMESPACE
```

What this does:

This tells Flux to immediately pull the latest Git changes.

Then reconcile the application:

```bash
flux reconcile kustomization day22-secure-app \
  -n $FLUX_NAMESPACE
```

What this does:

This tells Flux to immediately apply the application manifests.

---

# Lab 9: Verify Application Deployment

## 1. Check Flux Kustomizations

```bash
flux get kustomizations
```

Expected:

```text
day22-secure-app   True
```

---

## 2. Check Pods

```bash
kubectl get pods -n $NAMESPACE
```

Expected:

```text
secure-nginx pods Running
```

---

## 3. Check Deployment

```bash
kubectl get deployment secure-nginx -n $NAMESPACE
```

---

## 4. Check Service

```bash
kubectl get svc secure-nginx -n $NAMESPACE
```

---

## 5. Describe Pod Security

```bash
kubectl describe pod <pod-name> -n $NAMESPACE
```

What this does:

This shows pod configuration, security context, events, and container details.

---

# Lab 10: Test Application Access

Use port-forwarding:

```bash
kubectl port-forward svc/secure-nginx \
  -n $NAMESPACE \
  8080:80
```

Open another terminal:

```bash
curl http://localhost:8080
```

What this does:

This forwards local traffic to the Kubernetes service for testing.

---

# Lab 11: Create AWS KMS Key for Secret Encryption

## 1. Create KMS Key

```bash
aws kms create-key \
  --description "Day22 EKS DevSecOps secrets encryption key" \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This creates a KMS key that can be used to encrypt sensitive values.

Record the Key ID.

```bash
export KMS_KEY_ID=<kms-key-id>
```

---

## 2. Create KMS Alias

```bash
aws kms create-alias \
  --alias-name alias/day22-devsecops \
  --target-key-id $KMS_KEY_ID \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This creates a friendly alias for the KMS key.

---

## 3. Verify KMS Key

```bash
aws kms describe-key \
  --key-id alias/day22-devsecops \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

---

# Lab 12: Create Secret in AWS Secrets Manager

## 1. Create Secret

```bash
aws secretsmanager create-secret \
  --name day22/app/config \
  --description "Day22 sample application secret" \
  --kms-key-id alias/day22-devsecops \
  --secret-string '{"APP_USER":"admin","APP_PASSWORD":"ChangeMe123!"}' \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This stores sensitive application configuration in AWS Secrets Manager encrypted with the KMS key.

Important:

Use strong passwords in real environments. Do not commit secrets to Git.

---

## 2. Verify Secret Metadata

```bash
aws secretsmanager describe-secret \
  --secret-id day22/app/config \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This shows metadata about the secret without exposing the secret value.

---

## 3. Retrieve Secret for Testing

```bash
aws secretsmanager get-secret-value \
  --secret-id day22/app/config \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query SecretString \
  --output text
```

What this does:

This retrieves the secret value.

Security note:

Avoid printing secrets in production terminals or logs.

---

# Lab 13: Create Kubernetes Secret from Secrets Manager Value

For this beginner lab, create a Kubernetes secret manually using the AWS Secrets Manager value.

```bash
export APP_USER=$(aws secretsmanager get-secret-value \
  --secret-id day22/app/config \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query SecretString \
  --output text | jq -r .APP_USER)

export APP_PASSWORD=$(aws secretsmanager get-secret-value \
  --secret-id day22/app/config \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query SecretString \
  --output text | jq -r .APP_PASSWORD)
```

What this does:

This reads the secret values from AWS Secrets Manager into environment variables.

Create Kubernetes Secret:

```bash
kubectl create secret generic app-secret \
  --from-literal=APP_USER=$APP_USER \
  --from-literal=APP_PASSWORD=$APP_PASSWORD \
  -n $NAMESPACE
```

What this does:

This creates a Kubernetes secret for the application.

Important DevSecOps note:

For production, use External Secrets Operator or AWS Secrets and Configuration Provider with the Secrets Store CSI Driver instead of manually copying secrets.

---

# Lab 14: Update Deployment to Use Kubernetes Secret

Edit the deployment in Git:

```bash
cat > apps/day22-secure-app/deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-nginx
  namespace: day22
  labels:
    app: secure-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: secure-nginx
  template:
    metadata:
      labels:
        app: secure-nginx
    spec:
      securityContext:
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        env:
        - name: APP_USER
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: APP_USER
        - name: APP_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: APP_PASSWORD
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
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

This updates the application so it consumes secrets from Kubernetes environment variables.

Commit and push:

```bash
git add .
git commit -m "Use Kubernetes secret in Day22 app"
git push
```

Reconcile:

```bash
flux reconcile source git flux-system -n $FLUX_NAMESPACE
flux reconcile kustomization day22-secure-app -n $FLUX_NAMESPACE
```

Verify rollout:

```bash
kubectl rollout status deployment/secure-nginx -n $NAMESPACE
```

---

# Lab 15: Verify Secret Injection

Get pod name:

```bash
export POD_NAME=$(kubectl get pods -n $NAMESPACE \
  -l app=secure-nginx \
  -o jsonpath='{.items[0].metadata.name}')
```

Verify environment variable names:

```bash
kubectl exec -n $NAMESPACE $POD_NAME -- printenv | grep APP_
```

What this does:

This confirms that the pod has received secret-backed environment variables.

Security note:

Avoid printing actual secret values during real audits or production operations.

---

# Lab 16: Progressive Delivery with Flagger Overview

Flux works well with Flagger for progressive delivery.

Flagger can automate:

- Canary deployments
- Blue/green deployments
- Traffic shifting
- Metric analysis
- Automatic rollback

Common Flagger integrations:

- NGINX Ingress
- AWS App Mesh
- Istio
- Linkerd
- Gateway API

This lab introduces the concept. Full production canary requires ingress, service mesh, or traffic routing configuration.

---

# Lab 17: Install Flagger for Progressive Delivery Demo

Add Helm repository:

```bash
helm repo add flagger https://flagger.app
helm repo update
```

Install Flagger with Kubernetes provider:

```bash
helm upgrade -i flagger flagger/flagger \
  --namespace flagger-system \
  --create-namespace \
  --set meshProvider=kubernetes
```

What this does:

This installs Flagger in basic Kubernetes mode.

Verify:

```bash
kubectl get pods -n flagger-system
```

---

# Lab 18: Progressive Delivery Strategy Example

Create a canary manifest example in Git:

```bash
cat > apps/day22-secure-app/canary.yaml <<'EOF'
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: secure-nginx
  namespace: day22
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: secure-nginx
  progressDeadlineSeconds: 60
  service:
    port: 80
    targetPort: 80
  analysis:
    interval: 1m
    threshold: 3
    iterations: 3
EOF
```

Update kustomization:

```bash
cat > apps/day22-secure-app/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
  - canary.yaml
EOF
```

Commit and push:

```bash
git add .
git commit -m "Add Flagger canary configuration"
git push
```

Reconcile:

```bash
flux reconcile source git flux-system -n $FLUX_NAMESPACE
flux reconcile kustomization day22-secure-app -n $FLUX_NAMESPACE
```

Verify canary:

```bash
kubectl get canary -n $NAMESPACE
kubectl describe canary secure-nginx -n $NAMESPACE
```

What this does:

This introduces a progressive delivery custom resource managed by Flagger.

Note:

For full traffic shifting, configure an ingress controller or service mesh integration.

---

# Lab 19: DevSecOps Validation Checklist

## 1. Verify No Secrets in Git

```bash
git grep -i "password"
git grep -i "secret"
git grep -i "token"
```

What this does:

This checks if secrets were accidentally committed to Git.

---

## 2. Verify Kubernetes Secret Exists

```bash
kubectl get secret app-secret -n $NAMESPACE
```

---

## 3. Verify KMS Key

```bash
aws kms describe-key \
  --key-id alias/day22-devsecops \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

---

## 4. Verify Secrets Manager Secret

```bash
aws secretsmanager describe-secret \
  --secret-id day22/app/config \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

---

## 5. Verify Pod Security Context

```bash
kubectl get deployment secure-nginx \
  -n $NAMESPACE \
  -o yaml | grep -A20 securityContext
```

---

## 6. Verify Flux Reconciliation

```bash
flux get sources git
flux get kustomizations
```

---

# Lab 20: Troubleshooting

## Flux Pods Not Running

```bash
kubectl get pods -n $FLUX_NAMESPACE
kubectl describe pod <pod-name> -n $FLUX_NAMESPACE
kubectl logs <pod-name> -n $FLUX_NAMESPACE
```

Common causes:

- Git authentication issue
- Network issue
- Kubernetes scheduling issue

---

## Flux Cannot Pull Git Repository

```bash
flux get sources git
kubectl describe gitrepository flux-system -n $FLUX_NAMESPACE
```

Common causes:

- Wrong GitHub token
- Repository does not exist
- Missing repository permission
- Wrong branch name

---

## Kustomization Fails

```bash
flux get kustomizations
kubectl describe kustomization day22-secure-app -n $FLUX_NAMESPACE
```

Common causes:

- Invalid YAML
- Wrong path
- Namespace missing
- Resource conflict

---

## Pod Fails to Start

```bash
kubectl get pods -n $NAMESPACE
kubectl describe pod <pod-name> -n $NAMESPACE
kubectl logs <pod-name> -n $NAMESPACE
```

Common causes:

- Container port mismatch
- Security context incompatible with image
- Missing Kubernetes secret
- Resource constraints

---

## Secret Not Found

```bash
kubectl get secret app-secret -n $NAMESPACE
```

If missing, recreate from Secrets Manager.

---

# Lab 21: Cleanup

## 1. Delete Flagger

```bash
helm uninstall flagger -n flagger-system
kubectl delete namespace flagger-system
```

---

## 2. Delete Kubernetes Secret

```bash
kubectl delete secret app-secret -n $NAMESPACE --ignore-not-found=true
```

---

## 3. Delete Application Namespace

```bash
kubectl delete namespace $NAMESPACE
```

---

## 4. Uninstall Flux

```bash
flux uninstall --namespace=$FLUX_NAMESPACE
```

Confirm when prompted.

---

## 5. Delete Secrets Manager Secret

```bash
aws secretsmanager delete-secret \
  --secret-id day22/app/config \
  --force-delete-without-recovery \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This permanently deletes the secret without recovery.

For production, avoid force deletion unless required.

---

## 6. Delete KMS Alias

```bash
aws kms delete-alias \
  --alias-name alias/day22-devsecops \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

---

## 7. Schedule KMS Key Deletion

```bash
aws kms schedule-key-deletion \
  --key-id $KMS_KEY_ID \
  --pending-window-in-days 7 \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This schedules the KMS key for deletion after 7 days.

KMS keys cannot be deleted immediately.

---

# Challenge Exercise

Implement a secure GitOps workflow:

1. Bootstrap Flux CD on EKS.
2. Create a GitOps repository structure.
3. Deploy a secure application through Flux.
4. Store application secrets in AWS Secrets Manager.
5. Encrypt the secret with AWS KMS.
6. Inject secret values into Kubernetes securely.
7. Add progressive delivery configuration with Flagger.
8. Validate Flux reconciliation and drift correction.
9. Confirm no secrets are stored in Git.
10. Document all commands and screenshots.

---

# Lab Deliverables

Submit:

- Flux CLI version screenshot
- Flux bootstrap output
- Flux pods screenshot
- Git repository structure screenshot
- Flux Kustomization screenshot
- Application pods screenshot
- KMS key screenshot
- Secrets Manager screenshot
- Kubernetes secret screenshot
- Secure deployment YAML screenshot
- Progressive delivery canary screenshot
- Cleanup verification screenshot

---

# Expected Learning Outcomes

After completing this lab, participants should be able to:

- Explain Flux CD and GitOps workflow.
- Bootstrap Flux CD on Amazon EKS.
- Deploy Kubernetes applications from Git using Flux.
- Understand progressive delivery principles.
- Describe how Flagger supports canary deployments.
- Use AWS KMS for secret encryption.
- Store secrets in AWS Secrets Manager.
- Understand secure deployment practices.
- Apply basic DevSecOps controls to Kubernetes workloads.
- Troubleshoot Flux reconciliation issues.
- Clean up Flux, Kubernetes, KMS, and Secrets Manager resources safely.
