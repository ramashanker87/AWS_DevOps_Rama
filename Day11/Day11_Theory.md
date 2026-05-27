# Day 11 — Terraform Fundamentals and Architecture
## Module 2 – Infrastructure as Code (IaC)
### Date: 20-May-2026 (Monday)

---

# Learning Objectives

By the end of this session, participants will be able to:

- Understand Terraform architecture
- Install and configure Terraform CLI
- Understand Infrastructure as Code concepts
- Write Terraform configuration files
- Manage Terraform state
- Provision AWS infrastructure using Terraform
- Work with providers, resources, variables, and outputs

---

# Introduction to Terraform

Terraform is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp.

Terraform allows users to:
- Define infrastructure using code
- Automate infrastructure provisioning
- Manage cloud resources consistently

Terraform supports:
- AWS
- Azure
- Google Cloud
- Kubernetes
- VMware
- Many other providers

---

# Benefits of Terraform

## Infrastructure Automation
Automates cloud resource provisioning.

## Multi-Cloud Support
Works across multiple cloud providers.

## Declarative Language
Uses HashiCorp Configuration Language (HCL).

## Version Control
Infrastructure code can be stored in Git.

## Reusability
Modules and reusable templates simplify deployments.

---

# Terraform Architecture

Terraform consists of:

- Terraform Core
- Providers
- State Files
- Configuration Files

---

# Terraform Workflow

1. Write configuration
2. Initialize Terraform
3. Validate configuration
4. Generate execution plan
5. Apply infrastructure
6. Destroy infrastructure (if needed)

---

# Terraform Configuration Files

Terraform files use:
```hcl
.tf
```

Example:
```hcl
provider "aws" {
  region = "us-east-1"
}
```

---

# Terraform Providers

Providers allow Terraform to interact with cloud platforms.

Example:
```hcl
provider "aws" {
  region = "us-east-1"
}
```

---

# Terraform Resources

Resources define infrastructure components.

Example:
```hcl
resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```

---

# Terraform Variables

Variables improve flexibility.

Example:
```hcl
variable "instance_type" {
  default = "t2.micro"
}
```

---

# Terraform Outputs

Outputs display important values after deployment.

Example:
```hcl
output "instance_id" {
  value = aws_instance.web.id
}
```

---

# Terraform State

Terraform stores infrastructure metadata in:
```text
terraform.tfstate
```

Purpose:
- Track resources
- Manage updates
- Detect changes

---

# Terraform Commands

| Command | Purpose |
|---|---|
| terraform init | Initialize project |
| terraform validate | Validate syntax |
| terraform plan | Preview changes |
| terraform apply | Deploy infrastructure |
| terraform destroy | Remove infrastructure |

---

# Terraform Modules

Modules allow reusable infrastructure components.

Benefits:
- Standardization
- Reusability
- Scalability

---

# Terraform Best Practices

- Use remote state storage
- Use version control
- Separate environments
- Use modules
- Avoid hardcoded values
- Protect sensitive variables

---

# AWS Resources Used

- EC2
- VPC
- IAM
- Terraform CLI

---

# Summary

In this session, we covered:
- Terraform fundamentals
- Terraform architecture
- Terraform workflow
- Providers and resources
- State management
- Infrastructure provisioning concepts

---
