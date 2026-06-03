# DevOps Assessment - AWS EKS Infrastructure with CI/CD Automation

A production-grade infrastructure-as-code project demonstrating automated provisioning and deployment of a microservices application on AWS EKS with complete CI/CD pipeline.

## Project Overview

This project automates the entire lifecycle of deploying a containerized Node.js application to AWS EKS, including:
- Infrastructure provisioning using Terraform
- Kubernetes cluster setup with EKS
- Microservices deployment with Helm
- Continuous Integration/Continuous Deployment with GitHub Actions
- Auto-scaling and monitoring

## Architecture
<img width="2460" height="2312" alt="image" src="https://github.com/user-attachments/assets/3619033d-551c-45dc-a944-ccec2522645e" />

### Infrastructure Layer (Terraform - HCL)
- **AWS EKS Cluster**: Managed Kubernetes using official AWS and EKS Terraform modules
- **IAM Roles**: CloudDrove's IAM role module for secure access management
- **Custom Components**:
  - Cluster Autoscaler for dynamic node scaling
  - NGINX Ingress Controller for L7 routing
  - Metrics Server for resource monitoring
  - Network Load Balancer (NLB) for external access

### Application Layer (Node.js)
- Frontend and Backend microservices
- 2 Kubernetes Deployments (separate frontend & backend)
- 2 ClusterIP Services for internal service discovery
- Horizontal Pod Autoscaler (HPA) for automatic scaling on both deployments

### Container & Deployment
- Docker containerization of Node applications
- Images pushed to Docker Hub/ECR
- Custom Helm charts for Kubernetes deployment
- Ingress configuration with NLB for external user access

## 🛠️ Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Infrastructure as Code | Terraform | Cloud resource provisioning |
| Container Orchestration | AWS EKS | Managed Kubernetes |
| Containerization | Docker | Application packaging |
| Package Management | Helm | Kubernetes deployments |
| External Access | NGINX Ingress Controller | Application routing |
| Auto-scaling | HPA + Cluster Autoscaler | Dynamic resource scaling |
| Monitoring | Metrics Server | Resource metrics collection |
| Application Framework | Node.js | Microservices |
| CI/CD | GitHub Actions | Automated deployment pipeline |


# CI/CD Pipeline Overview

This GitHub Actions workflow automates the validation, deployment, and verification of the application using separate Continuous Integration (CI) and Continuous Deployment (CD) stages.

## Continuous Integration (Pull Requests)

The CI pipeline is triggered whenever a Pull Request is opened against the `main` branch.

### Workflow Steps

1. Checkout the repository source code.
2. Configure Terraform and initialize the infrastructure code.
3. Validate Terraform configurations.
4. Build frontend and backend Docker images to verify successful image creation.
5. Lint the Helm chart to validate Kubernetes manifests and chart structure.
6. Upload the Helm chart as a workflow artifact for inspection.

This stage ensures that infrastructure code, application containers, and Kubernetes manifests are valid before merging changes into the main branch.

---

## Continuous Deployment (Push to Main)

The CD pipeline is triggered whenever code is merged into the `main` branch.

### Workflow Steps

1. Checkout the repository source code.
2. Configure AWS credentials and Terraform.
3. Provision or update AWS infrastructure using Terraform.
4. Build frontend and backend Docker images.
5. Push versioned Docker images to Docker Hub using the Git commit SHA as the image tag.
6. Configure access to the EKS cluster by updating the kubeconfig.
7. Deploy or upgrade the application using Helm.
8. Verify successful rollout of frontend and backend deployments.
9. Execute smoke tests to confirm application health and connectivity.

---

## Deployment Strategy

- Docker images are tagged using the Git commit SHA to provide immutable versioning.
- Helm performs deployments using `helm upgrade --install`, enabling both initial installation and subsequent upgrades.
- Rollout checks ensure Kubernetes deployments become healthy before proceeding.
- Smoke tests validate that frontend and backend services are reachable after deployment.

This approach provides automated validation, infrastructure provisioning, application deployment, and post-deployment verification within a single GitHub Actions workflow.

## Key Features

✅ **Infrastructure Automation**: Complete IaC with Terraform
✅ **Microservices Architecture**: Separated frontend and backend deployments
✅ **Auto-scaling**: HPA for pods + Cluster Autoscaler for nodes
✅ **High Availability**: Load balancing with NLB and Ingress
✅ **Resource Monitoring**: Metrics Server for observability
✅ **Automated Deployment**: GitHub Actions CI/CD pipeline
✅ **Helm Integration**: Declarative Kubernetes deployments





## Scaling Configuration

- **Pod Auto-scaling**: HPA monitors CPU/memory metrics and scales replicas
- **Node Auto-scaling**: Cluster Autoscaler adjusts node count based on pending pods
- **External Load Balancing**: NLB distributes traffic across backend instances
