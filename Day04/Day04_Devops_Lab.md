# Day 04 – DevOps Lab
## Module 1 – DevOps & CI/CD Fundamentals
### Date: 21-May-2026 (Thursday)

---

# Lab 1 – Buildspec Creation

## Objective
Create a `buildspec.yml` file and configure AWS CodeBuild to automate application builds.

---

# Prerequisites

Before starting:
- AWS Account
- IAM permissions for CodeBuild and S3
- Source code repository or sample application
- S3 bucket for artifacts

---

# Architecture Overview

```text
Developer → Source Repository → AWS CodeBuild → Amazon S3
                                 ↓
                           CloudWatch Logs
```

---

# Step 1 – Create an S3 Bucket

1. Open AWS Console
2. Navigate to Amazon S3
3. Click **Create bucket**
4. Enter bucket name:
   - `devops-training-build-artifacts`
5. Keep default settings
6. Click **Create bucket**

---

# Step 2 – Create IAM Role for CodeBuild

1. Open IAM Console
2. Create a new role
3. Select:
   - AWS Service
   - CodeBuild
4. Attach policies:
   - AmazonS3FullAccess
   - CloudWatchLogsFullAccess
5. Name the role:
   - `CodeBuildTrainingRole`

---

# Step 3 – Create Sample Application

Create a simple application folder.

Example structure:

```text
sample-app/
 ├── index.html
 └── buildspec.yml
```

Create a simple `index.html`:

```html
<h1>DevOps Training Build Successful</h1>
```

---

# Step 4 – Create buildspec.yml

Create a file named `buildspec.yml`.

Example:

```yaml
version: 0.2

phases:
  install:
    commands:
      - echo Installing dependencies

  pre_build:
    commands:
      - echo Preparing build environment

  build:
    commands:
      - echo Build started
      - mkdir output
      - cp index.html output/

  post_build:
    commands:
      - echo Build completed

artifacts:
  files:
    - '**/*'
  base-directory: output
```

---

# Understanding buildspec.yml

| Section | Purpose |
|---|---|
| version | Buildspec version |
| phases | Build lifecycle stages |
| commands | Commands executed |
| artifacts | Files generated after build |

---

# Lab 2 – Automated Builds

## Objective
Configure AWS CodeBuild to run builds automatically.

---

# Step 1 – Create CodeBuild Project

1. Open AWS CodeBuild
2. Click **Create build project**
3. Project name:
   - `DevOpsTrainingProject`

---

# Step 2 – Configure Source

Select source provider:
- GitHub
- CodeCommit
- S3

Example:
- Source Provider: GitHub
- Repository: Public repository

---

# Step 3 – Configure Environment

Environment settings:
- Managed image
- Ubuntu
- Standard runtime
- Latest image

Service role:
- Use existing role
- `CodeBuildTrainingRole`

---

# Step 4 – Configure Buildspec

Choose:
- Use buildspec.yml in source code

---

# Step 5 – Configure Artifacts

Artifact type:
- Amazon S3

Bucket:
- `devops-training-build-artifacts`

Packaging:
- ZIP

---

# Step 6 – Configure Logs

Enable:
- CloudWatch Logs

This allows monitoring:
- Build progress
- Errors
- Execution logs

---

# Step 7 – Start Build

1. Click **Create build project**
2. Click **Start build**
3. Monitor phases:
   - INSTALL
   - PRE_BUILD
   - BUILD
   - POST_BUILD

---

# Step 8 – Verify Output

## Check CloudWatch Logs
Review build logs and status.

## Check S3 Bucket
Verify artifact ZIP file is uploaded.

---

# Expected Output

After successful build:
- Build status = SUCCESS
- Artifacts available in S3
- Logs visible in CloudWatch

---

# Troubleshooting

## Build Failed
Check:
- IAM permissions
- Syntax errors in buildspec.yml

## Artifact Missing
Verify:
- S3 permissions
- Artifact configuration

## Logs Not Available
Ensure:
- CloudWatch Logs enabled
- IAM role has logging permissions

---

# Best Practices

- Store buildspec.yml in repository
- Use least privilege IAM access
- Enable CloudWatch monitoring
- Use version control for configurations
- Keep build environments consistent

---

# AWS Services Used

| Service | Purpose |
|---|---|
| CodeBuild | Automated builds |
| S3 | Artifact storage |
| IAM | Permissions |
| CloudWatch | Logs and monitoring |

---

# Summary

In this lab learners practiced:
- Creating buildspec.yml
- Configuring AWS CodeBuild
- Running automated builds
- Uploading artifacts to S3
- Monitoring builds using CloudWatch

This lab provides hands-on experience with AWS CI/CD automation.
