# Day 22 – 16-Jun-2026 (Tuesday)
# Module 5 & 6 – GitOps, Progressive Delivery & DevSecOps

# Theory: Flux CD, Progressive Delivery & DevSecOps Fundamentals

## Learning Objectives

By the end of this session, participants will be able to:

- Understand Flux CD architecture and GitOps workflows.
- Compare Flux CD and ArgoCD.
- Learn Progressive Delivery concepts.
- Understand Canary and Blue-Green deployments.
- Learn DevSecOps principles.
- Understand secrets management in Kubernetes.
- Integrate AWS KMS and Secrets Manager with EKS.
- Build secure deployment pipelines.

---

# 1. Introduction to Flux CD

Flux CD is a GitOps Continuous Delivery tool for Kubernetes.

Purpose:

- Automate deployments
- Monitor Git repositories
- Synchronize Kubernetes clusters

Git Repository
↓
Flux CD
↓
Amazon EKS
↓
Applications

---

# 2. Why Flux CD?

Benefits:

- Git-driven deployments
- Lightweight architecture
- Kubernetes-native
- Continuous reconciliation
- Strong ecosystem integration

---

# 3. Flux CD Architecture

Core Components:

- Source Controller
- Kustomize Controller
- Helm Controller
- Notification Controller
- Image Automation Controller

---

# 4. Source Controller

Responsibilities:

- Connect to Git repositories
- Download manifests
- Monitor repository changes

Supported Sources:

- GitHub
- GitLab
- Bitbucket

---

# 5. Kustomize Controller

Functions:

- Apply manifests
- Reconcile cluster state
- Manage environments

---

# 6. Helm Controller

Manages:

- Helm repositories
- Helm releases
- Chart upgrades

Benefits:

- Automated Helm deployments

---

# 7. Flux CD Workflow

Developer
↓
Git Commit
↓
Git Repository
↓
Flux CD
↓
Kubernetes Cluster

Continuous reconciliation ensures consistency.

---

# 8. Flux CD vs ArgoCD

| Feature | Flux CD | ArgoCD |
|----------|----------|---------|
| UI | Limited | Rich UI |
| GitOps | Yes | Yes |
| Helm Support | Yes | Yes |
| Reconciliation | Yes | Yes |
| Complexity | Lightweight | Feature Rich |

---

# 9. Progressive Delivery

Progressive Delivery reduces deployment risk.

Benefits:

- Safer releases
- Faster recovery
- Controlled rollouts

Strategies:

- Canary Deployment
- Blue-Green Deployment
- Feature Flags

---

# 10. Canary Deployments

Release application gradually.

Example:

Version 1 → 90%
Version 2 → 10%

Monitor behavior before full rollout.

---

# 11. Blue-Green Deployments

Two environments:

Blue = Current Production

Green = New Version

Traffic switched after validation.

Benefits:

- Near-zero downtime
- Easy rollback

---

# 12. Feature Flags

Enable features without redeployment.

Benefits:

- Faster testing
- Reduced risk

---

# 13. Introduction to DevSecOps

DevSecOps integrates security into:

- Development
- CI/CD
- Operations

Security becomes everyone's responsibility.

---

# 14. DevSecOps Principles

Shift Security Left

Security checks occur early.

Benefits:

- Reduced vulnerabilities
- Faster remediation

---

# 15. Kubernetes Security Considerations

Protect:

- Images
- Secrets
- Network Traffic
- Access Controls

---

# 16. Secrets Management

Never store:

- Passwords
- API Keys
- Tokens

Directly in YAML files.

Use:

- Kubernetes Secrets
- AWS Secrets Manager

---

# 17. AWS Secrets Manager

Centralized secret storage.

Features:

- Encryption
- Rotation
- Auditing

Examples:

- Database Passwords
- API Keys

---

# 18. AWS KMS

Key Management Service

Provides:

- Encryption Keys
- Secret Protection
- Compliance Support

Used by:

- EBS
- EFS
- Secrets Manager

---

# 19. EKS Security Best Practices

- Use IAM Roles for Service Accounts (IRSA)
- Enable KMS encryption
- Use private ECR repositories
- Implement RBAC
- Rotate credentials
- Enable audit logging

---

# 20. Secure GitOps Workflow

Developer
↓
Git Repository
↓
Flux CD
↓
Secrets Manager
↓
EKS
↓
Application

Benefits:

- Automated deployments
- Secure secrets handling
- Auditability

---

# Summary

Topics Covered:

✓ Flux CD Architecture

✓ GitOps Automation

✓ Progressive Delivery

✓ Canary Deployments

✓ Blue-Green Deployments

✓ DevSecOps Principles

✓ AWS KMS

✓ AWS Secrets Manager

✓ Kubernetes Security

✓ Secure GitOps Workflows

Next Session:
Advanced DevSecOps and Policy Enforcement
