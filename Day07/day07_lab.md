# Day 7 – DevOps Lab

## Module 1 – DevOps & CI/CD Fundamentals
### Date: 26-May-2026 (Tuesday)

---

# Lab Guide – Jenkins & GitHub Actions Integration with AWS EC2

---

# Lab Objectives

In this lab you will:

- Install Jenkins on AWS EC2
- Configure Jenkins pipelines
- Deploy applications to EC2
- Create GitHub Actions workflows
- Automate deployments to AWS
- Practice real-world CI/CD automation

---

# AWS Resources Used

| Resource | Purpose |
|---|---|
| EC2 | Application hosting |
| Jenkins | CI/CD automation |
| GitHub Actions | Workflow automation |
| Security Groups | Network access |
| IAM | Access control |

---

# Common Prerequisites

Ensure you have:

- AWS Account
- GitHub account
- EC2 Key Pair
- Internet connectivity
- Basic Linux knowledge

---

# Part 1 – Create AWS EC2 Application Server

---

# Step 1 – Launch EC2 Instance

1. Open AWS Console
2. Navigate to EC2
3. Click **Launch Instance**

---

## Configure Instance

| Setting | Value |
|---|---|
| Name | app-server |
| AMI | Amazon Linux 2 |
| Instance Type | t2.micro |
| Key Pair | Select existing key pair |

---

## Configure Security Group

Allow inbound rules:

| Type | Port |
|---|---|
| SSH | 22 |
| HTTP | 80 |

---

# Step 2 – Connect to EC2

```bash
ssh -i mykey.pem ec2-user@<EC2-PUBLIC-IP>
```

---

# Step 3 – Install Apache

```bash
sudo yum update -y
sudo yum install httpd git -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

---

# Step 4 – Create Test Application

```bash
echo "<h1>Application Server Ready</h1>" | sudo tee /var/www/html/index.html
```

---

# Step 5 – Verify Application

Open browser:

```text
http://<EC2-PUBLIC-IP>
```

---

# Part 2 – Jenkins Integration Lab

---

# Step 1 – Launch Jenkins EC2 Instance

Create another EC2 instance.

---

## Jenkins EC2 Configuration

| Setting | Value |
|---|---|
| Name | jenkins-server |
| AMI | Amazon Linux 2 |
| Instance Type | t2.micro |

---

## Security Group

Allow:

| Type | Port |
|---|---|
| SSH | 22 |
| Custom TCP | 8080 |

---

# Step 2 – Connect to Jenkins Server

```bash
ssh -i mykey.pem ec2-user@<JENKINS-IP>
```

---

# Step 3 – Install Java

```bash
sudo yum update -y
sudo amazon-linux-extras install java-openjdk11 -y
```

---

# Step 4 – Install Jenkins

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

sudo yum install jenkins -y
```

---

# Step 5 – Start Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

# Step 6 – Access Jenkins

Open browser:

```text
http://<JENKINS-IP>:8080
```

---

# Step 7 – Get Initial Admin Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

# Step 8 – Install Suggested Plugins

Complete Jenkins setup wizard.

---

# Step 9 – Install Additional Plugins

Install:

- Git Plugin
- Pipeline Plugin
- SSH Agent Plugin

---

# Step 10 – Create GitHub Repository

Create repository:

```text
jenkins-ec2-demo
```

Add file:

## index.html

```html
<h1>Application deployed using Jenkins</h1>
```

---

# Step 11 – Add Jenkins Credentials

Go to:

```text
Manage Jenkins → Credentials
```

Add:

- SSH Username with private key
- Username: ec2-user
- Paste EC2 private key

Credential ID:

```text
ec2-key
```

---

# Step 12 – Create Jenkins Pipeline

Create pipeline job:

```text
jenkins-ec2-deploy
```

---

# Step 13 – Add Jenkins Pipeline Script

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/<USERNAME>/<REPO>.git'
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['ec2-key']) {
                    sh '''
                    scp -o StrictHostKeyChecking=no index.html ec2-user@<APP-IP>:/tmp/index.html

                    ssh -o StrictHostKeyChecking=no ec2-user@<APP-IP> "
                    sudo cp /tmp/index.html /var/www/html/index.html
                    sudo systemctl restart httpd
                    "
                    '''
                }
            }
        }
    }
}
```

---

# Step 14 – Build Pipeline

Click:

```text
Build Now
```

---

# Step 15 – Validate Deployment

Open:

```text
http://<APP-IP>
```

Expected:

```text
Application deployed using Jenkins
```

---

# Part 3 – GitHub Actions Integration Lab

---

# Step 1 – Create GitHub Repository

Create repository:

```text
github-actions-demo
```

---

# Step 2 – Add Application File

Create:

## index.html

```html
<h1>Application deployed using GitHub Actions</h1>
```

---

# Step 3 – Add GitHub Secrets

Go to:

```text
Repository → Settings → Secrets and variables → Actions
```

Create:

| Secret | Value |
|---|---|
| EC2_HOST | EC2 Public IP |
| EC2_USER | ec2-user |
| EC2_SSH_KEY | Private key content |

---

# Step 4 – Create Workflow Directory

```text
.github/workflows/
```

---

# Step 5 – Create Workflow File

Create:

```text
.github/workflows/deploy.yml
```

---

# Step 6 – Add Workflow Configuration

```yaml
name: Deploy to AWS EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Configure SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.EC2_SSH_KEY }}" > ~/.ssh/ec2_key
          chmod 600 ~/.ssh/ec2_key
          ssh-keyscan -H ${{ secrets.EC2_HOST }} >> ~/.ssh/known_hosts

      - name: Copy File
        run: |
          scp -i ~/.ssh/ec2_key index.html           ${{ secrets.EC2_USER }}@${{ secrets.EC2_HOST }}:/tmp/index.html

      - name: Deploy to EC2
        run: |
          ssh -i ~/.ssh/ec2_key           ${{ secrets.EC2_USER }}@${{ secrets.EC2_HOST }} "
          sudo cp /tmp/index.html /var/www/html/index.html
          sudo systemctl restart httpd
          "
```

---

# Step 7 – Commit and Push

```bash
git add .
git commit -m "Add GitHub Actions workflow"
git push origin main
```

---

# Step 8 – Monitor Workflow

1. Open GitHub repository
2. Click **Actions**
3. Open workflow run

Verify:

- Build successful
- Deployment successful

---

# Step 9 – Validate GitHub Actions Deployment

Open:

```text
http://<EC2-PUBLIC-IP>
```

Expected:

```text
Application deployed using GitHub Actions
```

---

# Part 4 – Cleanup

To avoid AWS charges:

1. Stop EC2 instances
2. Delete unused security groups
3. Delete test repositories

---

# Troubleshooting

---

# Jenkins Not Accessible

Check:

```bash
sudo systemctl status jenkins
```

Verify:

- Port 8080 open
- Jenkins running

---

# SSH Permission Denied

Fix permissions:

```bash
chmod 400 mykey.pem
```

---

# Apache Not Running

Restart service:

```bash
sudo systemctl restart httpd
```

---

# GitHub Actions Failure

Verify:

- Secrets configured correctly
- EC2 IP correct
- Security group allows SSH
- SSH key valid

---

# Best Practices

- Use secure secrets
- Restrict SSH access
- Use CI/CD approvals
- Monitor deployments
- Use staging environments

---

# Summary

In this lab you completed:

- Jenkins setup on AWS EC2
- Jenkins pipeline deployment
- GitHub Actions deployment workflow
- Automated application deployment to AWS
- Real-world CI/CD integration
