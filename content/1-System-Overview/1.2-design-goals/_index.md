---
title: "Design Goals"
date: 2026-06-03
weight: 2
chapter: false
pre: " <b> 1.2 </b> "
---

# 1.2 Design Goals

The system is designed around six core objectives:

| Goal | Description |
|------|-------------|
| **High Availability** | The system remains available even when one Region fails. Route 53 and Global Accelerator automatically shift traffic to the healthy Region. |
| **Scalability** | ECS Fargate automatically scales out/in based on real-time demand. DynamoDB Global Tables handle millions of requests without manual sharding. |
| **Security** | AWS WAF filters malicious requests. IAM enforces least-privilege access. Secrets Manager stores sensitive credentials instead of hardcoding them in source code or environment variables. |
| **Disaster Recovery** | DynamoDB Global Tables replicate data across Regions in real time. When one Region fails, the other continues serving with complete, up-to-date data. |
| **Automation (CI/CD & IaC)** | All infrastructure is defined as code using **Terraform**. A GitHub Actions CI/CD pipeline automatically builds, pushes images to ECR, and deploys to ECS on every code change. |
| **Observability** | CloudWatch collects logs and metrics from ECS, ALB, and AWS services. Amazon Managed Service for Prometheus and Amazon Managed Grafana provide visual dashboards for monitoring system health and configuring alerts. |

### Goal Details

#### High Availability
A highly available system tolerates failures in individual components or even entire Regions without impacting end users. In this architecture, **Route 53** performs health checks on each Region's endpoint. If a Region becomes unhealthy, **AWS Global Accelerator** automatically reroutes all traffic to the remaining healthy Region — with no manual intervention required.

#### Scalability
**ECS Fargate** removes the need to manage EC2 instances. Tasks scale horizontally based on CPU and memory utilization metrics via Application Auto Scaling. **DynamoDB Global Tables** provides a fully managed, serverless database that scales on-demand without pre-provisioning capacity.

#### Security
Security is applied at multiple layers:
- **Network layer**: AWS WAF inspects every inbound HTTP/HTTPS request to the ALB.
- **Identity layer**: IAM roles with least-privilege policies control what each service can access.
- **Data layer**: AWS Secrets Manager stores and automatically rotates secrets; SSM Parameter Store manages non-sensitive configuration.

#### Disaster Recovery
The system follows an **Active-Active** multi-region pattern. Both Regions serve live traffic simultaneously. **DynamoDB Global Tables** uses multi-active replication with sub-second latency, ensuring data is consistent across Regions at all times.

#### Automation (CI/CD & IaC)
- **Terraform** defines all AWS resources in code, enabling repeatable, auditable deployments.
- **GitHub Actions** automates the full pipeline: build Docker image → push to ECR → update ECS service — triggered on every merge to the main branch.

#### Observability
- **CloudWatch** provides centralized log aggregation and metric collection across both Regions.
- **Amazon Managed Prometheus** and **Amazon Managed Grafana** offer application-level metrics and dashboards.
- Alerts notify the on-call team immediately when latency, error rate, or resource utilization exceeds defined thresholds.
