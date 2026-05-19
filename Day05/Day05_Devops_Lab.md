# Day 05 – DevOps Lab
## Module 1 – DevOps & CI/CD Fundamentals
### Date: 22-May-2026 (Friday)

---

# Lab 1 – Blue/Green Deployments

## Objective

Learn how to perform Blue/Green deployments using AWS CodeDeploy and EC2 instances.

---

# Prerequisites

Before starting:
- AWS account
- EC2 instances
- IAM permissions
- CodeDeploy agent installed
- Application package uploaded to S3

---

# Blue/Green Deployment Architecture

```text
Users
   ↓
Elastic Load Balancer
   ↓
Blue Environment (Current)
Green Environment (New)
```

---

# Step 1 – Launch EC2 Instances

Create:
- 2 EC2 instances for Blue environment
- 2 EC2 instances for Green environment

Recommended settings:
- Amazon Linux 2
- t2.micro
- Security group with HTTP access

---

# Step 2 – Install CodeDeploy Agent

Connect to EC2 instance:

```bash
sudo yum update -y
sudo yum install ruby wget -y

cd /home/ec2-user
wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
```

Verify agent:

```bash
sudo service codedeploy-agent status
```

---

# Step 3 – Create Application Files

Example structure:

```text
sample-app/
 ├── appspec.yml
 ├── index.html
 └── scripts/
      ├── install.sh
      └── start_server.sh
```

---

# Step 4 – Create appspec.yml

```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /var/www/html

hooks:
  BeforeInstall:
    - location: scripts/install.sh

  ApplicationStart:
    - location: scripts/start_server.sh
```

---

# Step 5 – Upload Application Revision

Compress files:

```bash
zip -r sample-app.zip .
```

Upload ZIP to S3 bucket.

---

# Step 6 – Create CodeDeploy Application

1. Open AWS CodeDeploy Console
2. Click **Create Application**
3. Application name:
   - DevOpsTrainingApp
4. Compute platform:
   - EC2/On-Premises

---

# Step 7 – Create Deployment Group

Configure:
- EC2 instances
- Load balancer
- Auto Scaling group (optional)

Deployment type:
- Blue/Green

---

# Step 8 – Configure Load Balancer

Create an Application Load Balancer (ALB).

Attach:
- Blue target group
- Green target group

---

# Step 9 – Start Deployment

1. Create deployment
2. Select revision from S3
3. Monitor deployment status

---

# Step 10 – Validate Deployment

Verify:
- Traffic switches successfully
- Application accessible
- No downtime observed

---

# Lab 2 – Canary Deployments

## Objective

Implement a Canary deployment strategy using AWS CodeDeploy and ECS.

---

# Canary Deployment Overview

Traffic distribution example:

```text
90% → Old Version
10% → New Version
```

If successful:
- Increase gradually to 100%

---

# Step 1 – Create ECS Cluster

1. Open Amazon ECS
2. Create cluster
3. Launch type:
   - Fargate or EC2

---

# Step 2 – Create ECS Service

Deploy containerized application.

Example:
- NGINX container
- Sample web application

---

# Step 3 – Configure Load Balancer

Create:
- Target groups
- Listener rules

Enable weighted traffic routing.

---

# Step 4 – Create CodeDeploy ECS Application

1. Open CodeDeploy
2. Create application
3. Select:
   - ECS platform

---

# Step 5 – Configure Deployment Group

Settings:
- ECS service
- Load balancer
- Canary deployment configuration

Example:
- 10% traffic for 5 minutes
- Remaining 90% after validation

---

# Step 6 – Deploy New Version

Push updated container image.

Trigger deployment.

---

# Step 7 – Monitor Deployment

Use:
- CloudWatch Metrics
- ECS Service Events
- Load Balancer metrics

Monitor:
- Error rates
- Response times
- Container health

---

# Step 8 – Validate Canary Deployment

Verify:
- Traffic gradually shifts
- No application failures
- ECS tasks remain healthy

---

# Automatic Rollback

Configure rollback if:
- CloudWatch alarm triggers
- Deployment fails
- Health checks fail

---

# Troubleshooting

## Deployment Failed
Check:
- IAM permissions
- AppSpec syntax
- ECS task health

## Load Balancer Issues
Verify:
- Listener rules
- Target group health checks

## CodeDeploy Agent Offline
Restart service:

```bash
sudo service codedeploy-agent restart
```

---

# Best Practices

- Always test in staging first
- Use automated rollback
- Monitor deployments continuously
- Use small canary percentages initially
- Enable health checks and alarms

---

# AWS Services Used

| Service | Purpose |
|---|---|
| CodeDeploy | Automated deployments |
| EC2 | Application hosting |
| ECS | Container deployment |
| ELB | Traffic routing |
| Auto Scaling | Dynamic scaling |

---

# Summary

In this lab learners practiced:
- Creating Blue/Green deployments
- Performing Canary deployments
- Configuring Load Balancers
- Using ECS with CodeDeploy
- Monitoring deployments with CloudWatch

This lab provides hands-on deployment automation experience using AWS DevOps services.
