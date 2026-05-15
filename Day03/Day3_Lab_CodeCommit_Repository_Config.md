# Lab.md — Day 3 AWS CodeCommit Integration & Repository Configuration

# Day 3 — CodeCommit Integration and Repository Configuration

## Lab Duration
**2 Hours**

---

# Lab Objectives

By the end of this lab, learners will be able to:

- Create AWS CodeCommit repositories
- Configure IAM access
- Clone repositories locally
- Push code to CodeCommit
- Configure repository settings
- Monitor repository activity

---

# Prerequisites

- AWS Account
- IAM User with CodeCommit permissions
- Git installed
- AWS CLI installed
- Internet connection

---

# AWS Resources Used

| Service | Purpose |
|---|---|
| CodeCommit | Source repository |
| IAM | Authentication |
| CloudWatch Logs | Monitoring |
| CloudTrail | Audit logging |

---

# Part 1: Configure AWS CLI

## Verify AWS CLI

```bash
aws --version
```

---

## Configure AWS CLI

```bash
aws configure
```

Provide:

```text
AWS Access Key ID
AWS Secret Access Key
Region
Output Format
```

---

# Part 2: Create IAM Policy

## Navigate to IAM

```text
AWS Console → IAM → Policies
```

---

## Create CodeCommit Policy

Use JSON:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "codecommit:*"
      ],
      "Resource": "*"
    }
  ]
}
```

---

# Part 3: Create CodeCommit Repository

## Using AWS Console

Navigate:

```text
Developer Tools → CodeCommit
```

Click:

```text
Create Repository
```

---

## Repository Details

| Field | Value |
|---|---|
| Repository Name | DevOpsRepo |
| Description | DevOps Training Repository |

---

# Part 4: Create Repository Using AWS CLI

```bash
aws codecommit create-repository \
--repository-name DevOpsRepo \
--repository-description "DevOps Training Repository"
```

---

# Part 5: Clone Repository

## Copy Clone URL

Example:

```text
https://git-codecommit.us-east-1.amazonaws.com/v1/repos/DevOpsRepo
```

---

## Clone Repository

```bash
git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/DevOpsRepo
```

---

# Part 6: Add Files to Repository

## Navigate to Repository

```bash
cd DevOpsRepo
```

---

## Create README File

```bash
echo "# AWS CodeCommit Lab" > README.md
```

---

## Add File

```bash
git add README.md
```

---

## Commit Changes

```bash
git commit -m "Initial commit"
```

---

## Push to CodeCommit

```bash
git push origin main
```

---

# Part 7: Branch Configuration

## Create Feature Branch

```bash
git checkout -b feature-login
```

---

## Add New File

```bash
echo "Login module" > login.txt
```

---

## Commit Changes

```bash
git add login.txt
git commit -m "Added login feature"
```

---

## Push Branch

```bash
git push origin feature-login
```

---

# Part 8: Pull Request Configuration

## Create Pull Request

Navigate:

```text
CodeCommit → Pull Requests → Create Pull Request
```

---

## Configure Pull Request

| Field | Value |
|---|---|
| Source Branch | feature-login |
| Destination Branch | main |

---

# Part 9: Repository Settings

## Configure Repository

Navigate:

```text
Repository → Settings
```

---

# Configure Notifications

Enable:

- Repository activity notifications
- Pull request alerts
- Merge notifications

---

# Part 10: Enable CloudTrail Logging

## Navigate to CloudTrail

```text
AWS Console → CloudTrail
```

---

## Create Trail

Enable:

- Management events
- Read/Write events

---

# Part 11: Monitor Logs in CloudWatch

## Navigate to CloudWatch

```text
CloudWatch → Logs
```

---

## View Logs

Monitor:

- Repository access
- Git operations
- Authentication logs

---

# Part 12: Simulate Repository Activity

## Pull Latest Changes

```bash
git pull origin main
```

---

## Create Additional File

```bash
echo "Deployment notes" > deployment.txt
```

---

## Commit and Push

```bash
git add deployment.txt
git commit -m "Added deployment notes"
git push origin main
```

---

# Validation Checklist

| Task | Status |
|---|---|
| AWS CLI configured | ✅ |
| IAM policy created | ✅ |
| CodeCommit repository created | ✅ |
| Repository cloned | ✅ |
| README pushed | ✅ |
| Feature branch created | ✅ |
| Pull request created | ✅ |
| CloudWatch logs enabled | ✅ |

---

# Common Commands

| Command | Purpose |
|---|---|
| aws configure | Configure AWS CLI |
| git clone | Clone repository |
| git add . | Stage changes |
| git commit | Commit changes |
| git push | Push changes |
| git pull | Pull updates |
| git branch | View branches |

---

# Lab Assignment

## Create Additional Branches

```text
feature-payment
feature-dashboard
feature-monitoring
```

Each branch should:

- Create one file
- Commit changes
- Push to CodeCommit
- Create pull request

---

# Expected Repository Structure

```text
DevOpsRepo/
├── README.md
├── login.txt
├── deployment.txt
├── payment.txt
├── dashboard.txt
└── monitoring.txt
```

---

# Summary

In this lab, you practiced:

- AWS CodeCommit setup
- IAM configuration
- Repository management
- Branching strategies
- Pull requests
- CloudWatch monitoring
- Repository configuration

---
