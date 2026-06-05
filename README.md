# EKS Observability & CI/CD DevOps Platform

## Table of Contents

* [Project Overview](#project-overview)

  * [Key Components](#key-components)

    * [Infrastructure (Terraform)](#infrastructure-terraform)
    * [IAM & Pod Identity](#iam--pod-identity)
    * [Application](#application)
* [Kubernetes Deployment](#kubernetes-deployment)

  * [Workload Resources](#workload-resources)
  * [Reliability Features](#reliability-features)
  * [Scaling Features](#scaling-features)
  * [Operations Features](#operations-features)
* [Monitoring Stack](#monitoring-stack)

  * [Prometheus](#prometheus)
  * [Grafana Dashboards](#grafana-dashboards)
* [Logging Stack (EFK)](#logging-stack-efk)

  * [Log Collection Flow](#log-collection-flow)
  * [Kibana Dashboards](#kibana-dashboards)
* [CI/CD Pipeline](#cicd-pipeline)

  * [CI Pipeline](#ci-pipeline)
  * [CD Pipeline](#cd-pipeline)
* [Terraform Deployment](#terraform-deployment)

  * [Prerequisites](#prerequisites)
  * [Remote State Setup](#remote-state-setup)
  * [Deploy Infrastructure](#deploy-infrastructure)
* [Application Deployment](#application-deployment)
* [Architecture Overview](#architecture-overview)
* [Challenges & Solutions](#challenges--solutions)

  * [CloudDrove IAM Module Compatibility Issue](#1-clouddrove-iam-module-compatibility-issue)
  * [Cluster Autoscaler Permission Issue](#2-cluster-autoscaler-permission-issue)
* [Future Improvements](#future-improvements)

  * [Enable HTTPS Using cert-manager](#enable-https-using-cert-manager)
  * [Additional Enhancements](#additional-enhancements)
* [Technologies Used](#technologies-used)

  * [Cloud & Infrastructure](#cloud--infrastructure)
  * [Containerization](#containerization)
  * [Kubernetes](#kubernetes)
  * [Monitoring](#monitoring)
  * [Logging](#logging)
  * [CI/CD](#cicd)
  * [Application](#application-1)
* [References](#references)

---

## Project Overview

This project demonstrates the deployment of a production-style Node.js application on Amazon EKS using Infrastructure as Code, GitOps principles, automated CI/CD pipelines, monitoring, logging, and autoscaling capabilities.

The infrastructure is provisioned using Terraform and deployed on AWS EKS. The application is packaged and deployed through a custom Helm chart and continuously delivered using GitHub Actions.

### Key Components

#### Infrastructure (Terraform)

The EKS infrastructure is created using the official Terraform AWS modules:

* AWS EKS Terraform Module
* AWS VPC Terraform Module

These modules automatically provision:

* VPC
* Public and Private Subnets
* Route Tables
* NAT Gateway
* Security Groups
* EKS Control Plane
* Managed Node Groups

Additionally, a custom Terraform module was created to deploy operational tooling directly into the cluster using the Terraform Helm Provider:

* Cluster Autoscaler
* Prometheus
* Grafana
* EFK Stack (Elasticsearch, Fluent Bit, Kibana)

#### IAM & Pod Identity

To securely grant AWS permissions to Kubernetes workloads:

* CloudDrove's IAM Role Terraform Module was used to create IAM roles.
* Required AWS IAM policies were attached to the roles.
* `aws_eks_pod_identity_association` resources were used to associate IAM roles with Kubernetes Service Accounts.
* Cluster Autoscaler receives permissions through Pod Identity rather than static AWS credentials.

#### Application

A custom Node.js application was developed with:

* `/health` endpoint for health checks
* `/metrics` endpoint for Prometheus metrics scraping
* Structured JSON logging
* Request tracking and metrics collection

The application was containerized using Docker and published to Docker Hub.

---

## Kubernetes Deployment

The application is deployed through a custom Helm chart that includes:

### Workload Resources

* Deployments
* Services
* ConfigMaps
* Secrets

### Reliability Features

* Resource Requests & Limits
* Liveness Probes
* Readiness Probes
* Rolling Updates

### Scaling Features

* Horizontal Pod Autoscaler (HPA)
* Cluster Autoscaler integration

### Operations Features

* NGINX Ingress Controller
* API Health Check CronJob
* Prometheus ServiceMonitor

The ServiceMonitor automatically allows Prometheus Operator to discover and scrape application metrics after deployment.

---

## Monitoring Stack

### Prometheus

Used for:

* Application metrics collection
* Kubernetes metrics collection
* Node metrics collection

### Grafana Dashboards

Custom dashboards were created for:

* Pod CPU Usage
* Pod Memory Usage
* Node CPU Usage
* Node Memory Usage
* API Request Count
* API Latency

---

## Logging Stack (EFK)

The EFK stack consists of:

* Elasticsearch
* Fluent Bit
* Kibana

### Log Collection Flow

```text
Application Logs → Fluent Bit → Elasticsearch → Kibana
```

### Kibana Dashboards

Custom dashboards were created for:

* Error Tracking
* Request Volume
* HTTP Status Codes
* Top Endpoints
* Application Activity Analysis

---

## CI/CD Pipeline

GitHub Actions is used to implement Continuous Integration and Continuous Deployment.

### CI Pipeline

Triggered on Pull Requests targeting the `main` branch.

#### CI Steps

* Terraform Validate
* Docker Build Verification
* Helm/Kubernetes Manifest Linting
* Artifact Upload

This ensures infrastructure and application changes are validated before merge.

### CD Pipeline

Triggered on Push events to the `main` branch.

#### CD Steps

* Terraform Plan
* Terraform Apply
* Docker Build
* Docker Image Push
* Helm Deployment
* Rollout Verification
* API Smoke Tests

This provides fully automated infrastructure provisioning and application deployment.

---

## Terraform Deployment

### Prerequisites

* AWS CLI configured
* Terraform installed
* AWS permissions for EKS deployment

### Remote State Setup

Create an S3 bucket for Terraform state storage.

Update the bucket name inside:

```bash
terraform-eks/providers.tf
```

### Deploy Infrastructure

```bash
cd terraform-eks

terraform init

terraform validate

terraform plan

terraform apply
```

---

## Application Deployment

Update your kubeconfig:

```bash
aws eks update-kubeconfig \
  --region <your-aws-region> \
  --name <your-cluster-name>
```

Deploy the application:

```bash
helm install node-app helm-chart-node-app -n node-app --create-namespace
```

Verify deployment:

```bash
kubectl get pods -n node-app

kubectl get svc -n node-app

kubectl get ingress -n node-app
```

---

## Architecture Overview

```text
                    GitHub
                       |
                       |
                 GitHub Actions
                 /            \
                /              \
         Terraform           Docker
              |                 |
              |                 |
              v                 v
         AWS Infrastructure   Docker Hub
              |
              |
              v
            Amazon EKS
              |
   --------------------------------
   |              |              |
   v              v              v
Node App     Prometheus      EFK Stack
                 |               |
                 v               v
             Grafana         Kibana
```

---

## Challenges & Solutions

### 1. CloudDrove IAM Module Compatibility Issue

#### Issue

The latest version of the CloudDrove IAM Role module failed during deployment due to an issue within the module configuration.

#### Solution

Instead of modifying the module source code (which would create CI/CD maintenance issues), an earlier stable version of the module was used.

This resolved the issue while preserving upgradeability and automation.

---

### 2. Cluster Autoscaler Permission Issue

#### Issue

The Cluster Autoscaler pod continuously crashed due to missing AWS permissions.

Investigation showed:

* IAM role existed
* Policies were attached
* Pod Identity Association existed

However, the Autoscaler deployment was starting before the Pod Identity Association was fully created.

#### Solution

A Terraform `depends_on` relationship was added to ensure:

```text
IAM Role
    ↓
Pod Identity Association
    ↓
Cluster Autoscaler Deployment
```

This guaranteed the workload received permissions before startup and eliminated the issue permanently.

---

## Future Improvements

### Enable HTTPS Using cert-manager

Currently the application can be exposed over HTTP.

A production-grade improvement would be integrating cert-manager with Let's Encrypt.

Benefits include:

* Automatic TLS certificate generation
* Automatic certificate renewal
* Encrypted communication
* Improved security posture
* Production-ready ingress configuration

Implementation would involve:

1. Installing cert-manager
2. Creating a ClusterIssuer
3. Integrating Ingress resources with TLS
4. Automatically provisioning Let's Encrypt certificates

### Additional Enhancements

* ArgoCD GitOps deployment model
* Terraform CI/CD approval workflow
* External Secrets Operator integration
* Grafana Alerting
* Multi-environment support (Dev / Stage / Prod)
* AWS Load Balancer Controller
* Karpenter for advanced node provisioning
* OpenTelemetry-based distributed tracing

---

## Technologies Used

### Cloud & Infrastructure

* AWS EKS
* AWS VPC
* Terraform

### Containerization

* Docker
* Docker Hub

### Kubernetes

* Amazon EKS
* Helm
* NGINX Ingress Controller
* Cluster Autoscaler

### Monitoring

* Prometheus
* Grafana

### Logging

* Elasticsearch
* Fluent Bit
* Kibana

### CI/CD

* GitHub Actions

### Application

* Node.js
* Express.js

---

## References

* https://github.com/terraform-aws-modules/terraform-aws-eks
* https://github.com/terraform-aws-modules/terraform-aws-vpc
* https://github.com/cloudposse/terraform-aws-eks-pod-identity
* https://kubernetes.io
* https://helm.sh
* https://prometheus.io
* https://grafana.com
* https://www.elastic.co
