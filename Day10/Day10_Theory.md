# Day 10 – DevOps Theory
# Amazon ECR, Container Security & Amazon ECS

## Module – Containers & Container Orchestration
### Date: 29-May-2026 (Friday)

---

# Learning Objectives

In this session you will learn:

- What are containers
- What is Docker
- What is Amazon ECR
- What is Amazon ECS
- Container security basics
- Container deployment workflow

---

# What are Containers?

Containers are:

```text
Lightweight packages containing application code and dependencies.
```

Containers ensure applications run consistently across environments.

---

# Why Containers are Important

Benefits:

| Benefit | Description |
|---|---|
| Portability | Run anywhere |
| Fast Deployment | Quick startup |
| Consistency | Same environment everywhere |
| Scalability | Easy scaling |
| Isolation | Applications separated |

---

# What is Docker?

Docker is:

```text
A platform used to create and run containers.
```

Docker packages applications into container images.

---

# Docker Workflow

```text
Application Code
      ↓
Docker Image
      ↓
Container
      ↓
Deployment
```

---

# What is Amazon ECR?

Amazon ECR means:

```text
Elastic Container Registry
```

It is AWS managed container image repository.

---

# Amazon ECR Features

| Feature | Purpose |
|---|---|
| Private Registry | Secure image storage |
| Image Versioning | Store multiple versions |
| AWS Integration | Works with ECS/EKS |
| Security Scanning | Detect vulnerabilities |
| High Availability | Managed by AWS |

---

# What is Container Security?

Container security means:

```text
Protecting container images and runtime environments.
```

---

# Common Container Security Risks

| Risk | Example |
|---|---|
| Vulnerable Packages | Old software libraries |
| Hardcoded Secrets | Passwords in image |
| Untrusted Images | Downloaded from unknown source |
| Open Ports | Unnecessary exposure |

---

# Container Security Best Practices

- Use official base images
- Scan images regularly
- Avoid hardcoded passwords
- Use least privilege access
- Keep images updated

---

# What is Amazon ECS?

Amazon ECS means:

```text
Elastic Container Service
```

It is AWS managed container orchestration service.

---

# ECS Responsibilities

ECS helps:

- Run containers
- Manage scaling
- Monitor containers
- Deploy applications
- Manage clusters

---

# ECS Components

| Component | Purpose |
|---|---|
| Cluster | Group of servers |
| Task Definition | Container configuration |
| Task | Running container |
| Service | Manages tasks |

---

# ECS Workflow

```text
Docker Image
      ↓
Push to Amazon ECR
      ↓
ECS Pulls Image
      ↓
Container Runs
```

---

# What is a Task Definition?

Task Definition contains:

- Docker image
- CPU settings
- Memory settings
- Ports
- Environment variables

---

# ECS Launch Types

| Launch Type | Description |
|---|---|
| EC2 | Containers on EC2 |
| Fargate | Serverless containers |

---

# Why Use ECS Fargate?

Benefits:

- No server management
- Automatic scaling
- Simplified deployment
- Beginner friendly

---

# Real-World CI/CD Flow

```text
Developer Pushes Code
          ↓
Docker Build
          ↓
Push Image to ECR
          ↓
Deploy Container to ECS
```

---

# AWS Services Used

| Service | Purpose |
|---|---|
| ECR | Container registry |
| ECS | Container deployment |
| IAM | Access control |
| CloudWatch | Monitoring |

---

# Benefits of Containers in DevOps

- Faster deployments
- Better scalability
- Environment consistency
- Easy automation
- Simplified CI/CD

---

# Summary

In this session you learned:

- Container fundamentals
- Docker basics
- Amazon ECR
- Container security
- Amazon ECS
- Container deployment workflow
