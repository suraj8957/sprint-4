# Documentation: Terraform CI/CD Shared Library 
---
## Document Details

| Author | Created on | Version | Last updated by | Last edited on | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer |
|--------|------------|---------|-----------------|----------------|--------------|-------------|-------------|-------------|
| Suraj Tripathi | 13-04-2026 | v1.0 | Suraj Tripathi | 13-04-2026 |              | Aniruddh    | Shreya S    | Ashwani     |

---
## Table of Contents
- [Overview](#1-overview)
- [Prerequisites](#2-prerequisites)
- [Architecture](#3-architecture)

- [CI Shared Library](#4-ci-shared-library)
  - [CI Purpose](#41-ci-purpose)
  - [CI Supported Functions](#42-ci-supported-functions)
  - [CI Inputs](#43-ci-inputs)
  - [CI Outputs](#44-ci-outputs)
  - [CI Jenkins Usage Example](#45-ci-jenkins-usage-example)

- [CD Shared Library](#5-cd-shared-library)
  - [CD Purpose](#51-cd-purpose)
  - [CD Supported Functions](#52-cd-supported-functions)
  - [CD Inputs](#53-cd-inputs)
  - [CD Outputs](#54-cd-outputs)
  - [CD Jenkins Usage Example](#55-cd-jenkins-usage-example)

- [Advanced Usage](#6-advanced-usage)
  - [Environment-Based Deployment](#61-environment-based-deployment)
  - [Parallel Execution](#62-parallel-execution)
  - [Plan Approval Flow](#63-plan-approval-flow)

- [Troubleshooting](#7-troubleshooting)
- [Contact Information](#8-contact-information)
- [References](#9-references)
---

This repository provides Jenkins Shared Libraries for automating Terraform workflows across:

- CI (Validation & Planning)
- CD (Deployment & Destruction)

It standardizes infrastructure automation, improves reusability, and enforces best practices.

---

## 1. Overview
This shared library simplifies Terraform automation by splitting workflows into:
| Stage  | Purpose                                           |
| ------ | ------------------------------------------------- |
| CI     | Validate Terraform code & generate execution plan |
| CD     | Apply or destroy infrastructure                   |

---
## 2. Prerequisites
| Category           | Requirement               | Description                                            |
| ------------------ | ------------------------- | ------------------------------------------------------ |
| Tools Installed    | Terraform (>= 1.x)        | Required to manage infrastructure as code              |
| Tools Installed    | Jenkins                   | CI/CD automation server                                |
| Tools Installed    | Git                       | Version control system                                 |
| Cloud Setup        | AWS CLI                   | Must be configured for Terraform to interact with AWS  |
| Cloud Setup        | IAM Permissions           | Proper roles & policies required for resource creation |
| Jenkins Setup      | Shared Library Configured | Library must be added in Jenkins Global Configuration  |
| Jenkins Setup      | Credentials Stored        | Store AWS keys or secrets securely in Jenkins          |

---

## 3. Architecture
```
Developer → Git Push → Jenkins CI Pipeline → Terraform Validate + Plan
                                           ↓
                                      Approval
                                           ↓
                                 Jenkins CD Pipeline
                                           ↓
                              Terraform Apply / Destroy
```

---

## 4. CI Shared Library

### 4.1 CI Purpose
The CI library is responsible for:

- Initializing Terraform
- Validating syntax
- Generating execution plan
- Ensuring code quality before deployment

### 4.2 CI Supported Functions
| Function     | Description                             |
| ------------ | --------------------------------------- |
| `init()`     | Initializes Terraform working directory |
| `validate()` | Validates Terraform configuration       |
| `plan()`     | Generates execution plan                |

### 4.3 CI Inputs
| Parameter       | Description                      | Required |
| --------------- | -------------------------------- | -------- |
| `workingDir`    | Terraform directory path         | ✅        |
| `backendConfig` | Backend configuration (S3, etc.) | ❌        |
| `varFile`       | Terraform variables file         | ❌        |

### 4.4 CI Outputs
| Output    | Description            |
| --------- | ---------------------- |
| Plan File | Binary plan output     |
| Logs      | Validation & plan logs |
| Status    | Success / Failure      |


### 4.5 CI Jenkins Usage Example
```
@Library('terraform-shared-library') _

node {
    stage('Terraform Init') {
        terraformInit(
            workingDir: 'infra/',
            backendConfig: 'backend.tfvars'
        )
    }

    stage('Terraform Validate') {
        terraformValidate(
            workingDir: 'infra/'
        )
    }

    stage('Terraform Plan') {
        terraformPlan(
            workingDir: 'infra/',
            varFile: 'terraform.tfvars'
        )
    }
}
```
## 5. CD Shared Library
### 5.1 CD Purpose
The CD library handles:

- Infrastructure provisioning
- Infrastructure destruction
- Safe and controlled deployments

### 5.2 CD Supported Functions
| Function    | Description             |
| ----------- | ----------------------- |
| `apply()`   | Applies Terraform plan  |
| `destroy()` | Destroys infrastructure |

### 5.3 CD Inputs
| Parameter     | Description          | Required |
| ------------- | -------------------- | -------- |
| `workingDir`  | Terraform directory  | ✅        |
| `varFile`     | Variables file       | ❌        |
| `autoApprove` | Skip manual approval | ❌        |

### 5.4 CD Outputs
| Output            | Description               |
| ----------------- | ------------------------- |
| Deployment Status | Success / Failure         |
| Logs              | Apply/Destroy logs        |
| Resources         | Created/Deleted resources |

### 5.5 CD Jenkins Usage Example
```
@Library('terraform-shared-library') _

node {
    stage('Terraform Apply') {
        terraformApply(
            workingDir: 'infra/',
            varFile: 'terraform.tfvars',
            autoApprove: true
        )
    }

    stage('Terraform Destroy') {
        input message: "Are you sure you want to destroy infrastructure?"

        terraformDestroy(
            workingDir: 'infra/',
            varFile: 'terraform.tfvars'
        )
    }
}
```
---
## 6. Advanced Usage
### 6.1 Environment-Based Deployment
```
def env = "dev"

terraformApply(
    workingDir: "infra/${env}",
    varFile: "${env}.tfvars"
)
```
### 6.2 Parallel Execution
```
parallel(
    dev: {
        terraformApply(workingDir: 'infra/dev')
    },
    prod: {
        terraformApply(workingDir: 'infra/prod')
    }
)
```
### 6.3 Plan Approval Flow
```
stage('Approval') {
    input message: "Approve Terraform Plan?"
}
```
---

## 7. Troubleshooting
| Issue                | Solution                |
| -------------------- | ----------------------- |
| Terraform init fails | Check backend config    |
| Permission denied    | Verify IAM roles        |
| State lock issue     | Check DynamoDB lock     |
| Plan mismatch        | Re-run `terraform plan` |

---

## 8. Contact Information

| Contact Type | Details                                                             |
| ------------ | ------------------------------------------------------------------- |
| Name         | Suraj Tripathi                                                      |
| Role         | DevOps Trainee                                                      |
| Email        | [suraj.tripathi.snaatak@mygurukulam.co](mailto:suraj.tripathi.snaatak@mygurukulam.co) |

---
## 9. References
| Topic                          | Description                                   | Link                                                                                                                                                 |
| ------------------------------ | --------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Terraform Documentation        | Official Terraform docs covering all concepts | [https://developer.hashicorp.com/terraform/docs](https://developer.hashicorp.com/terraform/docs)                                                     |
| Terraform State                | Understanding Terraform state management      | [https://developer.hashicorp.com/terraform/language/state](https://developer.hashicorp.com/terraform/language/state)                                 |
| Terraform Drift Detection      | Detecting infrastructure drift                | [https://developer.hashicorp.com/terraform/tutorials/state/resource-drift](https://developer.hashicorp.com/terraform/tutorials/state/resource-drift) |
| Remote Backend (S3 + DynamoDB) | Configure remote state storage and locking    | [https://developer.hashicorp.com/terraform/language/settings/backends/s3](https://developer.hashicorp.com/terraform/language/settings/backends/s3)   |
| Terraform CLI Commands         | All Terraform CLI commands and usage          | [https://developer.hashicorp.com/terraform/cli](https://developer.hashicorp.com/terraform/cli)                                                       |
| Jenkins Pipeline               | Official Jenkins pipeline documentation       | [https://www.jenkins.io/doc/book/pipeline/](https://www.jenkins.io/doc/book/pipeline/)                                                               |
| Jenkins Shared Library         | How to create and use shared libraries        | [https://www.jenkins.io/doc/book/pipeline/shared-libraries/](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)                             |
| AWS CLI                        | AWS CLI installation and usage                | [https://docs.aws.amazon.com/cli/](https://docs.aws.amazon.com/cli/)                                                                                 |
| AWS IAM                        | Identity and access management concepts       | [https://docs.aws.amazon.com/iam/](https://docs.aws.amazon.com/iam/)                                                                                 |




