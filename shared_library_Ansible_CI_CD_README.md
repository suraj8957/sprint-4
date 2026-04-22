# Ansible Role & Playbook CI/CD Shared Library Documentation

## Table of Contents

- Overview
- Objectives
- Architecture
- Repository Structure
- Prerequisites
- Ansible Basics
- What is Ansible Role?
- What is Ansible Playbook?
- CI/CD Workflow Overview
- Shared Library Purpose
- Shared Library Functions
  - checkoutCode()
  - installDependencies()
  - ansibleLint()
  - syntaxCheck()
  - dryRun()
  - runPlaybook()
  - inventoryValidation()
  - securityScan()
- Jenkins CI Pipeline
- Jenkins CD Pipeline
- Sample Jenkinsfile (CI)
- Sample Jenkinsfile (CD)
- Inputs and Outputs
- Variables and Parameters
- Inventory Management
- Ansible Vault Integration
- Secrets Management
- Error Handling
- Rollback Strategy
- Notifications Integration
- Testing Strategy
- Security Best Practices
- Troubleshooting
- Known Limitations
- Best Practices
- References

---

# Overview

This shared library provides reusable CI/CD automation for **Ansible Roles** and **Ansible Playbooks** using Jenkins.

It standardizes:

- Playbook validation
- Linting
- Syntax testing
- Dry runs
- Deployment execution
- Security scanning
- Notifications
- Rollback handling

Supports:

- Infrastructure provisioning
- Configuration management
- Application deployment
- Server patching
- Environment automation

---

# Objectives

The goal of this shared library is to:

- Standardize Ansible CI/CD
- Reuse common pipeline code
- Reduce duplicate Jenkins logic
- Improve deployment reliability
- Enforce validation and security checks
- Support scalable automation

---

# Architecture

## CI Flow

```text
Developer Commit
    |
Git Push
    |
Jenkins Trigger
    |
Checkout
    |
Install Dependencies
    |
Ansible Lint
    |
Syntax Validation
    |
Inventory Validation
    |
Dry Run (--check)
    |
Security Scan
    |
Success/Failure Notification
```

---

## CD Flow

```text
Validated Code
   |
Deploy Trigger
   |
Approval (Optional)
   |
Run Playbook
   |
Post Validation
   |
Rollback (if needed)
   |
Notification
```

---

# Repository Structure

```bash
ansible-project/
│
├── inventories/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
├── playbooks/
│   ├── site.yml
│   └── deploy.yml
│
├── roles/
│   ├── nginx/
│   ├── app/
│   └── db/
│
├── group_vars/
├── host_vars/
├── Jenkinsfile
└── README.md
```

---

# Prerequisites

Required:

- Jenkins
- Git
- Python3
- Ansible
- ansible-lint
- ssh access
- Jenkins shared library configured

Install:

```bash
sudo apt update
sudo apt install ansible -y

pip install ansible-lint
```

Check:

```bash
ansible --version
ansible-playbook --version
```

---

# Ansible Basics

## Ad Hoc Command

```bash
ansible all -m ping
```

---

## Simple Playbook

```yaml
---
- hosts: web
  become: yes

  tasks:
   - name: Install nginx
     apt:
       name: nginx
       state: present
```

Run:

```bash
ansible-playbook install-nginx.yml
```

---

# What is Ansible Role?

Role organizes automation into reusable structure.

Example:

```bash
roles/
 nginx/
   tasks/
   handlers/
   vars/
   templates/
   files/
```

Use:

```yaml
roles:
 - nginx
```

---

# What is Ansible Playbook?

Playbook is YAML automation instructions.

Can perform:

- provisioning
- patching
- deployments
- configuration

---

# CI/CD Workflow Overview

## CI Tasks

Includes:

- Code checkout
- Syntax validation
- Linting
- Dry run
- Security checks

---

## CD Tasks

Includes:

- Actual deployment
- Target inventory execution
- Health checks
- Rollback

---

# Shared Library Purpose

Reusable Jenkins shared library provides standardized methods for:

- Validation
- Deployment
- Security
- Notifications
- Rollbacks

---

# Shared Library Functions

---

## checkoutCode()

Purpose:

Clone source code.

```groovy
checkoutCode(repo, branch)
```

Example:

```groovy
checkoutCode(
'https://github.com/org/repo.git',
'main'
)
```

---

## installDependencies()

Installs:

- ansible
- ansible-lint

```groovy
installDependencies()
```

---

## ansibleLint()

Lint validation.

```groovy
ansibleLint('playbooks/site.yml')
```

Runs:

```bash
ansible-lint playbooks/site.yml
```

---

## syntaxCheck()

Checks playbook syntax.

```groovy
syntaxCheck(
'playbooks/site.yml',
'inventories/dev'
)
```

Runs:

```bash
ansible-playbook --syntax-check
```

---

## dryRun()

Runs check mode.

```groovy
dryRun(
'playbooks/site.yml',
'inventories/dev'
)
```

Command:

```bash
ansible-playbook --check
```

---

## runPlaybook()

Actual deployment.

```groovy
runPlaybook(
'playbooks/site.yml',
'inventories/prod'
)
```

---

## inventoryValidation()

Validate inventory.

```groovy
inventoryValidation()
```

Checks:

```bash
ansible-inventory --list
```

---

## securityScan()

Runs security scanning.

Examples:

```bash
ansible-lint
yamllint
```

Optional:

- Trivy
- Checkov

---

# Jenkins CI Pipeline

```groovy
@Library('ansible-shared-library') _

pipeline {

 agent any

 stages {

 stage('Checkout'){
 steps{
 checkoutCode(REPO_URL,BRANCH)
 }
 }

 stage('Dependencies'){
 steps{
 installDependencies()
 }
 }

 stage('Lint'){
 steps{
 ansibleLint('playbooks/site.yml')
 }
 }

 stage('Syntax Check'){
 steps{
 syntaxCheck(
 'playbooks/site.yml',
 'inventories/dev'
 )
 }
 }

 stage('Dry Run'){
 steps{
 dryRun(
 'playbooks/site.yml',
 'inventories/dev'
 )
 }
 }

 stage('Security Scan'){
 steps{
 securityScan()
 }
 }

 }

 post{
 success{
 echo "Validation passed"
 }

 failure{
 echo "Pipeline failed"
 }
 }

}
```

---

# Jenkins CD Pipeline

```groovy
@Library('ansible-shared-library') _

pipeline {

agent any

stages {

stage('Deploy'){
steps{
runPlaybook(
'playbooks/site.yml',
'inventories/prod'
)
}
}

}

}
```

---

# Sample Jenkinsfile (CI)

```groovy
node {

try {

stage('Checkout'){
checkout scm
}

stage('Lint'){
sh 'ansible-lint playbooks/site.yml'
}

stage('Syntax'){
sh '''
ansible-playbook \
--syntax-check \
-i inventories/dev \
playbooks/site.yml
'''
}

}
catch(err){
currentBuild.result='FAILURE'
throw err
}

}
```

---

# Sample Jenkinsfile (CD)

```groovy
node {

stage('Deploy'){
sh '''
ansible-playbook \
-i inventories/prod \
playbooks/site.yml
'''
}

}
```

---

# Inputs and Outputs

| Input | Description |
|------|-------------|
| Repository URL | Git source |
| Branch | Git branch |
| Inventory | Target servers |
| Playbook | Deployment YAML |
| Credentials | SSH/Vault Secrets |

---

## Outputs

| Output | Description |
|------|-------------|
| Validation Report | CI results |
| Deployment Logs | Execution logs |
| Notifications | Slack/Email alerts |

---

# Variables and Parameters

Example:

```groovy
parameters {

choice(
name:'ENV',
choices:['dev','qa','prod']
)

string(
name:'PLAYBOOK',
defaultValue:'site.yml'
)

}
```

---

# Inventory Management

Example:

```ini
[web]
10.0.1.10

[db]
10.0.2.20
```

Dynamic inventory supported.

---

# Ansible Vault Integration

Encrypt secrets:

```bash
ansible-vault encrypt vars.yml
```

Use:

```bash
ansible-playbook --ask-vault-pass
```

---

# Secrets Management

Recommended:

- Jenkins Credentials
- Vault
- AWS Secrets Manager
- HashiCorp Vault

Never hardcode:

- passwords
- tokens
- ssh keys

---

# Error Handling

Example:

```groovy
try{
 runPlaybook()
}

catch(err){
 echo "Deployment failed"
 throw err
}
```

---

# Rollback Strategy

Example rollback:

```yaml
- name: Rollback
  hosts: app
  tasks:
   - name: Deploy previous version
```

Strategies:

- Backup restore
- Previous release deploy
- Blue Green rollback

---

# Notifications Integration

Slack example:

```groovy
slackSend(
channel:'#devops',
message:'Deployment Success'
)
```

Email supported too.

---

# Testing Strategy

Include:

- Syntax tests
- Dry run
- Molecule tests
- Integration tests

Example:

```bash
molecule test
```

---

# Security Best Practices

Use:

- Least privilege
- Vault encryption
- SSH keys
- ansible-lint
- Secret scanning

Never:

- commit secrets
- use plaintext passwords

---

# Troubleshooting

## Permission denied

```bash
chmod 400 key.pem
```

---

## Inventory issue

```bash
ansible-inventory --list
```

---

## Syntax errors

```bash
ansible-playbook --syntax-check
```

---

## Debug

```bash
ansible-playbook -vvv site.yml
```

---

# Known Limitations

Current limitations:

- Manual approvals may be needed
- Rollback depends on playbook design
- Large inventories increase runtime
- Dynamic inventory plugins require setup

---

# Best Practices

Recommended:

- Use roles not huge playbooks
- Keep playbooks idempotent
- Separate inventories
- Use dry runs before deploy
- Use linting always
- Version control everything
- Use shared libraries for reuse

---

# References

| Reference | Link |
|---------|------|
| Ansible Docs | https://docs.ansible.com |
| Ansible Playbooks | https://docs.ansible.com/ansible/latest/playbook_guide |
| Ansible Roles | https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html |
| Ansible Lint | https://ansible-lint.readthedocs.io |
| Jenkins Shared Libraries | https://www.jenkins.io/doc/book/pipeline/shared-libraries |
| Ansible Vault | https://docs.ansible.com/ansible/latest/vault_guide |
| Molecule Testing | https://molecule.readthedocs.io |

---

# Summary

This shared library supports:

✔ Ansible CI validation  
✔ Playbook deployment automation  
✔ Reusable Jenkins pipelines  
✔ Security controls  
✔ Rollback support  
✔ Notifications  
✔ Production-grade automation
