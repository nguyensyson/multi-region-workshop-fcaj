---
title: "Architecture Overview"
date: 2026-06-03
weight: 4
chapter: false
pre: " <b> 1.4 </b> "
---

# 1.4 Architecture Overview

![Architecture Overview](/images/architecture-overview.png)

The diagram above illustrates the **Multi-Region architecture** of the MoneyTrack application. The system runs simultaneously across two AWS Regions, providing high availability and low latency for global users.

---

### Main Traffic Flow (API)

Users access the application API at `api.moneytrack.com`. The request travels through the following layers:

1. **User** sends a request to `api.moneytrack.com`.
2. **Route 53** resolves the DNS and forwards the request to **AWS Global Accelerator**.
3. **Global Accelerator** identifies the nearest healthy AWS Region and routes requests to the corresponding **Application Load Balancer (ALB)**.
4. **AWS WAF**, attached to the ALB, inspects and filters the request — blocking threats such as SQL Injection, XSS, or requests from malicious IP addresses.
5. The ALB distributes requests across **ECS Fargate** containers running in the **Private Subnet**.
6. ECS Fargate connects to **DynamoDB Global Tables** via a **VPC Endpoint** (Gateway Endpoint) — traffic never traverses the public internet.
7. **DynamoDB Global Tables** synchronize data between Regions in real time, ensuring data consistency.

---

### Frontend

- **Users** access `www.moneytrack.com`, resolved by Route 53 to **AWS Amplify**.
- Amplify hosts the frontend application (React/Next.js) and distributes content globally via its built-in CDN — no additional infrastructure required.

---

### Internal Networking

Each Region has its own **VPC** with a two-tier subnet structure:

| Subnet | Contains | Purpose |
|--------|----------|---------|
| **Public Subnet** | ALB, Internet Gateway (IGW), NAT Gateway | Entry point for inbound traffic from the internet |
| **Private Subnet** | ECS Fargate containers | Runs application workloads; not exposed to the internet |

- **Interface Endpoints** allow ECS Fargate to communicate with AWS services (ECR, Secrets Manager, SSM, CloudWatch) without going through the internet.
- **Gateway Endpoints** allow ECS Fargate to connect to DynamoDB over AWS's private internal network.

---

### CI/CD Pipeline

- Every time a code change is pushed to **GitHub**, the CI/CD pipeline is automatically triggered.
- The pipeline builds a Docker image, pushes it to **Amazon ECR**, and then deploys the new version to ECS Fargate in **both Regions** simultaneously.

---

### Security

| Layer | Service | Role |
|-------|---------|------|
| **Network** | AWS WAF | Inspects and blocks malicious HTTP/HTTPS requests at the ALB |
| **Identity** | IAM | Enforces least-privilege permissions for each service (ECS Task Role, GitHub Actions Role) |
| **Secrets** | AWS Secrets Manager | Stores and automatically rotates secrets (API keys, database credentials) |
| **Config** | AWS Systems Manager (SSM) | Manages environment-specific configuration parameters |

---

### Monitoring & Observability

| Service | Role |
|---------|------|
| **Amazon CloudWatch** | Centralized log aggregation and metric collection; CloudWatch Alarms trigger on threshold breaches |
| **Amazon Managed Service for Prometheus** | Scrapes Prometheus-compatible metrics from the application layer |
| **Amazon Managed Grafana** | Real-time dashboards for visualizing system performance; integrated alerting |

{{% notice info %}}
In the following sections, you will be guided step-by-step through deploying each component of this architecture — from provisioning infrastructure with Terraform, to configuring the CI/CD pipeline and setting up system monitoring.
{{% /notice %}}
