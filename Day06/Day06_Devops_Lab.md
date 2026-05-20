# Day 06 – DevOps Lab
## Module 1 – DevOps & CI/CD Fundamentals
### Date: 25-May-2026 (Monday)

---

# Lab – Complete CI/CD Pipeline Implementation

## Objective

Build a complete CI/CD pipeline using:
- AWS CodePipeline
- AWS CodeBuild
- AWS CodeDeploy
- Amazon SNS

---

# Prerequisites

Before starting:
- AWS Account
- GitHub or CodeCommit repository
- EC2 instances
- CodeDeploy agent installed
- IAM permissions configured

---

# CI/CD Architecture

```text
Developer
   ↓
GitHub Repository
   ↓
CodePipeline
   ↓
CodeBuild
   ↓
CodeDeploy
   ↓
EC2 Deployment
   ↓
SNS Notifications
```

---

# Step 1 – Create Source Repository

Use:
- GitHub
OR
- AWS CodeCommit

Example application structure:

```text
sample-app/
 ├── appspec.yml
 ├── buildspec.yml
 ├── index.html
 └── scripts/
      ├── install.sh
      └── start_server.sh
```

---

# Step 2 – Create buildspec.yml

```yaml
version: 0.2

phases:
  install:
    commands:
      - echo Installing dependencies

  build:
    commands:
      - echo Building application

artifacts:
  files:
    - '**/*'
```

---

# Step 3 – Create appspec.yml

```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /var/www/html

hooks:
  ApplicationStart:
    - location: scripts/start_server.sh
```

---

# Step 4 – Launch EC2 Instance

Create Amazon Linux 2 instance.

Security Group:
- HTTP (80)
- SSH (22)

---

# Step 5 – Install CodeDeploy Agent

Connect to EC2:

```bash
sudo yum update -y
sudo yum install ruby wget -y

cd /home/ec2-user
wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
```

Verify:

```bash
sudo service codedeploy-agent status
```

---

# Step 6 – Create IAM Roles

## CodeBuild Role
Attach:
- AmazonS3FullAccess
- CloudWatchLogsFullAccess

## CodeDeploy Role
Attach:
- AWSCodeDeployRole

## CodePipeline Role
Attach:
- AWSCodePipelineFullAccess

---

# Step 7 – Create SNS Topic

1. Open Amazon SNS
2. Create Topic
3. Type:
   - Standard
4. Topic name:
   - DevOpsPipelineNotifications

Add email subscription for alerts.

---

# Step 8 – Create CodeBuild Project

1. Open AWS CodeBuild
2. Create build project

Configuration:
- Source provider = CodePipeline
- Environment = Managed Image
- Buildspec = buildspec.yml

Artifacts:
- Managed by CodePipeline

---

# Step 9 – Create CodeDeploy Application

1. Open AWS CodeDeploy
2. Create application
3. Platform:
   - EC2/On-Premises

Create deployment group:
- Select EC2 instances
- Configure deployment settings

---

# Step 10 – Create CodePipeline

1. Open AWS CodePipeline
2. Click Create Pipeline

Pipeline stages:

---

## Source Stage

Provider:
- GitHub or CodeCommit

Configure repository and branch.

---

## Build Stage

Provider:
- AWS CodeBuild

Select:
- Previously created build project

---

## Deploy Stage

Provider:
- AWS CodeDeploy

Select:
- Application
- Deployment Group

---

# Step 11 – Configure SNS Notifications

Enable notifications for:
- Pipeline success
- Pipeline failure
- Deployment status

Subscribe:
- Email endpoint

---

# Step 12 – Run Pipeline

1. Commit code changes
2. Push to repository
3. Pipeline automatically triggers

Observe stages:
- Source
- Build
- Deploy

---

# Step 13 – Validate Deployment

Verify:
- Application deployed successfully
- EC2 web server accessible
- SNS notifications received

---

# Working Module Example

## CI/CD Workflow

### Source
Developer pushes code to GitHub.

### Build
CodeBuild:
- Compiles application
- Executes build commands
- Generates artifacts

### Deploy
CodeDeploy:
- Copies files to EC2
- Starts application services

### Notification
SNS sends pipeline status alerts.

---

# Sample Deployment Validation

Access EC2 public IP:

```text
http://EC2-Public-IP
```

Expected output:

```html
<h1>CI/CD Pipeline Deployment Successful</h1>
```

---

# Monitoring Pipeline

Use:
- AWS CodePipeline Dashboard
- CloudWatch Logs
- SNS Email Alerts

---

# Troubleshooting

## Build Failed
Check:
- buildspec.yml syntax
- IAM permissions

---

## Deployment Failed
Check:
- appspec.yml syntax
- CodeDeploy agent status

---

## SNS Notifications Missing
Verify:
- Subscription confirmation
- SNS permissions

---

# Best Practices

- Store configurations in Git
- Use least privilege IAM access
- Enable CloudWatch monitoring
- Configure rollback mechanisms
- Automate testing before deployment

---

# AWS Services Used

| Service | Purpose |
|---|---|
| CodePipeline | CI/CD orchestration |
| CodeBuild | Build automation |
| CodeDeploy | Application deployment |
| SNS | Notifications and alerts |

---

# Summary

In this lab learners practiced:
- Building an end-to-end CI/CD pipeline
- Integrating CodeBuild and CodeDeploy
- Automating deployments
- Configuring SNS notifications
- Monitoring pipeline execution

This lab provides practical hands-on experience with AWS DevOps CI/CD automation.
