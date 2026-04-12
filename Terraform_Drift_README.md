# Terraform Drift - POC

---

## Introduction

This project demonstrates Terraform Drift Detection using a practical Proof of Concept (POC).

Terraform is an Infrastructure as Code (IaC) tool that allows you to define and manage infrastructure in a declarative way. However, when changes are made manually outside Terraform (e.g., via AWS Console), the actual infrastructure may deviate from the defined configuration. This is called Terraform Drift.

---

## Objective

The goal of this POC is to:

- Create infrastructure using Terraform
- Introduce manual changes (drift)
- Detect drift using Terraform
- Optionally fix the drift

---

## What is Terraform Drift?

Terraform Drift occurs when:

The **actual infrastructure state** ≠ **Terraform configuration (desired state)**

### Example:
- Terraform defines EC2 instance as `t2.micro`
- Someone manually changes it to `t2.large` in AWS Console

This mismatch is called Drift

---

## Architecture Overview

- AWS EC2 Instance created via Terraform
- Manual modification via AWS Console
- Drift detection using `terraform plan`

---

## Prerequisites

Before starting, ensure you have:

- AWS Account
- IAM user with programmatic access
- AWS CLI configured (`aws configure`)
- Terraform installed (v1.x recommended)

---

## Project Structure
```
terraform-drift-poc/
│
├── main.tf # Main Terraform configuration
├── variables.tf # Input variables (optional)
├── outputs.tf # Outputs (optional)
├── terraform.tfstate # State file (auto-generated)
└── README.md # Documentation
```

---

## Step-by-Step Implementation

### Step 1: Initialize Terraform

```bash
terraform init
```

### Step 2: Create Infrastructure

```bash
terraform apply
```
Type `yes` when prompted.

**or**
```bash
terraform apply -auto-approve
```
### Step 3: Verify Resource

Go to AWS Console → EC2 → Instances
Confirm the instance is created.

### Step 4: Introduce Drift (Manual Change)

Now simulate drift:

- Go to AWS Console
- Select the EC2 instance
- Change instance type (e.g., t2.micro → t2.large)
- Save changes
  
### Step 5: Detect Drift
```bash
terraform plan
```
**Expected Output:**

### Step 6: Fix Drift (Optional)
To revert the infrastructure to the desired state:
```bash
terraform apply -auto-approve
```
Terraform will:

- Detect mismatch
- Reconfigure the resource to match the code

---
## Key Terraform Concepts

| Concept        | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| Desired State  | Defined in `.tf` files. Represents how your infrastructure *should* look.   |
| Current State  | Stored in `terraform.tfstate`. Represents actual deployed infrastructure.   |
| Drift          | Difference between the desired state and the actual infrastructure.         |

---

## State & Lock File Explanation

| File Name              | Description                                                                |
|-----------------------|-----------------------------------------------------------------------------|
| terraform.tfstate     | Stores real-time infrastructure data                                        |
|                       | Used by Terraform to compare desired vs actual state during `plan`          |
|                       | Critical for drift detection                                                |
|                       | Do NOT edit manually                                                        |
| .terraform.lock.hcl   | Ensures provider version consistency                                        |
|                       | Prevents unexpected changes due to version mismatches                       |

---

## Advanced Concepts

| Topic                          | Description                                                                |
|--------------------------------|----------------------------------------------------------------------------|
| Remote State                   | Store Terraform state remotely instead of locally                          |
|                                | Use AWS S3 for state storage                                               |
|                                | Use DynamoDB for state locking                                             |
| Drift Detection in CI/CD       | Automate drift detection using CI/CD pipelines                             |
|                                | Tools: GitHub Actions, Jenkins, GitLab CI                                  |
|                                | Example: `terraform plan -detailed-exitcode`                               |
| Terraform Cloud Drift Detection| Provides automated drift detection                                         |
|                                | Sends notifications                                                        |
|                                | Supports scheduled checks                                                  |
| Alternative Tools              | AWS Config, Spacelift, Atlantis                                            |

---
## Best Practices

- Avoid manual changes in the cloud
- Always use Terraform for updates
- Enable remote state + locking
- Use CI/CD for automation
- Regularly run terraform plan

---

## References

| Topic                                  | Link                                                                 |
|----------------------------------------|----------------------------------------------------------------------|
| Terraform Official Documentation       | https://developer.hashicorp.com/terraform/docs                       |
| Terraform State File                   | https://developer.hashicorp.com/terraform/language/state             |
| Terraform Drift Detection (Concept)    | https://developer.hashicorp.com/terraform/tutorials/state/resource-drift |
| Remote State with S3 & DynamoDB        | https://developer.hashicorp.com/terraform/language/settings/backends/s3 |
| Terraform CLI Commands (`plan`, `apply`)| https://developer.hashicorp.com/terraform/cli                        |
| Terraform Cloud Drift Detection        | https://developer.hashicorp.com/terraform/cloud-docs/workspaces/drift-detection |
| AWS Config Documentation               | https://docs.aws.amazon.com/config/                                  |
| Spacelift Drift Detection              | https://docs.spacelift.io/concepts/drift-detection                   |
| Atlantis (Terraform Automation Tool)   | https://github.com/runatlantis/atlantis                              |
