---
title: "System Scope"
date: 2026-06-03
weight: 3
chapter: false
pre: " <b> 1.3 </b> "
---

# 1.3 System Scope

### In Scope

This workshop covers the following components end-to-end:

| Component | Description |
|-----------|-------------|
| **Frontend** | Web application hosted on **AWS Amplify**, distributing static content globally via a built-in CDN. |
| **Backend** | Containerized API running on **ECS Fargate** within a Private Subnet, not directly exposed to the internet. |
| **Database** | **DynamoDB Global Tables** replicating data between two Regions in real time. |
| **Networking** | VPC with Public/Private Subnets, NAT Gateway, Internet Gateway, VPC Endpoints (Interface & Gateway). |
| **Routing & Traffic Management** | **Route 53** for DNS resolution; **AWS Global Accelerator** for intelligent routing to the lowest-latency Region. |
| **Security** | **AWS WAF** at the ALB layer; **IAM** for least-privilege service permissions; **AWS Secrets Manager** and **AWS Systems Manager** for secrets and configuration management. |
| **Container Registry** | **Amazon ECR** for storing and versioning backend Docker images. |
| **CI/CD** | **GitHub Actions** automating the full build and deployment pipeline. |
| **Infrastructure as Code** | **Terraform** managing all AWS infrastructure declaratively. |
| **Monitoring & Observability** | **CloudWatch**, **Amazon Managed Service for Prometheus**, **Amazon Managed Grafana**. |

### Out of Scope

To keep the focus on architecture and operations, this workshop does **not** cover:

- **Application source code**: The workshop provides a pre-built sample application. Detailed implementation of business logic is not covered.
- **Advanced cost optimization**: Strategies such as Reserved Instances, Savings Plans, Spot Instances for Fargate, or Cost Anomaly Detection are outside the scope of this workshop.
- **Software testing**: Unit tests, integration tests, and detailed load testing methodologies are not covered.

{{% notice tip %}}
If you are interested in cost optimization after completing this workshop, refer to the [AWS Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html) of the AWS Well-Architected Framework.
{{% /notice %}}
