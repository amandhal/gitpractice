# DevOps Assessment - AWS EKS Infrastructure with CI/CD Automation

A production-grade infrastructure-as-code project demonstrating automated provisioning and deployment of a microservices application on AWS EKS with complete CI/CD pipeline.

## 📋 Project Overview

This project automates the entire lifecycle of deploying a containerized Node.js application to AWS EKS, including:
- Infrastructure provisioning using Terraform
- Kubernetes cluster setup with EKS
- Microservices deployment with Helm
- Continuous Integration/Continuous Deployment with GitHub Actions
- Auto-scaling and monitoring

## 🏗️ Architecture

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
| Infrastructure as Code | Terraform (HCL - 70.5%) | Cloud resource provisioning |
| Container Orchestration | AWS EKS | Managed Kubernetes |
| Containerization | Docker (2.4%) | Application packaging |
| Package Management | Helm | Kubernetes deployments |
| API Gateway | NGINX Ingress Controller | Application routing |
| Auto-scaling | HPA + Cluster Autoscaler | Dynamic resource scaling |
| Monitoring | Metrics Server | Resource metrics collection |
| Application Framework | Node.js (27.1%) | Microservices |
| CI/CD | GitHub Actions | Automated deployment pipeline |

## 🚀 Deployment Pipeline

The CI/CD workflow is automatically triggered on push to the `main` branch:

1. **Build Phase**: Docker image build and push
2. **Infrastructure Phase**: Terraform provisioning via Helm provider
3. **Deployment Phase**: Helm chart deployment to EKS cluster
4. **Validation Phase**: Application accessibility verification

## 📊 Language Composition

- **HCL (Terraform)**: 70.5% - Infrastructure definitions
- **JavaScript (Node.js)**: 27.1% - Application code
- **Dockerfile**: 2.4% - Container configurations

## ✨ Key Features

✅ **Infrastructure Automation**: Complete IaC with Terraform
✅ **Microservices Architecture**: Separated frontend and backend deployments
✅ **Auto-scaling**: HPA for pods + Cluster Autoscaler for nodes
✅ **High Availability**: Load balancing with NLB and Ingress
✅ **Resource Monitoring**: Metrics Server for observability
✅ **Automated Deployment**: GitHub Actions CI/CD pipeline
✅ **Helm Integration**: Declarative Kubernetes deployments

## 📦 Project Structure
. ├── terraform/ # Infrastructure as Code │ ├── eks/ # EKS cluster configuration │ ├── iam/ # IAM roles and policies │ ├── addons/ # Cluster Autoscaler, NGINX, Metrics Server │ └── main.tf # Primary configuration ├── helm/ # Kubernetes Helm charts │ ├── frontend/ # Frontend deployment chart │ ├── backend/ # Backend deployment chart │ └── values.yaml # Helm values configuration ├── app/ # Node.js application │ ├── frontend/ # Frontend application │ ├── backend/ # Backend application │ └── Dockerfile # Container build configuration └── .github/ └── workflows/ # GitHub Actions CI/CD pipelines


## 🔄 CI/CD Workflow

The GitHub Actions workflow:
1. Triggers on push to `main` branch
2. Builds and pushes Docker images
3. Provisions/updates infrastructure with Terraform
4. Deploys applications via Helm charts
5. Configures auto-scaling policies
6. Validates application accessibility

## 📈 Scaling Configuration

- **Pod Auto-scaling**: HPA monitors CPU/memory metrics and scales replicas
- **Node Auto-scaling**: Cluster Autoscaler adjusts node count based on pending pods
- **External Load Balancing**: NLB distributes traffic across backend instances

## 🔍 Monitoring & Observability

- Metrics Server collects resource utilization data
- HPA uses metrics for scaling decisions
- CloudWatch integration for centralized logging
- NGINX Ingress logs for traffic analysis

## 🚦 Getting Started

### Prerequisites
- AWS Account with appropriate permissions
- Terraform installed locally
- kubectl configured for EKS access
- Docker installed for local testing

### Deployment Steps

1. **Configure AWS credentials**
   ```bash
   aws configure
   

<img width="2460" height="2312" alt="image" src="https://github.com/user-attachments/assets/3619033d-551c-45dc-a944-ccec2522645e" />
