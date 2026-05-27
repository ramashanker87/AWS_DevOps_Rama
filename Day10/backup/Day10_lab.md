# Day 10 – DevOps Lab
# Simple Amazon ECR & ECS Lab

## Module – Containers & Container Orchestration
### Date: 29-May-2026 (Friday)

---

# Lab Objective

In this lab you will:

- Create Docker container
- Push container image to Amazon ECR
- Deploy container to Amazon ECS
- Run application using ECS Fargate

This lab is simplified for beginners.

Estimated Time:

```text
1 Hour
```

---

# Architecture

```text
Docker Image
      ↓
Amazon ECR
      ↓
Amazon ECS Fargate
      ↓
Running Container
```

---

# AWS Resources Used

| Resource | Purpose |
|---|---|
| ECR | Container registry |
| ECS | Container deployment |
| IAM | Permissions |
| CloudWatch | Logs |

---

# Prerequisites

Before starting:

- AWS Account
- Docker installed locally
- AWS CLI configured

Recommended Region:

```text
us-east-1
```

---

# Part 1 – Create Docker Application

## Step 1 – Create Project Folder

Create folder:

```text
ecs-demo-app
```

---

# Step 2 – Create index.html

Create:

```html
<h1>Container deployed successfully on ECS</h1>
```

---

# Step 3 – Create Dockerfile

Create file:

```text
Dockerfile
```

Paste:

```dockerfile
FROM nginx:latest

COPY index.html /usr/share/nginx/html/index.html
```

---

# Step 4 – Build Docker Image

Run:

```bash
docker build -t ecs-demo .
```

Verify:

```bash
docker images
```

---

# Step 5 – Run Container Locally

Run:

```bash
docker run -d -p 8080:80 ecs-demo
```

Open browser:

```text
http://localhost:8080
```

Expected:

```html
<h1>Container deployed successfully on ECS</h1>
```

---

# Part 2 – Create Amazon ECR Repository

## Step 6 – Open Amazon ECR

Open:

```text
AWS Console → Amazon ECR
```

Click:

```text
Create Repository
```

Configuration:

| Setting | Value |
|---|---|
| Repository Name | ecs-demo-repo |

Create repository.

---

# Step 7 – View Push Commands

Open repository.

Click:

```text
View Push Commands
```

AWS provides commands automatically.

---

# Step 8 – Authenticate Docker to ECR

Run command from AWS console.

Example:

```bash
aws ecr get-login-password --region us-east-1 | \
docker login --username AWS --password-stdin \
<ACCOUNT-ID>.dkr.ecr.us-east-1.amazonaws.com
```

---

# Step 9 – Tag Docker Image

Run:

```bash
docker tag ecs-demo:latest \
<ACCOUNT-ID>.dkr.ecr.us-east-1.amazonaws.com/ecs-demo-repo:latest
```

---

# Step 10 – Push Image to ECR

Run:

```bash
docker push \
<ACCOUNT-ID>.dkr.ecr.us-east-1.amazonaws.com/ecs-demo-repo:latest
```

Verify image visible in ECR.

---

# Part 3 – Create ECS Cluster

## Step 11 – Open Amazon ECS

Open:

```text
AWS Console → ECS
```

Click:

```text
Create Cluster
```

Configuration:

| Setting | Value |
|---|---|
| Cluster Name | ecs-demo-cluster |
| Infrastructure | AWS Fargate |

Create cluster.

---

# Part 4 – Create Task Definition

## Step 12 – Create Task Definition

Open:

```text
ECS → Task Definitions → Create
```

Configuration:

| Setting | Value |
|---|---|
| Task Definition Family | ecs-demo-task |
| Launch Type | AWS Fargate |
| CPU | 0.25 vCPU |
| Memory | 0.5 GB |

---

# Step 13 – Add Container

Container Settings:

| Setting | Value |
|---|---|
| Name | ecs-demo-container |
| Image URI | ECR image URI |
| Port Mapping | 80 |

Create task definition.

---

# Part 5 – Create ECS Service

## Step 14 – Create ECS Service

Open:

```text
ECS → Cluster → Create Service
```

Configuration:

| Setting | Value |
|---|---|
| Launch Type | Fargate |
| Task Definition | ecs-demo-task |
| Service Name | ecs-demo-service |
| Desired Tasks | 1 |

Networking:

| Setting | Value |
|---|---|
| Public IP | Enabled |

Security Group:

Allow:

| Type | Port |
|---|---|
| HTTP | 80 |

Create service.

---

# Step 15 – Wait for Task to Run

Status should become:

```text
Running
```

---

# Step 16 – Open ECS Task Public IP

Open:

```text
ECS → Cluster → Tasks
```

Copy public IP.

Open browser:

```text
http://<TASK-PUBLIC-IP>
```

Expected:

```html
<h1>Container deployed successfully on ECS</h1>
```

---

# Container Security Demonstration

Explain:

- ECR stores images securely
- ECS isolates containers
- IAM controls permissions
- AWS supports image scanning

---

# Common Errors and Fixes

---

# Docker Not Running

Start Docker:

```bash
sudo systemctl start docker
```

---

# ECR Login Failed

Verify:

- AWS CLI configured
- Correct region used
- IAM permissions available

---

# ECS Task Not Running

Verify:

- Correct image URI
- Container port mapping
- Security Group allows HTTP

---

# Website Not Opening

Verify:

- Public IP enabled
- HTTP port 80 open
- Task status running

---

# Useful Commands

## View Docker Images

```bash
docker images
```

---

## View Running Containers

```bash
docker ps
```

---

## Stop Container

```bash
docker stop <container-id>
```

---

# Cleanup

To avoid AWS charges:

- Delete ECS service
- Delete ECS cluster
- Delete ECR repository

---

# Success Checklist

✅ Docker image created

✅ Image pushed to ECR

✅ ECS cluster created

✅ ECS task running

✅ Website accessible

---

# Summary

In this lab you completed:

- Docker container creation
- Amazon ECR image push
- ECS Fargate deployment
- Container deployment workflow
