# Infrastructure Architecture Diagram

This diagram represents the AWS infrastructure and Kubernetes deployment for the Node.js application.

```mermaid
graph TD
    subgraph AWS_Cloud ["AWS Cloud (ap-south-1)"]
        subgraph VPC ["VPC (10.10.0.0/16)"]
            IGW["Internet Gateway"]
            NAT["NAT Gateway"]
            
            subgraph Public_Subnets ["Public Subnets (AZ1 & AZ2)"]
                NLB["AWS NLB (Nginx Ingress)"]
            end
            
            subgraph Private_Subnets ["Private Subnets (AZ1 & AZ2)"]
                subgraph EKS_Cluster ["EKS Cluster (v1.35)"]
                    direction TB
                    
                    subgraph K8s_System ["kube-system / ingress-nginx"]
                        IngressController["Nginx Ingress Controller"]
                        Autoscaler["Cluster Autoscaler"]
                        Metrics["Metrics Server"]
                    end
                    
                    subgraph App_Namespace ["Application"]
                        Ingress["K8s Ingress Resource"]
                        FService["Frontend Service (ClusterIP)"]
                        FDeployment["Frontend Pods (HPA)"]
                        BService["Backend Service (ClusterIP)"]
                        BDeployment["Backend Pods (HPA)"]
                        Cron["CronJob"]
                        Config["ConfigMap (BACKEND_URL)"]
                    end
                end
            end
        end
    end

    User((User)) --> IGW
    IGW --> NLB
    NLB --> IngressController
    IngressController --> Ingress
    Ingress --> FService
    FService --> FDeployment
    FDeployment -- "Internal API Call" --> BService
    BService --> BDeployment
    FDeployment -.-> Config
    Autoscaler -- "Scale Nodes" --> VPC
    BDeployment -.-> Metrics
    FDeployment -.-> Metrics
```

## How to View This Diagram

1. **GitHub/GitLab:** If you push this file to a GitHub or GitLab repository, it will automatically render the diagram.
2. **VS Code:** Install the **"Mermaid Previewer"** or **"Markdown Preview Mermaid Support"** extension. Then, open this file and press `Ctrl+Shift+V` to see the live preview.
3. **Mermaid Live Editor:** 
   - Copy the code block above (from `graph TD` to the end of the block).
   - Go to [Mermaid.live](https://mermaid.live/).
   - Paste the code into the editor on the left.
4. **Notion/Obsidian:** These tools support Mermaid diagrams natively. Simply paste the code block into a page.
