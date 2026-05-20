# Day 7 – DevOps Theory

## Module 1 – DevOps & CI/CD Fundamentals

### Date: 26-May-2026 (Tuesday)

---

# Topic: Third-Party CI/CD Tools

## Topics Covered

- Jenkins
- GitHub Actions
- GitLab CI integration
- CI/CD integration with AWS EC2

---

# 1. Introduction to CI/CD

CI/CD stands for:

- **Continuous Integration**
- **Continuous Delivery**
- **Continuous Deployment**

CI/CD is a DevOps practice used to automate software build, test, and deployment processes.

---

## Continuous Integration

Continuous Integration means developers frequently merge code changes into a shared repository.

Every code change can automatically trigger:

- Code checkout
- Dependency installation
- Build process
- Unit testing
- Static code analysis
- Artifact creation

### Benefits of Continuous Integration

- Early bug detection
- Faster development feedback
- Reduced integration issues
- Improved code quality
- Better team collaboration

---

## Continuous Delivery

Continuous Delivery means the application is always kept ready for deployment.

The deployment may still require manual approval before releasing to production.

### Example

Code is automatically built and tested, but a DevOps engineer manually approves deployment to production.

---

## Continuous Deployment

Continuous Deployment means every successful code change is automatically deployed to production without manual approval.

### Example

A developer pushes code to GitHub. If all tests pass, the application is automatically deployed to an AWS EC2 instance.

---

# 2. Jenkins

## What is Jenkins?

Jenkins is an open-source automation server widely used for CI/CD pipelines.

It helps automate:

- Building applications
- Running tests
- Creating artifacts
- Deploying applications
- Integrating with cloud platforms
- Running scheduled jobs

---

## Why Jenkins is Used

Jenkins is popular because it is:

- Open source
- Extensible using plugins
- Supports many programming languages
- Supports distributed builds
- Integrates with GitHub, GitLab, Docker, AWS, Kubernetes, Maven, Gradle, and many other tools

---

## Jenkins Architecture

Jenkins architecture usually contains:

- Jenkins Controller
- Jenkins Agents
- Jobs
- Pipelines
- Plugins
- Credentials

---

## Jenkins Controller

The Jenkins Controller is the main Jenkins server.

It manages:

- User interface
- Job configuration
- Plugin management
- Build scheduling
- Pipeline orchestration

---

## Jenkins Agent

A Jenkins Agent is a worker machine that executes build or deployment tasks.

Agents help distribute workload across multiple machines.

### Example

The Jenkins Controller runs on one EC2 instance, while agents run on separate EC2 instances to execute jobs.

---

## Jenkins Job

A Jenkins job is a task configured in Jenkins.

Types of jobs include:

- Freestyle project
- Pipeline job
- Multibranch pipeline
- Maven project

---

## Jenkins Pipeline

A Jenkins pipeline is a set of automated steps written as code.

Pipeline files are usually written in a file called:

```text
Jenkinsfile
```

---

## Jenkins Pipeline Stages

A common Jenkins pipeline includes:

```text
Checkout → Build → Test → Package → Deploy
```

Example stages:

- Checkout source code from Git
- Install dependencies
- Run tests
- Build application
- Deploy application to AWS EC2

---

## Jenkinsfile Example

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/example/repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'echo Building application'
            }
        }

        stage('Test') {
            steps {
                sh 'echo Running tests'
            }
        }

        stage('Deploy') {
            steps {
                sh 'echo Deploying application'
            }
        }
    }
}
```

---

## Jenkins Plugins

Plugins extend Jenkins functionality.

Common plugins:

| Plugin | Purpose |
|---|---|
| Git Plugin | Integrates Jenkins with Git repositories |
| GitHub Plugin | Connects Jenkins with GitHub |
| Pipeline Plugin | Enables Jenkins pipeline as code |
| SSH Agent Plugin | Allows SSH-based deployment |
| AWS Credentials Plugin | Stores AWS credentials securely |
| Docker Plugin | Integrates Jenkins with Docker |

---

## Jenkins Credentials

Jenkins Credentials are used to securely store sensitive information such as:

- SSH keys
- Passwords
- AWS access keys
- GitHub tokens
- Docker registry credentials

Credentials should never be hardcoded inside pipeline scripts.

---

## Jenkins Integration with AWS

Jenkins can integrate with AWS services such as:

- EC2
- S3
- CodeDeploy
- ECR
- ECS
- CloudFormation
- IAM

### Example Use Case

Jenkins builds application code and deploys it to an EC2 instance using SSH.

---

# 3. GitHub Actions

## What is GitHub Actions?

GitHub Actions is a CI/CD platform built directly into GitHub.

It allows users to automate workflows directly from a GitHub repository.

GitHub Actions can automate:

- Code build
- Unit testing
- Security scanning
- Docker image creation
- Deployment to AWS
- Notifications

---

## GitHub Actions Workflow

A GitHub Actions workflow is defined using YAML files.

Workflow files are stored inside:

```text
.github/workflows/
```

Example file:

```text
.github/workflows/deploy.yml
```

---

## Key GitHub Actions Concepts

| Concept | Description |
|---|---|
| Workflow | Complete automation process |
| Event | Trigger that starts workflow |
| Job | Group of steps executed on a runner |
| Step | Individual command or action |
| Runner | Machine that executes the workflow |
| Action | Reusable automation component |

---

## GitHub Actions Events

A workflow can be triggered by events such as:

- push
- pull_request
- workflow_dispatch
- schedule

### Example

```yaml
on:
  push:
    branches:
      - main
```

This means the workflow runs whenever code is pushed to the main branch.

---

## GitHub Actions Runner

A runner is a machine that executes GitHub Actions jobs.

Types of runners:

- GitHub-hosted runner
- Self-hosted runner

### GitHub-hosted Runner

Managed by GitHub.

Example:

```yaml
runs-on: ubuntu-latest
```

### Self-hosted Runner

Installed and managed by the user.

Example use case:

- Running workflows on an AWS EC2 instance inside a private network

---

## GitHub Actions Workflow Example

```yaml
name: Simple CI Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Build Application
        run: echo "Building application"

      - name: Run Tests
        run: echo "Running tests"
```

---

## GitHub Actions Integration with AWS

GitHub Actions can deploy to AWS using:

- AWS access keys
- IAM roles with OIDC
- AWS CLI
- AWS CodeDeploy
- AWS ECS
- AWS S3

### Recommended Method

Use OpenID Connect, also called OIDC, instead of long-term AWS access keys.

OIDC allows GitHub Actions to assume an AWS IAM role securely.

---

## Advantages of GitHub Actions

- Integrated with GitHub
- Easy YAML-based workflow
- Large marketplace of reusable actions
- Good for open-source and cloud-native projects
- Supports many deployment targets

---

# 4. GitLab CI

## What is GitLab CI?

GitLab CI/CD is a built-in CI/CD platform provided by GitLab.

It allows teams to automate build, test, and deployment processes directly from GitLab repositories.

---

## GitLab CI Pipeline File

GitLab CI uses a YAML file named:

```text
.gitlab-ci.yml
```

This file is placed in the root of the repository.

---

## GitLab CI Concepts

| Concept | Description |
|---|---|
| Pipeline | Complete CI/CD process |
| Stage | Logical phase such as build, test, deploy |
| Job | Task executed inside a stage |
| Runner | Machine that executes jobs |
| Artifact | Output generated by a job |
| Variable | Configuration or secret value |

---

## GitLab CI Stages

Common stages:

```text
build → test → deploy
```

---

## GitLab CI Example

```yaml
stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  script:
    - echo "Building application"

test-job:
  stage: test
  script:
    - echo "Running tests"

deploy-job:
  stage: deploy
  script:
    - echo "Deploying application"
```

---

## GitLab Runner

GitLab Runner is the agent that executes GitLab CI jobs.

Runner types:

- Shared runner
- Specific runner
- Group runner
- Self-managed runner

A self-managed GitLab Runner can be installed on AWS EC2.

---

## GitLab CI Integration with AWS

GitLab CI can deploy applications to AWS using:

- AWS CLI
- SSH
- CodeDeploy
- ECS
- EKS
- S3
- CloudFormation

### Example Use Case

GitLab CI builds an application and deploys it to an EC2 instance using SSH.

---

# 5. Jenkins vs GitHub Actions vs GitLab CI

| Feature | Jenkins | GitHub Actions | GitLab CI |
|---|---|---|---|
| Hosting | Self-managed | GitHub-managed or self-hosted | GitLab-managed or self-hosted |
| Configuration | Jenkinsfile | YAML workflow | .gitlab-ci.yml |
| Plugin ecosystem | Very large | Marketplace actions | Built-in integrations |
| Setup effort | Medium to high | Low | Low to medium |
| Best for | Enterprise/custom pipelines | GitHub projects | GitLab projects |
| AWS integration | Strong | Strong | Strong |

---

# 6. Best Practices

## Jenkins Best Practices

- Use pipelines as code
- Store Jenkinsfile in source control
- Use credentials securely
- Use agents for build execution
- Keep Jenkins plugins updated
- Avoid running builds on controller when possible

---

## GitHub Actions Best Practices

- Use secrets for sensitive data
- Use OIDC for AWS authentication
- Keep workflows modular
- Use branch protection rules
- Use manual approval for production
- Pin action versions where possible

---

## GitLab CI Best Practices

- Store pipeline configuration in `.gitlab-ci.yml`
- Use CI/CD variables for secrets
- Use separate environments for dev, staging, and production
- Use artifacts for build outputs
- Use manual approval for production deployments

---

# 7. Summary

In this theory session, you learned:

- What CI/CD means
- Difference between Continuous Integration, Delivery, and Deployment
- Jenkins architecture and pipelines
- GitHub Actions workflows
- GitLab CI pipelines
- How third-party CI/CD tools integrate with AWS
- Best practices for secure and reliable CI/CD
