# EKS Platform Automation Project

## Table of Contents
- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Infrastructure Provisioning](#infrastructure-provisioning)
- [Platform Services](#platform-services)
- [Application Deployment](#application-deployment)
- [Observability](#observability)
- [CI/CD Pipeline](#cicd-pipeline)
- [Issues Faced and Fixes](#issues-faced-and-fixes)
- [Deployment Steps](#deployment-steps)
- [Roadmap](#roadmap)

## Project Overview
This project delivers an end-to-end AWS EKS platform built with Terraform, Helm, Docker, and GitHub Actions. It covers infrastructure provisioning, cluster add-ons, application deployment, observability, and CI/CD automation in a single workflow.

The platform starts with an Amazon EKS cluster created using the official AWS EKS and VPC Terraform modules. These modules provision the core networking and cluster resources under the hood, including the VPC, public and private subnets, route tables, NAT, and security groups.

On top of the base infrastructure, a custom Terraform module installs and manages operational tooling such as Cluster Autoscaler, Prometheus, Grafana, and the EFK stack through the Terraform Helm provider. This keeps cluster services declarative and version controlled alongside the infrastructure.

The project also includes a production-oriented Node.js application packaged as a Docker image and deployed to EKS with a custom Helm chart. The chart includes health endpoints, metrics exposure, structured JSON logging, autoscaling, probes, ingress, and a cronjob, making it a good example of a cloud-native deployment.

## Architecture
The solution is organized into four layers:

1. **Infrastructure layer**: Terraform provisions the VPC, subnets, routing, NAT, security groups, and the EKS control plane.
2. **Platform layer**: Terraform and Helm install cluster-level services such as Cluster Autoscaler, Prometheus, Grafana, Elasticsearch, Fluent Bit, and Kibana.
3. **Application layer**: A custom Helm chart deploys the Node.js application, Kubernetes resources, autoscaling configuration, and monitoring templates.
4. **Delivery layer**: GitHub Actions validates, builds, plans, applies, deploys, and verifies changes through CI and CD workflows.

This layered approach separates concerns clearly. Infrastructure, platform services, application manifests, and delivery automation can evolve independently while still working as one system.

## Infrastructure Provisioning
The EKS cluster was provisioned using the official AWS Terraform modules for EKS and VPC. This provides a reliable baseline for creating production-ready AWS infrastructure using community-standard modules.

The networking stack includes:
- VPC
- Public and private subnets
- Route tables
- NAT gateway
- Security groups

A custom Terraform module was then used to install platform add-ons through the Helm provider. This made it possible to keep both infrastructure and cluster add-ons managed through Terraform rather than splitting responsibility across multiple tools.

For IAM integration, CloudDrove's `aws iam-role` Terraform module was used to create roles and attach the required IAM policies. Those roles were associated with Kubernetes service accounts using `aws_eks_pod_identity_association`, allowing workloads such as Cluster Autoscaler to access AWS APIs with least-privilege permissions.

## Platform Services
Several operational components were installed to make the cluster production-oriented:

- **Cluster Autoscaler** to scale worker nodes based on unschedulable pods.
- **Prometheus** to collect application and cluster metrics.
- **Grafana** to visualize infrastructure and application performance.
- **EFK stack** (Elasticsearch, Fluent Bit, Kibana) to centralize and analyze logs.

This setup gives visibility into cluster health, application behavior, and operational events. It also creates a strong foundation for alerting, debugging, capacity planning, and performance monitoring.

## Application Deployment
A Node.js application was created with endpoints such as `/health` and `/metrics`, along with additional APIs and structured JSON logs. The application was containerized and pushed to Docker Hub for use in the deployment pipeline.

A custom Helm chart was created to deploy the application to EKS. The chart includes:

- Deployments
- Services
- NGINX Ingress integration
- ConfigMaps and Secrets
- Resource requests and limits
- Liveness and readiness probes
- Rolling update strategy
- Horizontal Pod Autoscaler (HPA)
- A simple CronJob for API-related scheduled work
- A Prometheus `ServiceMonitor` template for automatic metrics discovery

Including the `ServiceMonitor` directly in the Helm chart ensures that once the application is deployed, Prometheus can immediately discover and scrape its metrics. This removes the need for separate monitoring manifests and improves deployment consistency.

## Observability
The monitoring and logging stack was designed to cover both infrastructure and application visibility.

### Grafana Dashboards
Grafana dashboards were created for:
- Pod CPU and memory usage
- Node CPU and memory usage
- API latency
- Request count

These dashboards help identify resource pressure, scaling behavior, and application performance trends over time.

### Kibana Dashboards
Custom Kibana dashboards were created to track:
- Errors
- Requests
- Status codes
- Top endpoints

This makes it easier to debug application issues, inspect traffic patterns, and understand endpoint-level behavior from centralized logs.

## CI/CD Pipeline
The project uses GitHub Actions with separate **CI** and **CD** jobs.

### CI Job
The CI workflow is triggered on pull requests targeting the `main` branch. Its purpose is to validate changes before they are merged.

It performs:
- Terraform validate
- Docker build test
- Kubernetes lint
- Artifact upload

### CD Job
The CD workflow is triggered on pushes to the `main` branch. Its purpose is to apply approved changes and deploy the latest application version.

It performs:
- Terraform plan and apply
- Docker build and push
- Deployment to the EKS cluster
- Rollout verification
- API smoke test

This separation between CI and CD improves safety and clarity. Pull requests focus on validation, while merges to `main` act as the promotion point for actual deployment.

## Issues Faced and Fixes
### 1. CloudDrove IAM Role Module Version Issue
When using the latest version of the CloudDrove IAM role module, an error occurred because of an issue in the module's `version.tf`. Instead of modifying the module source directly, which would make the pipeline brittle and harder to maintain, an older stable version (`latest - 1`) was used.

This fix preserved CI/CD compatibility and kept the implementation maintainable. It is usually better to pin to a working version than patch third-party module code locally unless there is a strong reason to fork it.

### 2. Cluster Autoscaler Permission Issue
The Cluster Autoscaler pod was crashing because it did not have the AWS permissions it needed at startup. Even though the IAM role and pod identity association resources existed in Terraform, the `aws_eks_pod_identity_association` was being created after the autoscaler deployment had already completed.

After troubleshooting the logs, it became clear that the pod needed to be restarted once the association was in place. To prevent this race condition in future runs, `depends_on` was added so the pod identity association is created before the autoscaler deployment proceeds.

## Deployment Steps
### Terraform Deployment
1. Create an S3 bucket for Terraform state.
2. Update the bucket name in `providers.tf`.
3. Change into the Terraform directory:

```bash
cd terraform-eks
```

4. Initialize Terraform:

```bash
terraform init
```

5. Validate the configuration:

```bash
terraform validate
```

6. Review the execution plan:

```bash
terraform plan
```

7. Apply the infrastructure:

```bash
terraform apply
```

### Manual Fluent Bit Step
One logging component needs to be deployed manually: Fluent Bit. The reason is that the Elasticsearch password is only available after the Elasticsearch secret is created, and that password must then be added to `fluentbit-values.yaml`.

After updating the values file, deploy Fluent Bit with:

```bash
helm upgrade --install fluent-bit oci://ghcr.io/fluent/helm-charts/fluent-bit \
  -n logging \
  -f fluentbit-values.yaml \
  --version 0.57.6
```

### Kubernetes Application Deployment
To connect to the cluster:

```bash
aws eks update-kubeconfig --region your-aws-region --name your-cluster-name
```

Then deploy the application Helm chart:

```bash
helm install node-app helm-chart-node-app -n node-app
```

## Roadmap
### Enable HTTPS with cert-manager
A strong next improvement is enabling HTTPS using **cert-manager**. At the moment, ingress can expose the application over HTTP, but production environments should terminate TLS to encrypt traffic between users and the platform.

With cert-manager, TLS certificates can be issued and renewed automatically, commonly using Let's Encrypt. This removes the operational burden of manually creating, rotating, and updating certificates in Kubernetes.

A good implementation path would be:
- Install cert-manager in the cluster.
- Create a `ClusterIssuer` or `Issuer` for Let's Encrypt.
- Update the Ingress resource in the Helm chart to include TLS configuration.
- Add DNS records that point the domain to the ingress load balancer.
- Let cert-manager automatically request and renew certificates.

This improvement would strengthen security, improve browser trust, and make the application more production-ready for real-world exposure.
