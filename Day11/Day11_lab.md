# Day 11 Lab Guide — Terraform on AWS
## Module 2 – Infrastructure as Code (IaC)
### Date: 20-May-2026 (Monday)

---

# Lab Objectives

Participants will learn how to:

- Install Terraform CLI
- Configure AWS credentials
- Create Terraform configurations
- Provision EC2 instances
- Create VPC infrastructure
- Manage Terraform state
- Destroy infrastructure safely

---

# Prerequisites

Ensure the following are available:

- AWS Account
- IAM User with permissions
- AWS CLI installed
- Terraform CLI installed
- VS Code or text editor

---

# AWS Resources Used

| Resource | Purpose |
|---|---|
| EC2 | Virtual servers |
| VPC | Networking |
| IAM | Permissions |
| Terraform CLI | IaC provisioning |

---

# Lab 1 — Install Terraform

sudo apt update && sudo apt install -y gnupg software-properties-common curl

curl -fsSL https://apt.releases.hashicorp.com/gpg | \
sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt install terraform
## Step 1 — Verify Installation

Run:

```bash
terraform version
```

Expected output:
```text
Terraform v1.x.x
```

---

# Lab 2 — Configure AWS Credentials

## Step 1 — Configure AWS CLI

```bash
aws configure
```

Provide:
- AWS Access Key
- AWS Secret Key
- Region
- Output format

---

# Lab 3 — Create Terraform Project

## Step 1 — Create Project Directory

```bash
mkdir terraform-lab
cd terraform-lab
```

---

# Step 2 — Create Provider Configuration

Create file:
```text
provider.tf
```

Add:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

---

# Step 3 — Create EC2 Instance Configuration

Create file:
```text
main.tf
```

```hcl
resource "aws_instance" "webserver" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "Terraform-WebServer"
  }
}
```

---

# Step 4 — Initialize Terraform

```bash
terraform init
```

---

# Step 5 — Validate Configuration

```bash
terraform validate
```

---

# Step 6 — Generate Execution Plan

```bash
terraform plan
```

---

# Step 7 — Apply Infrastructure

```bash
terraform apply
```

Type:
```text
yes
```

---

# Step 8 — Verify EC2 Instance

Verify in AWS Console:
- EC2 Dashboard
- Running instances

---

# Lab 4 — Create VPC Using Terraform

## Step 1 — Add VPC Resource

Append to:
```text
main.tf
```

```hcl
resource "aws_vpc" "mainvpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "Terraform-VPC"
  }
}
```

---

# Step 2 — Apply Changes

```bash
terraform apply
```

---

# Step 3 — Verify VPC

Verify in AWS Console:
- VPC Dashboard

---

# Lab 5 — Use Variables

## Step 1 — Create Variables File

Create:
```text
variables.tf
```

```hcl
variable "instance_type" {
  default = "t2.micro"
}
```

---

# Step 2 — Update EC2 Resource

```hcl
instance_type = var.instance_type
```

---

# Step 3 — Reapply Configuration

```bash
terraform apply
```

---

# Lab 6 — Terraform Outputs

## Step 1 — Create Outputs File

Create:
```text
outputs.tf
```

```hcl
output "instance_id" {
  value = aws_instance.webserver.id
}
```

---

# Step 2 — Apply Configuration

```bash
terraform apply
```

---

# Step 3 — View Outputs

```bash
terraform output
```

---

# Lab 7 — Destroy Infrastructure

## Step 1 — Remove Resources

```bash
terraform destroy
```

Type:
```text
yes
```

---

# Troubleshooting Guide

| Issue | Solution |
|---|---|
| Terraform not found | Verify PATH configuration |
| Invalid AWS credentials | Reconfigure AWS CLI |
| Provider download failed | Check internet access |
| EC2 creation failed | Verify AMI ID and permissions |
| State lock issue | Remove stale lock |

---

# Best Practices

- Use separate environments
- Store state remotely
- Use IAM least privilege
- Use variables for flexibility
- Use modules for reuse

---

# Challenge Exercises

## Challenge 1
Create multiple EC2 instances.

## Challenge 2
Add Security Groups.

## Challenge 3
Create subnets inside VPC.

## Challenge 4
Store Terraform state in S3.

---

# Expected Outcomes

After completing the labs, participants will:
- Understand Terraform basics
- Provision AWS infrastructure
- Manage Terraform workflows
- Use Terraform variables and outputs
- Create reusable IaC deployments

---

# End of Lab

Congratulations!
You have completed Day 11 Terraform labs.

---
