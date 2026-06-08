---
title: "Traffic routing - Route 53, Global Accelerator, ALB"
date: 2026-06-03
weight: 5
chapter: false
pre: " <b> 2.5 </b> "
---

# 2.5 Traffic routing - Route 53, Global Accelerator, ALB

### Route 53
Amazon Route 53 is a highly available and scalable cloud Domain Name System (DNS) web service. It is used to manage DNS records and route users to Internet applications. In our architecture, it acts as the initial entry point, translating our human-readable domain name (like `api.moneytrack.com`) into the IP addresses of our AWS Global Accelerator.

### AWS Global Accelerator
AWS Global Accelerator is a networking service that improves the availability and performance of applications with global users. It provides static IP addresses that act as a fixed entry point to your application endpoints in a single or multiple AWS Regions.
* It routes traffic over the AWS global network to the optimal region based on geographic proximity and health.

### Application Load Balancer (ALB)
The Application Load Balancer operates at the request level (Layer 7), routing traffic to targets (ECS Fargate tasks) within an Amazon VPC based on the content of the request. It distributes incoming application requests across multiple targets in multiple Availability Zones.

### Overall Traffic Flow
1. **User** initiates a request to `api.moneytrack.com`.
2. **Route 53** resolves the domain to the static IP addresses provided by **Global Accelerator**.
3. **Global Accelerator** directs the traffic to the optimal region's endpoint group (our ALB).
4. The **ALB** receives the request and distributes it to a healthy **ECS Fargate** task running our backend API.

### The Role of Health Checks in Failover
Health checks are critical for high availability.
* The **ALB** continuously checks the health of the ECS tasks. If a task fails, the ALB stops routing traffic to it.
* **Global Accelerator** continuously checks the health of its endpoint groups (the ALBs in each region). If an entire region goes down or becomes unhealthy, Global Accelerator instantly fails over and routes all traffic to the healthy region.

### Why Combine Route 53, Global Accelerator, and ALB?
* **Route 53** provides reliable DNS resolution.
* **Global Accelerator** provides deterministic routing over the AWS network, improving performance for global users and offering rapid, IP-agnostic failover across regions.
* **ALB** provides granular, layer 7 load balancing and health checking within a specific region.
Combining them provides a robust, high-performance, and highly available routing architecture.

### Deployment Considerations
* **DNS Records:** Configure Alias records in Route 53 to point to the Global Accelerator DNS name.
* **Listeners and Target Groups:** Configure ALB listeners (e.g., port 443 for HTTPS) and target groups that point to the ECS service.
* **Health Checks:** Configure accurate and responsive health check paths on the ALB so it can quickly identify failing tasks.
* **SSL/TLS Certificates:** Use AWS Certificate Manager (ACM) to provision and manage SSL/TLS certificates for the ALB to enable secure HTTPS communication.
