graph TB
    subgraph "AWS Cloud (ap-south-1)"
        subgraph "VPC (10.10.0.0/16)"
            subgraph "Public Subnets"
                NAT["NAT Gateway"]
                LB["Load Balancer"]
            end
            
            subgraph "Private Subnets - AZ-a & AZ-b"
                EKS["EKS Cluster<br/>kubernetes-version: 1.35<br/>2-5 Nodes<br/>c7i-flex.large"]
                
                subgraph "Kubernetes Workloads"
                    FE["Frontend Pod<br/>amandhal/node-app-frontend:1.0.0<br/>replicas: 2"]
                    BE["Backend Pod<br/>amandhal/node-app-backend:1.0.0<br/>replicas: 2"]
                    CA["Cluster Autoscaler<br/>Pod Identity"]
                    CD["CoreDNS<br/>Kube-Proxy<br/>VPC-CNI"]
                end
            end
        end
    end
    
    subgraph "IAM & Security"
        IAM["IAM Roles<br/>- Cluster Creator Admin<br/>- Cluster Autoscaler<br/>- SSM Core Permissions<br/>- Pod Identity Association"]
    end
    
    subgraph "Deployment & Configuration"
        TF["Terraform<br/>- VPC Module v6.6.1<br/>- EKS Module v21.23.0<br/>- IAM Role Module v1.3.4<br/>- Helm Add-ons Module"]
        
        HC["Helm Chart<br/>node-app<br/>v0.1.0"]
        
        DOC["Docker Image<br/>Node.js App<br/>amandhal/node-app"]
    end
    
    subgraph "CI/CD"
        GH[".github/workflows<br/>GitHub Actions"]
    end
    
    User["End Users"]
    
    User -->|Access| LB
    LB -->|Route| FE
    LB -->|Route| BE
    FE -.->|Internal| BE
    
    EKS -->|Managed by| TF
    HC -->|Deploys to| EKS
    DOC -->|Built into| FE
    DOC -->|Built into| BE
    
    TF -->|Creates| IAM
    IAM -->|Authenticates| CA
    CA -->|Scales| EKS
    
    GH -->|Triggers| TF
    GH -->|Triggers| HC
    
    CD -->|Manages| EKS
    NAT -->|Egress| EKS
    
    style EKS fill:#FF9900
    style FE fill:#4285F4
    style BE fill:#4285F4
    style LB fill:#34A853
    style TF fill:#623CE4
    style HC fill:#0F1419
    style IAM fill:#FF6B6B
    style CA fill:#FFD700
