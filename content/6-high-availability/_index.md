---
title: "High Availability"
date: 2026-06-03
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

# 6. High Availability

High Availability (HA) in the context of the MoneyTrack project means ensuring the application remains accessible and operational even if individual components, entire Availability Zones (AZs), or a full AWS Region experiences an outage.

### Frontend HA

*   The frontend is hosted on **AWS Amplify**, a fully managed static web hosting service backed by a global Content Delivery Network (CDN).
*   By decoupling the frontend from the backend, the static assets are highly resilient and can be served to users independently of backend compute status.

### Backend HA

*   Our backend API runs on **ECS Fargate** within private subnets. Fargate manages the underlying servers, abstracting away host-level failures.
*   The **ECS Service** is configured to maintain a desired number of tasks. If a task crashes, ECS automatically replaces it.
*   The **Application Load Balancer (ALB)** continuously performs health checks on the Fargate tasks. It routes incoming traffic *only* to healthy tasks, ensuring users are not sent to failing containers.
*   Within a single region, Fargate tasks are distributed across multiple Availability Zones, protecting the backend from single-AZ failures.

### Database HA

*   **Amazon DynamoDB** is inherently highly available within a single region, automatically replicating data synchronously across multiple AZs.
*   To achieve multi-region HA, we utilize **DynamoDB Global Tables**. This feature continuously and asynchronously replicates our data between our primary and secondary regions.
*   If one region goes completely offline, the database in the surviving region contains the up-to-date data required to continue serving the application.

### Networking HA

*   The **VPC** is designed with public subnets for the ALB and NAT Gateways, and private subnets for the ECS Fargate tasks.
*   This structure is replicated across multiple AZs. Redundant NAT Gateways (one per AZ) ensure that private tasks can always reach external services.
*   **VPC Endpoints** provide a highly available, direct connection to AWS services like DynamoDB and ECR, bypassing the public internet and potential internet routing issues.

### Multi-AZ and Multi-Region Resilience

*   **Multi-AZ:** Within each region, deploying across at least two AZs mitigates the risk of localized power, cooling, or network failures in a single AWS data center.
*   **Multi-Region:** At a global level, **Route 53** and **AWS Global Accelerator** intelligently route traffic. If an entire region becomes unhealthy, Global Accelerator rapidly redirects API traffic to the remaining healthy region. The backend and database in the backup region are pre-provisioned and ready to handle the load.

### High Availability Request Flow

```txt
User → Route 53 / Global Accelerator → Healthy Region → ALB → ECS Fargate → DynamoDB Global Tables
```

### Conclusion

The High Availability of the MoneyTrack project is not achieved by a single feature, but through a layered approach. By combining fully managed serverless services (Amplify, Fargate, DynamoDB), multi-AZ network design, multi-region database replication, and intelligent global traffic routing with health checks, the architecture is designed to withstand significant failures and maintain continuous service.
