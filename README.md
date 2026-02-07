# DevSecOps IaC on AWS (Terraform + Ansible + Jenkins + EKS)

Provision and configure AWS infrastructure using Infrastructure as Code, and run a CI/CD pipeline that builds, scans, and deploys workloads to Amazon EKS. [web:8]

## Repository structure

- `infrastructure-as-code/` — AWS provisioning and configuration automation using Terraform and Ansible (EKS, VPC, EC2, IAM, access).  
- `ci-cd-pipeline/` — Jenkins CI/CD pipelines and supporting assets for build, quality checks, security scans, and deployment to EKS.

## What this project delivers

- Amazon EKS cluster provisioned with Terraform using the official EKS and VPC modules.  
- Secure, automated provisioning of EC2 instances for:
  - Jenkins Controller
  - Jenkins Agent
  - SonarQube server
- EKS access configuration using `aws_eks_access_entry` to grant cluster access to an IAM role used by the Jenkins Agent instance.
- IAM role + instance profile attachment for the Jenkins Agent automated via Ansible.
- Secure handling of AWS credentials using Ansible Vault.

## Tech stack

- **IaC & config management:** Terraform, Ansible (roles), Ansible Vault  
- **Cloud:** AWS (VPC, EC2, IAM, EKS)  
- **CI/CD:** Jenkins (pipeline as code)  
- **Quality & security:** SonarQube (code quality / static analysis in pipeline)

## infrastructure-as-code/

This directory contains the end-to-end automation for provisioning and configuring AWS resources.

### Terraform (EKS + networking)

- Creates the base AWS network using the official Terraform VPC module.
- Creates an EKS cluster using the official Terraform EKS module.
- Uses `aws_eks_access_entry` to grant EKS access to an IAM role associated with the Jenkins Agent node.

### Ansible (EC2 provisioning + configuration)

- Provisions and configures EC2 instances for:
  - `jenkins-controller`
  - `jenkins-agent`
  - `sonarqube`
- Uses Ansible roles to keep configuration modular and repeatable.
- Creates and attaches an IAM role to the Jenkins Agent EC2 instance via instance profile (automated through an Ansible role).
- Stores AWS credentials securely using Ansible Vault and consumes them during automation runs.

## ci-cd-pipeline/

This directory contains Jenkins pipeline code and configuration to support CI/CD workflows targeting EKS.

### Pipeline capabilities (high level)

- Checkout source code from GitHub.
- Build and test application artifacts (where applicable).
- Run code quality analysis via SonarQube.
- Run security checks/scans as part of the pipeline stages (as configured in the Jenkinsfile/pipeline).
- Deploy to Kubernetes (EKS) using cluster access granted to the Jenkins Agent IAM role.

> Note: The pipeline implementation follows common Jenkins production patterns (multistage pipeline, tool integrations, gated quality/security steps) and is designed to be adapted per application repository.

## How EKS access is handled

The Jenkins Agent runs with an IAM role attached via instance profile. Terraform configures EKS access with `aws_eks_access_entry` so that the agent role can authenticate/authorize against the cluster for deployment operations.  

## Security considerations

- AWS secrets are not stored in plaintext; Ansible Vault is used for encrypting sensitive values and credentials.
- Access to EKS is explicitly granted to the Jenkins Agent IAM role via Terraform, instead of relying on manual cluster access steps.

## Usage (typical flow)

1. Use Terraform to provision VPC + EKS and create EKS access entry for the Jenkins Agent role.
2. Use Ansible roles to provision/configure EC2 instances (Jenkins Controller, Jenkins Agent, SonarQube) and attach IAM instance profile to the agent.
3. Configure Jenkins jobs/pipelines from `ci-cd-pipeline/` and run builds that deploy to EKS.

## Reference (optional)

For additional Jenkins pipeline patterns and examples, see:  
https://github.com/CloudWithVarJosh/Jenkins-Basics-To-Production/tree/main/Project%2001 [web:10]

## Author

Maintained by Aman Dhal.
