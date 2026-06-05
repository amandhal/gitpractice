# 🚀 Production-Grade Node.js on AWS EKS — Full DevOps Pipeline

A complete end-to-end DevOps project deploying a Node.js application on AWS EKS with Infrastructure as Code, GitOps CI/CD, auto-scaling, centralized logging, and full observability.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [Infrastructure (Terraform)](#-infrastructure-terraform)
- [Node.js Application](#-nodejs-application)
- [Helm Chart](#-helm-chart)
- [CI/CD Pipeline (GitHub Actions)](#-cicd-pipeline-github-actions)
- [Observability — Grafana Dashboards](#-observability--grafana-dashboards)
- [Logging — Kibana Dashboards](#-logging--kibana-dashboards)
- [Issues Faced & Fixes](#-issues-faced--fixes)
- [Improvements Roadmap](#-improvements-roadmap)
- [Deployment Guide — Terraform](#-deployment-guide--terraform)
- [Deployment Guide — Kubernetes](#-deployment-guide--kubernetes)

---

## 📌 Project Overview

This project provisions a production-ready EKS cluster on AWS and deploys a Node.js application through a fully automated CI/CD pipeline. The stack covers every layer of a modern cloud-native deployment:

| Layer | Technology |
|---|---|
| Cloud Infrastructure | AWS EKS, VPC, Subnets, NAT, Security Groups |
| IaC | Terraform (official AWS EKS + VPC modules, CloudDrove IAM module) |
| App | Node.js with `/health`, `/metrics`, structured JSON logs |
| Packaging | Docker → Docker Hub |
| Kubernetes | Custom Helm chart with HPA, Ingress, Probes, CronJob |
| Autoscaling | Cluster Autoscaler via EKS Pod Identity |
| Monitoring | Prometheus + Grafana (via Helm) |
| Logging | EFK Stack — Elasticsearch, Fluentbit, Kibana (via Helm) |
| CI/CD | GitHub Actions (CI on PR, CD on merge to main) |

---

## 🏗 Architecture

```
                        ┌─────────────────────────────────────┐
                        │              AWS VPC                 │
                        │  ┌──────────────┐ ┌───────────────┐ │
                        │  │ Public Subnet│ │Private Subnet │ │
                        │  │  (NAT GW,   │ │  (EKS Nodes,  │ │
                        │  │   Ingress)  │ │   Pods)       │ │
                        │  └──────────────┘ └───────────────┘ │
                        └─────────────────────────────────────┘
                                         │
                              ┌──────────▼──────────┐
                              │     EKS Cluster      │
                              │                      │
              ┌───────────────┼──────────────────────┼──────────────────┐
              │               │                      │                  │
     ┌────────▼──────┐  ┌─────▼──────┐   ┌──────────▼──────┐  ┌───────▼───────┐
     │  Node.js App  │  │ Prometheus │   │   Grafana        │  │  EFK Stack    │
     │  (Deployment) │  │ (Scraping  │   │  (Dashboards)    │  │  (ES+Kibana   │
     │  HPA + Ingress│  │  via SM)   │   │                  │  │  +Fluentbit)  │
     └───────────────┘  └────────────┘   └──────────────────┘  └───────────────┘
              │
     ┌────────▼──────┐
     │Cluster        │
     │Autoscaler     │
     │(Pod Identity) │
     └───────────────┘
```

---

## 🌍 Infrastructure (Terraform)

Infrastructure is defined under `terraform-eks/` and uses:

- **Official AWS EKS Terraform module** — provisions the EKS control plane and managed node groups.
- **Official AWS VPC Terraform module** — creates VPC, public + private subnets, route tables, NAT Gateway, and security groups automatically.
- **CloudDrove `aws-iam-role` module** — creates an IAM role with required policies attached.
- **`aws_eks_pod_identity_association` resource** — associates the IAM role with the service account used by Cluster Autoscaler, granting it the permissions it needs without static credentials.
- **Custom Terraform Helm module** — installs Cluster Autoscaler, Prometheus, Grafana, and the EFK stack into the cluster via the Helm provider.

> **State Management:** Remote state is stored in S3. See [Deployment Guide — Terraform](#-deployment-guide--terraform) for setup steps.

---

## 🟢 Node.js Application

A lightweight Express application exposing:

| Endpoint | Description |
|---|---|
| `GET /health` | Liveness/readiness health check |
| `GET /metrics` | Prometheus-compatible metrics (via `prom-client`) |
| Additional routes | Business logic endpoints |

**Key characteristics:**
- Structured JSON logging (compatible with Fluentbit parsing)
- Prometheus metrics instrumented at the application level
- Containerized via Docker and pushed to Docker Hub

---

## ⎈ Helm Chart

The custom Helm chart (`helm-chart-node-app/`) packages the full Kubernetes workload:

| Resource | Details |
|---|---|
| `Deployment` | Rolling update strategy, resource limits/requests |
| `Service` | ClusterIP for internal routing |
| `Ingress` | Nginx Ingress Controller |
| `HPA` | Horizontal Pod Autoscaler based on CPU/memory |
| `ConfigMap` + `Secret` | Externalised configuration and sensitive values |
| `Liveness Probe` | Hits `/health` to detect crashed pods |
| `Readiness Probe` | Hits `/health` before routing traffic |
| `CronJob` | Periodic API job |
| `ServiceMonitor` | Prometheus ServiceMonitor so the app is auto-discovered for scraping |

---

## 🔄 CI/CD Pipeline (GitHub Actions)

The workflow has two jobs:

### CI — Triggered on Pull Request to `main`

| Step | Action |
|---|---|
| Terraform Validate | Validates all `.tf` files for syntax and correctness |
| Docker Build Test | Builds the Docker image to catch build-time errors early |
| Kubernetes Lint | Lints Helm chart templates |
| Upload Artifacts | Stores build outputs for the CD job |

### CD — Triggered on Push (merge) to `main`

| Step | Action |
|---|---|
| Terraform Plan & Apply | Provisions or updates AWS infrastructure |
| Docker Build & Push | Builds a fresh image and pushes to Docker Hub with the commit SHA tag |
| Deploy to Cluster | Runs `helm upgrade --install` against the EKS cluster |
| Rollout Verification | Waits for the deployment rollout to complete successfully |
| Smoke Test | Hits the `/health` endpoint to confirm the live deployment is healthy |

---

## 📊 Observability — Grafana Dashboards

Four dashboards are configured in Grafana:

| Dashboard | Panels |
|---|---|
| **Pod Resources** | CPU usage per pod, Memory usage per pod |
| **Node Resources** | Node CPU utilisation, Node memory utilisation |
| **API Latency** | P50 / P95 / P99 request duration histograms |
| **Request Count** | Total requests/sec broken down by endpoint and status code |

Metrics are scraped by Prometheus via the `ServiceMonitor` resource deployed with the Helm chart.

---

## 📜 Logging — Kibana Dashboards

A custom Kibana dashboard visualises application logs shipped by Fluentbit into Elasticsearch:

| Panel | Description |
|---|---|
| Error rate over time | Log volume filtered to `level: error` |
| Request volume | Total log events per minute |
| Status code distribution | Pie/bar chart of HTTP response codes |
| Top endpoints | Most-hit API paths by request count |

Logs are structured as JSON at the application level so Kibana field parsing works out of the box.

---

## 🐛 Issues Faced & Fixes

### 1. CloudDrove IAM Role Module — Version Incompatibility

**Problem:** Using the latest version of the CloudDrove `aws-iam-role` module threw a Terraform error caused by a bug inside the module's `version.tf`.

**Fix:** Instead of patching the module manually (which would break the CI/CD pipeline since it pulls modules fresh), the previous stable version (latest − 1) was used. It worked without modification.

---

### 2. Cluster Autoscaler Pod Crashing — Missing Permissions at Startup

**Problem:** After deployment, the Cluster Autoscaler pod was crash-looping. Logs showed it lacked the IAM permissions needed to describe and modify EC2 Auto Scaling Groups — even though the IAM role and `aws_eks_pod_identity_association` resource had been created.

**Root Cause:** Terraform was creating the `aws_eks_pod_identity_association` resource *after* the Cluster Autoscaler Helm release had already completed. The pod started before the identity association existed, so it inherited no permissions.

**Fix:**
1. **Immediate:** Restarted the pod (`kubectl rollout restart`) after the association was confirmed active.
2. **Permanent:** Added `depends_on = [aws_eks_pod_identity_association.cluster_autoscaler]` to the Helm release resource, ensuring the association is fully created before the Helm chart is deployed.

---

## 🗺 Improvements Roadmap

### HTTPS with cert-manager

Currently the Nginx Ingress serves traffic over plain HTTP. The next step is to enable automatic TLS using [cert-manager](https://cert-manager.io/):

1. Install cert-manager into the cluster via Helm.
2. Create a `ClusterIssuer` resource pointing to Let's Encrypt (staging first, then production).
3. Annotate the `Ingress` resource with `cert-manager.io/cluster-issuer` and add a `tls` block specifying the secret name and host.
4. cert-manager will automatically provision and renew the TLS certificate via the ACME HTTP-01 or DNS-01 challenge.

This eliminates the need to manually manage certificates and ensures zero-downtime renewals.

---

## 🚢 Deployment Guide — Terraform

### 1. Bootstrap remote state

Create an S3 bucket for Terraform state, then update the bucket name in `terraform-eks/providers.tf`:

```hcl
terraform {
  backend "s3" {
    bucket = "your-tf-state-bucket-name"
    key    = "eks/terraform.tfstate"
    region = "your-aws-region"
  }
}
```

### 2. Run Terraform

```bash
cd terraform-eks

terraform init
terraform validate
terraform plan
terraform apply
```

### 3. Deploy Fluentbit manually

Fluentbit requires the Elasticsearch password, which is only available after Elasticsearch is created by Terraform. Once `terraform apply` completes:

1. Retrieve the generated Elasticsearch password from the Kubernetes secret.
2. Add the password to `fluentbit-values.yaml`.
3. Install Fluentbit:

```bash
helm upgrade --install fluent-bit \
  oci://ghcr.io/fluent/helm-charts/fluent-bit \
  -n logging \
  -f fluentbit-values.yaml \
  --version 0.57.6
```

---

## ☸️ Deployment Guide — Kubernetes

### 1. Configure kubeconfig

```bash
aws eks update-kubeconfig \
  --region your-aws-region \
  --name your-cluster-name
```

### 2. Deploy the application

```bash
helm install node-app helm-chart-node-app -n node-app
```

To upgrade after a new image is pushed:

```bash
helm upgrade node-app helm-chart-node-app -n node-app \
  --set image.tag=<new-tag>
```

---

> Built with Terraform · Kubernetes · Helm · GitHub Actions · Prometheus · Grafana · EFK Stack
