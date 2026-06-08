---
title: "Backend Request Flow"
date: 2026-06-03
weight: 2
chapter: false
pre: " <b> 3.2 </b> "
---

# 3.2 Backend Request Flow

This flow details how dynamic API requests are routed and processed by the backend services.

### Flow Breakdown

```txt
User → api.moneytrack.com → Route 53 → Global Accelerator → AWS WAF → ALB → ECS Fargate → Response
```

1. **User Action:** The user performs an action on the frontend (e.g., logging in or viewing transactions), causing the application to send an HTTP request to `api.moneytrack.com`.
2. **DNS Resolution:** **Route 53** resolves the API domain name to the anycast IP addresses provided by AWS Global Accelerator.
3. **Global Routing:** **Global Accelerator** receives the traffic and routes it over the AWS global network to the optimal region (typically the closest one) or the region that is currently healthy.
4. **Security Inspection:** Before reaching the load balancer, the request passes through **AWS WAF**. WAF inspects the request against security rules (e.g., checking for SQL injection or malicious IPs).
5. **Load Balancing:** If the request is valid, WAF allows it through to the Application Load Balancer (**ALB**) in the public subnet.
6. **Processing:** The ALB forwards the request to an available **ECS Fargate** task running in the private subnet. The Fargate task executes the application's business logic.
7. **Result:** ECS Fargate generates a response and sends it back through the ALB, Global Accelerator, and finally to the user.

### Key Concepts

* **Role of Route 53:** Route 53 acts as the DNS service, directing users trying to reach `api.moneytrack.com` to the correct entry point (Global Accelerator).
* **Role of Global Accelerator:** It improves performance by routing traffic over the AWS private network instead of the public internet, and it provides fast failover between regions.
* **Role of WAF:** WAF acts as a shield, filtering out malicious requests before they consume compute resources or compromise the application.
* **Role of ALB:** The ALB distributes incoming requests evenly across multiple ECS tasks within a region, ensuring no single task is overwhelmed.
* **Why ECS Fargate runs in a private subnet:** Running backend tasks in a private subnet ensures they are not directly accessible from the internet, significantly reducing the attack surface. They only accept traffic from the ALB.
