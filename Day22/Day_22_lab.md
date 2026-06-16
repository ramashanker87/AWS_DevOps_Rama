# Day 22 Lab - Flux CD, Progressive Delivery, DevSecOps, AWS CodeCommit, and Amazon ECR on Amazon EKS

## Module: Kubernetes with Amazon EKS

## Topics Covered

- Flux CD
- GitOps with AWS CodeCommit
- Progressive delivery concepts
- DevSecOps basics
- Secure Kubernetes deployment
- Amazon ECR image repository
- AWS KMS
- AWS Secrets Manager
- Amazon EKS
- IAM, AWS CLI, kubectl, Docker, Helm, and Flux CLI

---

# Lab Objectives

By the end of this lab, participants will be able to:

- Understand Flux CD and GitOps workflow.
- Install and validate Flux CLI.
- Create an AWS CodeCommit repository for GitOps manifests.
- Create an Amazon ECR repository for container images.
- Build, tag, and push an application image to ECR.
- Install Flux CD on Amazon EKS using AWS-native Git access.
- Deploy Kubernetes workloads from AWS CodeCommit using Flux.
- Store application secrets in AWS Secrets Manager.
- Encrypt secrets using AWS KMS.
- Create a Kubernetes Secret from AWS Secrets Manager for lab usage.
- Validate secure Kubernetes deployment settings.
- Understand progressive delivery concepts with Flagger.
- Clean up Flux, Kubernetes, ECR, KMS, Secrets Manager, and CodeCommit resources.

---

# AWS Resources Used

- Amazon EKS
- AWS CodeCommit
- Amazon ECR
- AWS KMS
- AWS Secrets Manager
- IAM
- Flux CD
- Flagger
- Kubernetes Namespaces
- Kubernetes Deployments
- Kubernetes Services
- Kubernetes Secrets
- AWS CLI
- kubectl
- Docker
- Helm

---

# Flux CD Overview

Flux CD is a GitOps continuous delivery tool for Kubernetes. Flux continuously watches a Git repository and reconciles the desired state stored in Git with the live state in the Kubernetes cluster.

## Flux CD Core Components

- **source-controller**: watches Git, Helm, and OCI sources.
- **kustomize-controller**: applies Kubernetes manifests using Kustomize.
- **helm-controller**: manages Helm releases.
- **notification-controller**: sends alerts and events.
- **image-reflector-controller**: scans image repositories for new versions.
- **image-automation-controller**: updates Git manifests when new images are available.

## Flux CD Workflow

1. Kubernetes manifests are stored in AWS CodeCommit.
2. Flux watches the CodeCommit Git repository.
3. Flux pulls changes from CodeCommit.
4. Flux applies manifests to the EKS cluster.
5. Flux reconciles cluster state with Git state.
6. If drift occurs, Flux restores the desired state from Git.

---

# Progressive Delivery Overview

Progressive delivery is a deployment strategy where application changes are released gradually.

Common strategies:

- Blue/green deployment
- Canary deployment
- A/B testing
- Traffic shifting
- Automated rollback
- Metric-based promotion

Flux can work with tools like **Flagger** to implement progressive delivery.

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
- Track all changes through Git history.
- Use KMS encryption for secrets.
- Use Secrets Manager for application secrets.
- Use ECR image scanning for container image visibility.

---

# Lab Environment

Use the existing EKS cluster from previous labs.

```bash
export AWS_REGION=us-east-1
export AWS_PROFILE=devops
export CLUSTER_NAME=day-22-rama-eks-cluster
export NAMESPACE=day22
export FLUX_NAMESPACE=flux-system
export CODECOMMIT_REPO=day22-flux-gitops
export CODECOMMIT_BRANCH=main
export ECR_REPO=day22-secure-nginx
export IMAGE_TAG=v1
export NODEGROUP_NAME=day22-managed-ng
export KMS_ALIAS=alias/day22-devsecops
export SECRET_NAME=day22/app/config
```

Get your AWS account ID:

```bash
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity \
  --profile $AWS_PROFILE \
  --query Account \
  --output text)
```

Create reusable repository URLs:

```bash
export CODECOMMIT_REPO_URL=https://git-codecommit.${AWS_REGION}.amazonaws.com/v1/repos/${CODECOMMIT_REPO}
export ECR_REPO_URI=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}
```

Explanation:

- `AWS_REGION` is the AWS region where EKS, CodeCommit, ECR, KMS, and Secrets Manager are used.
- `AWS_PROFILE` is the AWS CLI profile used for authentication.
- `CLUSTER_NAME` is the existing EKS cluster name.
- `NAMESPACE` is the namespace where the sample application runs.
- `FLUX_NAMESPACE` is where Flux components are installed.
- `CODECOMMIT_REPO` is the GitOps repository name in AWS CodeCommit.
- `CODECOMMIT_BRANCH` is the Git branch Flux tracks.
- `ECR_REPO` is the Amazon ECR repository for the sample image.
- `IMAGE_TAG` is the container image version.
- `KMS_ALIAS` is the friendly alias for the KMS key.
- `SECRET_NAME` is the AWS Secrets Manager secret name.

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
export VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query 'Vpcs[0].VpcId' \
  --output text)
```
```bash
echo $VPC_ID
```

```bash
export SUBNET_IDS=$(aws ec2 describe-subnets \
  --filters Name=vpc-id,Values=$VPC_ID \
            Name=availability-zone,Values=us-east-1a,us-east-1b \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query 'join(`,`, Subnets[*].SubnetId)' \
  --output text)
```
```bash
echo $SUBNET_IDS
```
```bash
eksctl create cluster \
  --name $CLUSTER_NAME \
  --region $AWS_REGION \
  --vpc-public-subnets $SUBNET_IDS \
  --nodes 1 \
  --nodes-min 1 \
  --nodes-max 4 \
  --nodegroup-name $NODEGROUP_NAME \
  --managed \
  --profile $AWS_PROFILE
```




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

## 5. Verify Required Local Tools

```bash
aws --version
git --version
kubectl version --client
docker --version
helm version
jq --version
```

What this does:

This confirms that the required CLI tools are available.

---

## 6. Configure Git to Use AWS CodeCommit Credential Helper

```bash
git config --global credential.helper '!aws codecommit credential-helper $@'
git config --global credential.UseHttpPath true
export AWS_PROFILE=devops
```

What this does:

This allows Git to authenticate to AWS CodeCommit using your AWS CLI profile or environment credentials.

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

This confirms that the Flux CLI is installed.

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

# Lab 2: Create AWS CodeCommit GitOps Repository

## 1. Create CodeCommit Repository

```bash
aws codecommit create-repository \
  --repository-name $CODECOMMIT_REPO \
  --repository-description "Day 22 Flux GitOps repository for EKS" \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This creates a private AWS CodeCommit repository that stores Kubernetes manifests and Flux configuration.

---

## 2. Verify CodeCommit Repository

```bash
aws codecommit get-repository \
  --repository-name $CODECOMMIT_REPO \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query 'repositoryMetadata.{Name:repositoryName,CloneUrlHttp:cloneUrlHttp,Arn:Arn}' \
  --output table
```

What this does:

This confirms that the repository exists and shows the HTTPS clone URL.

---

## 3. Clone the Empty Repository

```bash
git clone $CODECOMMIT_REPO_URL
cd $CODECOMMIT_REPO
```

What this does:

This downloads the empty CodeCommit repository to your local machine.

---

## 4. Create the Main Branch and Initial Commit

```bash
echo "# Day 22 Flux GitOps Repository" > README.md
git add README.md
git commit -m "Initial Day22 GitOps repository"
git branch -M $CODECOMMIT_BRANCH
git push -u origin $CODECOMMIT_BRANCH
```

What this does:

This creates the first commit and pushes the `main` branch to CodeCommit.

---

# Lab 3: Create Amazon ECR Repository

## 1. Create ECR Repository

```bash
aws ecr create-repository \
  --repository-name $ECR_REPO \
  --image-scanning-configuration scanOnPush=true \
  --encryption-configuration encryptionType=AES256 \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This creates an Amazon ECR repository for the sample application image and enables image scanning on push.

---

## 2. Verify ECR Repository

```bash
aws ecr describe-repositories \
  --repository-names $ECR_REPO \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query 'repositories[0].{RepositoryName:repositoryName,RepositoryUri:repositoryUri,ScanOnPush:imageScanningConfiguration.scanOnPush}' \
  --output table
```

What this does:

This confirms that the ECR repository exists.

---

## 3. Authenticate Docker to ECR

```bash
aws ecr get-login-password --region $AWS_REGION --profile $AWS_PROFILE | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
```

What this does:

This logs Docker in to Amazon ECR so you can push images.

---

# Lab 4: Build and Push Sample Application Image to ECR

## 1. Create a Local Application Folder

```bash
mkdir -p ../day22-secure-nginx-image
cd ../day22-secure-nginx-image
```

What this does:

This creates a separate folder for the container image source files.

---

## 2. Create a Simple Nginx Web Page

```bash
cat > index.html <<'APPHTML'
<!DOCTYPE html>
<html>
<head>
  <title>Day 22 Secure GitOps App</title>
</head>
<body>
  <h1>Day 22 Secure GitOps App running on Amazon EKS</h1>
  <p>Image is stored in Amazon ECR and deployed by Flux from AWS CodeCommit.</p>
</body>
</html>
APPHTML
```

What this does:

This creates a simple HTML page served by nginx.

---

## 3. Create Dockerfile

```bash
cat > Dockerfile <<'DOCKERFILE'
FROM nginxinc/nginx-unprivileged:1.25-alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 8080
DOCKERFILE
```

What this does:

This uses an unprivileged nginx image that is more suitable for non-root Kubernetes deployments.

---

## 4. Build Docker Image

```bash
docker build -t $ECR_REPO:$IMAGE_TAG .
```

What this does:

This builds the local Docker image.

---

## 5. Tag Image for ECR

```bash
docker tag $ECR_REPO:$IMAGE_TAG $ECR_REPO_URI:$IMAGE_TAG
```

What this does:

This tags the local image with the full ECR repository URI.

---

## 6. Push Image to ECR

```bash
docker push $ECR_REPO_URI:$IMAGE_TAG
```

What this does:

This uploads the container image to Amazon ECR.

---

## 7. Verify Image in ECR

```bash
aws ecr describe-images \
  --repository-name $ECR_REPO \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query 'imageDetails[*].{Tags:imageTags,PushedAt:imagePushedAt,ScanStatus:imageScanStatus.status}' \
  --output table
```

What this does:

This confirms that the image was pushed and shows image scan status.

---

# Lab 5: Create Kubernetes Namespace

```bash
kubectl create namespace $NAMESPACE --dry-run=client -o yaml | kubectl apply -f -
```

What this does:

This creates the namespace where the secure sample application will run.

Verify:

```bash
kubectl get namespace $NAMESPACE
```

---

# Lab 6: Install Flux CD on EKS for AWS CodeCommit

Flux has built-in bootstrap helpers for GitHub and GitLab. For AWS CodeCommit, install Flux on the cluster and manually create the GitRepository and Kustomization resources.

## 1. Install Flux Controllers

```bash
flux install \
  --namespace=$FLUX_NAMESPACE
```

What this does:

This installs Flux controllers into the EKS cluster.

Flux creates:

- `flux-system` namespace
- source-controller
- kustomize-controller
- helm-controller
- notification-controller

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

# Lab 7: Create GitOps Repository Structure in CodeCommit

Return to the CodeCommit repository folder:

```bash
cd ../$CODECOMMIT_REPO
```

## 1. Create Folder Structure

```bash
mkdir -p clusters/$CLUSTER_NAME/apps
mkdir -p apps/day22-secure-app
```

What this does:

This creates a standard GitOps folder structure:

- `clusters/$CLUSTER_NAME` stores cluster-specific Flux configuration.
- `apps/day22-secure-app` stores application Kubernetes manifests.

---

## 2. Create Secure Deployment Manifest Using ECR Image

```bash
cat > apps/day22-secure-app/deployment.yaml <<APPDEPLOY
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-nginx
  namespace: ${NAMESPACE}
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
        image: ${ECR_REPO_URI}:${IMAGE_TAG}
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
APPDEPLOY
```

What this does:

This creates a secure baseline Kubernetes deployment using the image from Amazon ECR.

Security settings:

- `runAsNonRoot: true` avoids running the pod as root.
- `seccompProfile: RuntimeDefault` uses the default Linux syscall security profile.
- `allowPrivilegeEscalation: false` prevents privilege escalation.
- `capabilities.drop: ALL` removes Linux capabilities from the container.
- Resource requests and limits prevent uncontrolled resource usage.

---

## 3. Create Service Manifest

```bash
cat > apps/day22-secure-app/service.yaml <<APPSERVICE
apiVersion: v1
kind: Service
metadata:
  name: secure-nginx
  namespace: ${NAMESPACE}
spec:
  type: ClusterIP
  selector:
    app: secure-nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
APPSERVICE
```

What this does:

This creates a ClusterIP service for internal access to the application.

---

## 4. Create Application Kustomization File

```bash
cat > apps/day22-secure-app/kustomization.yaml <<APPKUSTOMIZE
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
APPKUSTOMIZE
```

What this does:

This tells Kustomize which manifests belong to the application.

---

## 5. Create Flux Kustomization for the Application

```bash
cat > clusters/$CLUSTER_NAME/apps/day22-secure-app.yaml <<FLUXAPP
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: day22-secure-app
  namespace: ${FLUX_NAMESPACE}
spec:
  interval: 1m
  path: ./apps/day22-secure-app
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
  targetNamespace: ${NAMESPACE}
FLUXAPP
```

What this does:

This tells Flux to continuously reconcile the manifests from `apps/day22-secure-app`.

Explanation:

- `interval: 1m` means Flux checks every minute.
- `path` points to the application manifests in Git.
- `prune: true` removes cluster resources deleted from Git.
- `sourceRef` points to the Git repository Flux tracks.
- `targetNamespace` sets the deployment namespace.

---

## 6. Create Root Cluster Kustomization

```bash
cat > clusters/$CLUSTER_NAME/kustomization.yaml <<ROOTKUSTOMIZE
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - apps/day22-secure-app.yaml
ROOTKUSTOMIZE
```

What this does:

This makes the application Flux Kustomization part of the cluster GitOps path.

---

# Lab 8: Commit and Push GitOps Changes to CodeCommit

```bash
git add .
git commit -m "Add Day22 secure app with Flux and ECR"
git push origin $CODECOMMIT_BRANCH
```

What this does:

This pushes the desired application state to AWS CodeCommit.

---

# Lab 9: Connect Flux to AWS CodeCommit

This lab uses CodeCommit HTTPS Git authentication. The simplest training approach is to create Git credentials for an IAM user and store them in a Kubernetes secret for Flux.

> Production note: prefer short-lived or strongly controlled credentials, least-privilege IAM, and a defined rotation process.

## 1. Create an IAM User for Flux Git Read Access

```bash
aws iam create-user \
  --user-name flux-codecommit-day22 \
  --profile $AWS_PROFILE
```

What this does:

This creates a dedicated IAM user for Flux to read the CodeCommit repository.

---

## 2. Attach CodeCommit Read-Only Policy

```bash
aws iam attach-user-policy \
  --user-name flux-codecommit-day22 \
  --policy-arn arn:aws:iam::aws:policy/AWSCodeCommitReadOnly \
  --profile $AWS_PROFILE
```

What this does:

This grants the IAM user read-only access to CodeCommit.

---

## 3. Create CodeCommit HTTPS Git Credentials

```bash
aws iam create-service-specific-credential \
  --user-name flux-codecommit-day22 \
  --service-name codecommit.amazonaws.com \
  --profile $AWS_PROFILE \
  --output json > flux-codecommit-credentials.json
```

What this does:

This creates service-specific credentials that Flux can use to clone the CodeCommit repository over HTTPS.

Export the credentials:

```bash
export FLUX_GIT_USERNAME=$(jq -r '.ServiceSpecificCredential.ServiceUserName' flux-codecommit-credentials.json)
export FLUX_GIT_PASSWORD=$(jq -r '.ServiceSpecificCredential.ServicePassword' flux-codecommit-credentials.json)
```

Important:

Do not commit `flux-codecommit-credentials.json` to Git.

---

## 4. Create Kubernetes Secret for Flux Git Authentication

```bash
kubectl create secret generic flux-system \
  --namespace=$FLUX_NAMESPACE \
  --from-literal=username=$FLUX_GIT_USERNAME \
  --from-literal=password=$FLUX_GIT_PASSWORD \
  --dry-run=client -o yaml | kubectl apply -f -
```

What this does:

This creates the Kubernetes secret Flux uses to authenticate to AWS CodeCommit.

---

## 5. Create Flux GitRepository Resource

```bash
cat > flux-gitrepository.yaml <<FLUXGIT
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: flux-system
  namespace: ${FLUX_NAMESPACE}
spec:
  interval: 1m
  url: ${CODECOMMIT_REPO_URL}
  ref:
    branch: ${CODECOMMIT_BRANCH}
  secretRef:
    name: flux-system
FLUXGIT
```

Apply it:

```bash
kubectl apply -f flux-gitrepository.yaml
```

What this does:

This tells Flux which CodeCommit repository and branch to watch.

---

## 6. Create Flux Root Kustomization Resource

```bash
cat > flux-root-kustomization.yaml <<FLUXROOT
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: flux-system
  namespace: ${FLUX_NAMESPACE}
spec:
  interval: 1m
  path: ./clusters/${CLUSTER_NAME}
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
FLUXROOT
```

Apply it:

```bash
kubectl apply -f flux-root-kustomization.yaml
```

What this does:

This tells Flux to apply everything under the cluster path in CodeCommit.

---

# Lab 10: Force Flux Reconciliation

## 1. Reconcile Git Source

```bash
flux reconcile source git flux-system \
  -n $FLUX_NAMESPACE
```

What this does:

This tells Flux to immediately pull the latest Git changes from CodeCommit.

---

## 2. Reconcile Root Kustomization

```bash
flux reconcile kustomization flux-system \
  -n $FLUX_NAMESPACE
```

What this does:

This tells Flux to apply the root cluster GitOps path.

---

## 3. Reconcile Application Kustomization

```bash
flux reconcile kustomization day22-secure-app \
  -n $FLUX_NAMESPACE
```

What this does:

This tells Flux to apply the application manifests immediately.

---

# Lab 11: Verify Application Deployment

## 1. Check Flux Git Sources

```bash
flux get sources git
```

Expected:

```text
flux-system   True
```

---

## 2. Check Flux Kustomizations

```bash
flux get kustomizations
```

Expected:

```text
flux-system        True
day22-secure-app   True
```

---

## 3. Check Pods

```bash
kubectl get pods -n $NAMESPACE
```

Expected:

```text
secure-nginx pods Running
```

---

## 4. Check Deployment

```bash
kubectl get deployment secure-nginx -n $NAMESPACE
```

---

## 5. Check Service

```bash
kubectl get svc secure-nginx -n $NAMESPACE
```

---

## 6. Describe Pod Security

```bash
export POD_NAME=$(kubectl get pods -n $NAMESPACE \
  -l app=secure-nginx \
  -o jsonpath='{.items[0].metadata.name}')

kubectl describe pod $POD_NAME -n $NAMESPACE
```

What this does:

This shows pod configuration, security context, events, and container details.

---

# Lab 12: Test Application Access

## 1. Port Forward the Service

```bash
kubectl port-forward svc/secure-nginx \
  -n $NAMESPACE \
  8080:80
```

What this does:

This forwards local port `8080` to the Kubernetes service.

---

## 2. Test with curl from Another Terminal

```bash
curl http://localhost:8080
```

Expected:

```text
Day 22 Secure GitOps App running on Amazon EKS
```

---

# Lab 13: Create AWS KMS Key for Secret Encryption

## 1. Create KMS Key

```bash
aws kms create-key \
  --description "Day22 EKS DevSecOps secrets encryption key" \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query 'KeyMetadata.KeyId' \
  --output text
```

What this does:

This creates a KMS key that can be used to encrypt sensitive values.

Export the key ID:

```bash
export KMS_KEY_ID=<kms-key-id-from-output>
```

---

## 2. Create KMS Alias

```bash
aws kms create-alias \
  --alias-name $KMS_ALIAS \
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
  --key-id $KMS_ALIAS \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query 'KeyMetadata.{KeyId:KeyId,Arn:Arn,Enabled:Enabled,KeyState:KeyState}' \
  --output table
```

What this does:

This confirms that the KMS key is enabled.

---

# Lab 14: Create Secret in AWS Secrets Manager

## 1. Create Secret

```bash
aws secretsmanager create-secret \
  --name $SECRET_NAME \
  --description "Day22 sample application secret" \
  --kms-key-id $KMS_ALIAS \
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
  --secret-id $SECRET_NAME \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query '{Name:Name,ARN:ARN,KmsKeyId:KmsKeyId,CreatedDate:CreatedDate}' \
  --output table
```

What this does:

This shows metadata about the secret without exposing the secret value.

---

## 3. Retrieve Secret for Testing

```bash
aws secretsmanager get-secret-value \
  --secret-id $SECRET_NAME \
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

# Lab 15: Create Kubernetes Secret from Secrets Manager Value

For this beginner lab, create a Kubernetes secret manually using the AWS Secrets Manager value.

## 1. Read Secret Values into Environment Variables

```bash
export APP_USER=$(aws secretsmanager get-secret-value \
  --secret-id $SECRET_NAME \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query SecretString \
  --output text | jq -r .APP_USER)

export APP_PASSWORD=$(aws secretsmanager get-secret-value \
  --secret-id $SECRET_NAME \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query SecretString \
  --output text | jq -r .APP_PASSWORD)
```

What this does:

This reads the secret values from AWS Secrets Manager into environment variables.

---

## 2. Create Kubernetes Secret

```bash
kubectl create secret generic app-secret \
  --from-literal=APP_USER=$APP_USER \
  --from-literal=APP_PASSWORD=$APP_PASSWORD \
  -n $NAMESPACE \
  --dry-run=client -o yaml | kubectl apply -f -
```

What this does:

This creates or updates a Kubernetes Secret for the application.

Important DevSecOps note:

For production, use External Secrets Operator or AWS Secrets and Configuration Provider with the Secrets Store CSI Driver instead of manually copying secrets.

---

# Lab 16: Update Deployment to Use Kubernetes Secret

Edit the deployment in CodeCommit.

## 1. Update Deployment Manifest

```bash
cd ../$CODECOMMIT_REPO

cat > apps/day22-secure-app/deployment.yaml <<APPDEPLOY2
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-nginx
  namespace: ${NAMESPACE}
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
        image: ${ECR_REPO_URI}:${IMAGE_TAG}
        ports:
        - containerPort: 8080
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
APPDEPLOY2
```

What this does:

This updates the application so it consumes secrets from Kubernetes environment variables.

---

## 2. Commit and Push

```bash
git add apps/day22-secure-app/deployment.yaml
git commit -m "Use Kubernetes secret in Day22 app"
git push origin $CODECOMMIT_BRANCH
```

What this does:

This updates the desired state in AWS CodeCommit.

---

## 3. Reconcile Flux

```bash
flux reconcile source git flux-system -n $FLUX_NAMESPACE
flux reconcile kustomization day22-secure-app -n $FLUX_NAMESPACE
```

What this does:

This forces Flux to pull and apply the latest Git change.

---

## 4. Verify Rollout

```bash
kubectl rollout status deployment/secure-nginx -n $NAMESPACE
```

What this does:

This confirms that the deployment successfully rolled out.

---

# Lab 17: Verify Secret Injection

## 1. Get Pod Name

```bash
export POD_NAME=$(kubectl get pods -n $NAMESPACE \
  -l app=secure-nginx \
  -o jsonpath='{.items[0].metadata.name}')
```

---

## 2. Verify Environment Variable Names

```bash
kubectl exec -n $NAMESPACE $POD_NAME -- printenv | grep APP_
```

What this does:

This confirms that the pod has received secret-backed environment variables.

Security note:

Avoid printing actual secret values during real audits or production operations.

---

# Lab 18: Test GitOps Drift Detection

## 1. Manually Scale the Deployment

```bash
kubectl scale deployment secure-nginx \
  --replicas=1 \
  -n $NAMESPACE
```

What this does:

This manually changes the live cluster state outside Git.

---

## 2. Check Flux Status

```bash
flux get kustomizations
```

What this does:

This shows Flux reconciliation status.

---

## 3. Force Reconciliation

```bash
flux reconcile kustomization day22-secure-app \
  -n $FLUX_NAMESPACE
```

What this does:

This asks Flux to restore the desired state from CodeCommit.

---

## 4. Verify Replica Count Restored

```bash
kubectl get deployment secure-nginx \
  -n $NAMESPACE \
  -o jsonpath='{.spec.replicas}'
echo
```

Expected:

```text
2
```

---

# Lab 19: Progressive Delivery with Flagger Overview

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

# Lab 20: Install Flagger for Progressive Delivery Demo

## 1. Add Flagger Helm Repository

```bash
helm repo add flagger https://flagger.app
helm repo update
```

What this does:

This adds the Flagger Helm chart repository.

---

## 2. Install Flagger with Kubernetes Provider

```bash
helm upgrade -i flagger flagger/flagger \
  --namespace flagger-system \
  --create-namespace \
  --set meshProvider=kubernetes
```

What this does:

This installs Flagger in basic Kubernetes mode.

---

## 3. Verify Flagger Pods

```bash
kubectl get pods -n flagger-system
```

Expected:

```text
Running
```

---

# Lab 21: Progressive Delivery Strategy Example

## 1. Create Canary Manifest in Git

```bash
cd ../$CODECOMMIT_REPO

cat > apps/day22-secure-app/canary.yaml <<CANARY
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
    targetPort: 8080
  analysis:
    interval: 1m
    threshold: 3
    iterations: 3
CANARY
```

What this does:

This creates a Flagger Canary custom resource for the secure nginx deployment.

---

## 2. Update Application Kustomization

```bash
cat > apps/day22-secure-app/kustomization.yaml <<APPKUSTOMIZE2
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
  - canary.yaml
APPKUSTOMIZE2
```

What this does:

This adds the canary manifest to the resources managed by Flux.

---

## 3. Commit and Push

```bash
git add apps/day22-secure-app/canary.yaml apps/day22-secure-app/kustomization.yaml
git commit -m "Add Flagger canary configuration"
git push origin $CODECOMMIT_BRANCH
```

What this does:

This stores the progressive delivery configuration in CodeCommit.

---

## 4. Reconcile Flux

```bash
flux reconcile source git flux-system -n $FLUX_NAMESPACE
flux reconcile kustomization day22-secure-app -n $FLUX_NAMESPACE
```

What this does:

This applies the Flagger Canary resource to the EKS cluster.

---

## 5. Verify Canary

```bash
kubectl get canary -n $NAMESPACE
kubectl describe canary secure-nginx -n $NAMESPACE
```

What this does:

This verifies the progressive delivery custom resource.

Note:

For full traffic shifting, configure an ingress controller or service mesh integration.

---

# Lab 22: DevSecOps Validation Checklist

## 1. Verify No Secrets in Git

```bash
git grep -i "password" || true
git grep -i "secret" || true
git grep -i "token" || true
```

What this does:

This checks whether sensitive values were accidentally committed to Git.

Important:

The deployment may include the Kubernetes Secret name `app-secret`. That is acceptable. The actual secret values must not be present in Git.

---

## 2. Verify Kubernetes Secret Exists

```bash
kubectl get secret app-secret -n $NAMESPACE
```

---

## 3. Verify KMS Key

```bash
aws kms describe-key \
  --key-id $KMS_ALIAS \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

---

## 4. Verify Secrets Manager Secret

```bash
aws secretsmanager describe-secret \
  --secret-id $SECRET_NAME \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

---

## 5. Verify ECR Image Scan Findings

```bash
aws ecr describe-image-scan-findings \
  --repository-name $ECR_REPO \
  --image-id imageTag=$IMAGE_TAG \
  --region $AWS_REGION \
  --profile $AWS_PROFILE \
  --query 'imageScanFindings.findingSeverityCounts' \
  --output table
```

What this does:

This reviews image scan severity counts for the pushed ECR image.

---

## 6. Verify Pod Security Context

```bash
kubectl get deployment secure-nginx \
  -n $NAMESPACE \
  -o yaml | grep -A30 securityContext
```

---

## 7. Verify Flux Reconciliation

```bash
flux get sources git
flux get kustomizations
```

---

# Lab 23: Troubleshooting

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
- Missing image pull permissions

---

## Flux Cannot Pull CodeCommit Repository

```bash
flux get sources git
kubectl describe gitrepository flux-system -n $FLUX_NAMESPACE
kubectl get secret flux-system -n $FLUX_NAMESPACE -o yaml
```

Common causes:

- Wrong CodeCommit repository URL
- IAM user does not have CodeCommit read permission
- Service-specific credentials are incorrect
- Secret name does not match `secretRef.name`
- Branch name is incorrect

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
- Flagger CRD missing before applying Canary resource

---

## Pod Fails to Start

```bash
kubectl get pods -n $NAMESPACE
kubectl describe pod <pod-name> -n $NAMESPACE
kubectl logs <pod-name> -n $NAMESPACE
```

Common causes:

- ECR image URI is wrong
- Node role cannot pull from ECR
- Container port mismatch
- Security context incompatible with image
- Missing Kubernetes secret
- Resource constraints

---

## Image Pull Error from ECR

Check pod events:

```bash
kubectl describe pod <pod-name> -n $NAMESPACE
```

Verify image exists:

```bash
aws ecr describe-images \
  --repository-name $ECR_REPO \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

Verify node IAM role has ECR pull permissions:

```bash
aws iam list-attached-role-policies \
  --role-name <eks-node-role-name> \
  --profile $AWS_PROFILE
```

The node role should usually have `AmazonEC2ContainerRegistryReadOnly` or equivalent ECR pull permissions.

---

## Secret Not Found

```bash
kubectl get secret app-secret -n $NAMESPACE
```

If missing, recreate from Secrets Manager:

```bash
kubectl create secret generic app-secret \
  --from-literal=APP_USER=$APP_USER \
  --from-literal=APP_PASSWORD=$APP_PASSWORD \
  -n $NAMESPACE \
  --dry-run=client -o yaml | kubectl apply -f -
```

---

# Lab 24: Cleanup

## 1. Delete Flagger

```bash
helm uninstall flagger -n flagger-system
kubectl delete namespace flagger-system --ignore-not-found=true
```

---

## 2. Delete Kubernetes Secret

```bash
kubectl delete secret app-secret -n $NAMESPACE --ignore-not-found=true
```

---

## 3. Delete Application Namespace

```bash
kubectl delete namespace $NAMESPACE --ignore-not-found=true
```

---

## 4. Uninstall Flux

```bash
flux uninstall --namespace=$FLUX_NAMESPACE
```

Confirm when prompted.

Alternative non-interactive cleanup:

```bash
flux uninstall --namespace=$FLUX_NAMESPACE --silent
```

---

## 5. Delete Secrets Manager Secret

```bash
aws secretsmanager delete-secret \
  --secret-id $SECRET_NAME \
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
  --alias-name $KMS_ALIAS \
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

## 8. Delete ECR Images

```bash
aws ecr batch-delete-image \
  --repository-name $ECR_REPO \
  --image-ids imageTag=$IMAGE_TAG \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This deletes the sample image from ECR.

---

## 9. Delete ECR Repository

```bash
aws ecr delete-repository \
  --repository-name $ECR_REPO \
  --force \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This deletes the ECR repository.

---

## 10. Delete CodeCommit Repository

```bash
aws codecommit delete-repository \
  --repository-name $CODECOMMIT_REPO \
  --region $AWS_REGION \
  --profile $AWS_PROFILE
```

What this does:

This deletes the CodeCommit GitOps repository.

---

## 11. Delete IAM Credentials and User for Flux

List service-specific credentials:

```bash
aws iam list-service-specific-credentials \
  --user-name flux-codecommit-day22 \
  --service-name codecommit.amazonaws.com \
  --profile $AWS_PROFILE
```

Delete the service-specific credential:

```bash
aws iam delete-service-specific-credential \
  --user-name flux-codecommit-day22 \
  --service-specific-credential-id <service-specific-credential-id> \
  --profile $AWS_PROFILE
```

Detach the policy:

```bash
aws iam detach-user-policy \
  --user-name flux-codecommit-day22 \
  --policy-arn arn:aws:iam::aws:policy/AWSCodeCommitReadOnly \
  --profile $AWS_PROFILE
```

Delete the IAM user:

```bash
aws iam delete-user \
  --user-name flux-codecommit-day22 \
  --profile $AWS_PROFILE
```

What this does:

This removes the dedicated IAM user and credentials created for Flux CodeCommit access.

---

## 12. Verify Cleanup

```bash
kubectl get namespaces | grep -E 'day22|flux-system|flagger-system' || true
aws ecr describe-repositories --repository-names $ECR_REPO --region $AWS_REGION --profile $AWS_PROFILE || true
aws codecommit get-repository --repository-name $CODECOMMIT_REPO --region $AWS_REGION --profile $AWS_PROFILE || true
```

Expected:

No Day 22 Kubernetes namespaces, ECR repository, or CodeCommit repository should remain.

---

# Challenge Exercise

Implement a secure AWS-native GitOps workflow:

1. Create a CodeCommit repository.
2. Create an ECR repository.
3. Build and push a container image to ECR.
4. Install Flux CD on EKS.
5. Connect Flux to CodeCommit.
6. Deploy a secure application through Flux.
7. Store application secrets in AWS Secrets Manager.
8. Encrypt the secret with AWS KMS.
9. Inject secret values into Kubernetes securely.
10. Add progressive delivery configuration with Flagger.
11. Validate Flux reconciliation and drift correction.
12. Confirm no secrets are stored in Git.
13. Document all commands and screenshots.

---

# Lab Deliverables

Submit:

- AWS CLI identity screenshot
- Flux CLI version screenshot
- CodeCommit repository screenshot
- ECR repository and image screenshot
- Docker build and push output
- Flux installation output
- Flux pods screenshot
- Git repository structure screenshot
- Flux GitRepository screenshot
- Flux Kustomization screenshot
- Application pods screenshot
- KMS key screenshot
- Secrets Manager screenshot
- Kubernetes secret screenshot
- Secure deployment YAML screenshot
- ECR image scan findings screenshot
- Progressive delivery canary screenshot
- Drift detection screenshot
- Cleanup verification screenshot

---

# Expected Learning Outcomes

After completing this lab, participants should be able to:

- Explain Flux CD and GitOps workflow.
- Use AWS CodeCommit as the Git source for Flux.
- Use Amazon ECR as the container image registry.
- Deploy Kubernetes applications from Git using Flux.
- Understand progressive delivery principles.
- Describe how Flagger supports canary deployments.
- Use AWS KMS for secret encryption.
- Store secrets in AWS Secrets Manager.
- Understand secure deployment practices.
- Apply basic DevSecOps controls to Kubernetes workloads.
- Troubleshoot Flux reconciliation issues.
- Clean up Flux, Kubernetes, ECR, CodeCommit, KMS, and Secrets Manager resources safely.
