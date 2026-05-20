# Day 7 – DevOps Lab

## Module 1 – DevOps & CI/CD Fundamentals

### Date: 26-May-2026 (Tuesday)

---

# Lab: Third-Party CI/CD Integration with AWS EC2

## Objective

In this lab, you will learn how to integrate third-party CI/CD tools with AWS EC2.

You will complete three labs:

1. Jenkins integration with AWS EC2
2. GitHub Actions deployment to AWS EC2
3. GitLab CI deployment to AWS EC2

---

# AWS Resources Used

- EC2
- Security Groups
- Key Pair
- Jenkins
- GitHub Actions
- GitLab CI
- SSH

---

# Common Prerequisites

Before starting this lab, ensure you have:

- AWS account
- EC2 key pair
- Basic Linux knowledge
- GitHub account
- GitLab account
- Jenkins server or Jenkins installed on EC2
- Sample application repository

---

# Part 1 – Prepare AWS EC2 Deployment Server

This EC2 instance will host the sample application.

---

## Step 1 – Launch EC2 Instance

1. Open AWS Console
2. Go to EC2
3. Click **Launch Instance**
4. Configure:

| Setting | Value |
|---|---|
| Name | devops-app-server |
| AMI | Amazon Linux 2 |
| Instance Type | t2.micro |
| Key Pair | Select your key pair |
| Security Group | Allow SSH and HTTP |

---

## Step 2 – Configure Security Group

Allow inbound traffic:

| Type | Port | Source |
|---|---|---|
| SSH | 22 | Your IP |
| HTTP | 80 | Anywhere |

---

## Step 3 – Connect to EC2

```bash
ssh -i mykey.pem ec2-user@<EC2-PUBLIC-IP>
```

---

## Step 4 – Install Apache Web Server

```bash
sudo yum update -y
sudo yum install httpd git -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

---

## Step 5 – Create Test Page

```bash
echo "<h1>Initial Application Running on AWS EC2</h1>" | sudo tee /var/www/html/index.html
```

---

## Step 6 – Verify Application

Open browser:

```text
http://<EC2-PUBLIC-IP>
```

You should see:

```text
Initial Application Running on AWS EC2
```

---

# Part 2 – Jenkins Integration Lab

## Objective

Configure Jenkins to deploy a sample application to AWS EC2.

---

## Step 1 – Launch Jenkins EC2 Instance

Create another EC2 instance for Jenkins.

| Setting | Value |
|---|---|
| Name | jenkins-server |
| AMI | Amazon Linux 2 |
| Instance Type | t2.micro |
| Ports | 22, 8080 |

Security Group inbound rules:

| Type | Port | Source |
|---|---|---|
| SSH | 22 | Your IP |
| Custom TCP | 8080 | Your IP |

---

## Step 2 – Connect to Jenkins Server

```bash
ssh -i mykey.pem ec2-user@<JENKINS-EC2-PUBLIC-IP>
```

---

## Step 3 – Install Java

```bash
sudo yum update -y
sudo amazon-linux-extras install java-openjdk11 -y
```

---

## Step 4 – Install Jenkins

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

sudo yum install jenkins -y
```

---

## Step 5 – Start Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```

---

## Step 6 – Access Jenkins

Open browser:

```text
http://<JENKINS-EC2-PUBLIC-IP>:8080
```

---

## Step 7 – Get Jenkins Initial Admin Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy the password and paste it in Jenkins UI.

---

## Step 8 – Install Suggested Plugins

1. Select **Install suggested plugins**
2. Create admin user
3. Complete setup

---

## Step 9 – Install Required Jenkins Plugins

Go to:

```text
Manage Jenkins → Plugins
```

Install:

- Git plugin
- Pipeline plugin
- SSH Agent plugin

---

## Step 10 – Create Sample Git Repository

Create a GitHub repository with this file:

### index.html

```html
<h1>Application deployed using Jenkins CI/CD</h1>
```

---

## Step 11 – Add EC2 Private Key to Jenkins Credentials

1. Open Jenkins
2. Go to **Manage Jenkins**
3. Open **Credentials**
4. Add new credential
5. Select:
   - Kind: SSH Username with private key
   - Username: ec2-user
   - Private Key: paste contents of `mykey.pem`
6. Set ID:

```text
ec2-ssh-key
```

---

## Step 12 – Create Jenkins Pipeline Job

1. Click **New Item**
2. Enter name:

```text
jenkins-ec2-deploy
```

3. Select **Pipeline**
4. Click **OK**

---

## Step 13 – Add Jenkins Pipeline Script

Use this pipeline:

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/<USERNAME>/<REPO-NAME>.git'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                    scp -o StrictHostKeyChecking=no index.html ec2-user@<APP-EC2-PUBLIC-IP>:/tmp/index.html
                    ssh -o StrictHostKeyChecking=no ec2-user@<APP-EC2-PUBLIC-IP> "sudo cp /tmp/index.html /var/www/html/index.html && sudo systemctl restart httpd"
                    '''
                }
            }
        }
    }
}
```

Replace:

```text
<USERNAME>
<REPO-NAME>
<APP-EC2-PUBLIC-IP>
```

---

## Step 14 – Run Jenkins Pipeline

1. Click **Build Now**
2. Open Console Output
3. Confirm deployment success

---

## Step 15 – Validate Jenkins Deployment

Open browser:

```text
http://<APP-EC2-PUBLIC-IP>
```

Expected output:

```text
Application deployed using Jenkins CI/CD
```

---

# Part 3 – GitHub Actions Integration Lab

## Objective

Deploy application changes from GitHub to AWS EC2 using GitHub Actions.

---

## Step 1 – Create GitHub Repository

Create a repository, for example:

```text
github-actions-ec2-deploy
```

---

## Step 2 – Add index.html

Create file:

```html
<h1>Application deployed using GitHub Actions</h1>
```

---

## Step 3 – Add GitHub Secrets

Go to:

```text
GitHub Repository → Settings → Secrets and variables → Actions → New repository secret
```

Create these secrets:

| Secret Name | Value |
|---|---|
| EC2_HOST | EC2 public IP |
| EC2_USER | ec2-user |
| EC2_SSH_KEY | Private key content from mykey.pem |

---

## Step 4 – Create Workflow Directory

In repository, create:

```text
.github/workflows/
```

---

## Step 5 – Create Workflow File

Create:

```text
.github/workflows/deploy.yml
```

Add:

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

      - name: Copy Application to EC2
        run: |
          scp -i ~/.ssh/ec2_key index.html ${{ secrets.EC2_USER }}@${{ secrets.EC2_HOST }}:/tmp/index.html

      - name: Deploy Application
        run: |
          ssh -i ~/.ssh/ec2_key ${{ secrets.EC2_USER }}@${{ secrets.EC2_HOST }} "sudo cp /tmp/index.html /var/www/html/index.html && sudo systemctl restart httpd"
```

---

## Step 6 – Commit and Push Code

```bash
git add .
git commit -m "Add GitHub Actions EC2 deployment"
git push origin main
```

---

## Step 7 – Monitor GitHub Actions

1. Open GitHub repository
2. Click **Actions**
3. Select workflow run
4. Verify all steps are successful

---

## Step 8 – Validate GitHub Actions Deployment

Open browser:

```text
http://<EC2-PUBLIC-IP>
```

Expected output:

```text
Application deployed using GitHub Actions
```

---

# Part 4 – GitLab CI Integration Lab

## Objective

Deploy application changes from GitLab to AWS EC2 using GitLab CI.

---

## Step 1 – Create GitLab Repository

Create a GitLab project, for example:

```text
gitlab-ci-ec2-deploy
```

---

## Step 2 – Add index.html

Create:

```html
<h1>Application deployed using GitLab CI</h1>
```

---

## Step 3 – Add GitLab CI/CD Variables

Go to:

```text
GitLab Project → Settings → CI/CD → Variables
```

Add variables:

| Variable | Value |
|---|---|
| EC2_HOST | EC2 public IP |
| EC2_USER | ec2-user |
| EC2_SSH_KEY | Private key content from mykey.pem |

Mark variables as:

- Masked
- Protected if using protected branches

---

## Step 4 – Create .gitlab-ci.yml

Create this file in repository root:

```yaml
stages:
  - deploy

deploy_to_ec2:
  stage: deploy
  image: alpine:latest

  before_script:
    - apk add --no-cache openssh-client
    - mkdir -p ~/.ssh
    - echo "$EC2_SSH_KEY" > ~/.ssh/ec2_key
    - chmod 600 ~/.ssh/ec2_key
    - ssh-keyscan -H "$EC2_HOST" >> ~/.ssh/known_hosts

  script:
    - scp -i ~/.ssh/ec2_key index.html "$EC2_USER@$EC2_HOST:/tmp/index.html"
    - ssh -i ~/.ssh/ec2_key "$EC2_USER@$EC2_HOST" "sudo cp /tmp/index.html /var/www/html/index.html && sudo systemctl restart httpd"

  only:
    - main
```

---

## Step 5 – Commit and Push

```bash
git add .
git commit -m "Add GitLab CI EC2 deployment"
git push origin main
```

---

## Step 6 – Monitor GitLab Pipeline

1. Open GitLab project
2. Go to **Build → Pipelines**
3. Open latest pipeline
4. Verify deploy job is successful

---

## Step 7 – Validate GitLab Deployment

Open browser:

```text
http://<EC2-PUBLIC-IP>
```

Expected output:

```text
Application deployed using GitLab CI
```

---

# Part 5 – Cleanup

To avoid AWS charges:

1. Stop or terminate EC2 instances
2. Delete unused security groups
3. Delete unused key pairs if no longer needed
4. Remove test repositories if no longer required

---

# Troubleshooting

## Jenkins Not Opening

Check Jenkins status:

```bash
sudo systemctl status jenkins
```

Check port 8080 security group rule.

---

## EC2 SSH Permission Denied

Fix key permissions:

```bash
chmod 400 mykey.pem
```

Use correct username:

```text
ec2-user
```

---

## Apache Page Not Updating

Restart Apache:

```bash
sudo systemctl restart httpd
```

Check file:

```bash
cat /var/www/html/index.html
```

---

## GitHub Actions or GitLab CI SSH Failure

Check:

- EC2 public IP is correct
- Private key is pasted correctly
- Security group allows SSH
- Username is `ec2-user`
- EC2 instance is running

---

# Best Practices

- Store secrets securely
- Do not hardcode private keys
- Use IAM roles where possible
- Restrict SSH access to trusted IPs
- Use separate environments for dev, test, and production
- Use manual approval before production deployment
- Review pipeline logs after every deployment

---

# Summary

In this lab, you completed:

- Jenkins deployment to AWS EC2
- GitHub Actions deployment to AWS EC2
- GitLab CI deployment to AWS EC2
- Third-party CI/CD integration with AWS
- Basic automated deployment workflow
