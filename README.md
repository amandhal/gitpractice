# Project Overview

This project demonstrates the deployment of a production-oriented two-tier microservices application on Amazon EKS using Infrastructure as Code, Kubernetes, Helm, and GitHub Actions.

The infrastructure is provisioned using Terraform and consists of a custom VPC with two public and two private subnets distributed across multiple Availability Zones. A NAT Gateway deployed in the public subnet enables outbound internet access for workloads running in private subnets. The Kubernetes control plane is managed by Amazon EKS, while application workloads run on an EKS Managed Node Group deployed entirely within private subnets for improved security.

To support scalability and high availability, the cluster includes the Kubernetes Cluster Autoscaler, which automatically adjusts the number of worker nodes based on workload demands. An NGINX Ingress Controller is deployed and exposed through a Network Load Balancer (NLB) in the public subnet, providing external access to services running inside the cluster.

The application consists of two Node.js microservices:

* Frontend Service – User-facing web application
* Backend Service – REST API exposing `/health`, `/metrics`, and application endpoints

The application is packaged and deployed using a custom Helm chart. Both frontend and backend services are exposed internally using Kubernetes ClusterIP services, enabling secure service-to-service communication through Kubernetes DNS and networking.

High availability and elasticity are achieved through Horizontal Pod Autoscalers (HPA) configured for both frontend and backend deployments. These HPAs automatically scale the number of application pods based on resource utilization metrics collected by the Kubernetes Metrics Server.

A GitHub Actions CI/CD pipeline automates the complete software delivery process, including infrastructure validation, container image builds, image publishing to Docker Hub, infrastructure provisioning with Terraform, application deployment using Helm, rollout verification, and smoke testing.

The overall solution demonstrates modern DevOps practices including Infrastructure as Code (Terraform), containerization (Docker), orchestration (Kubernetes), package management (Helm), automated CI/CD (GitHub Actions), autoscaling, and cloud-native application deployment on AWS.


<img width="2460" height="2312" alt="image" src="https://github.com/user-attachments/assets/3619033d-551c-45dc-a944-ccec2522645e" />
