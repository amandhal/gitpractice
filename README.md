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
