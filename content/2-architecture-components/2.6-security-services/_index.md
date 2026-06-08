---
title: "Security services - WAF, IAM, Secrets Manager"
date: 2026-06-03
weight: 6
chapter: false
pre: " <b> 2.6 </b> "
---

# 2.6 Security services - WAF, IAM, Secrets Manager

### AWS WAF
AWS WAF (Web Application Firewall) helps protect your web applications or APIs against common web exploits and bots that may affect availability, compromise security, or consume excessive resources. In this architecture, WAF is associated with the Application Load Balancer to inspect incoming HTTP/HTTPS requests before they reach the backend application.

### IAM (Identity and Access Management)
AWS IAM enables you to manage access to AWS services and resources securely. It is used to manage permissions based on the principle of least privilege—granting only the permissions required to perform a task.

### AWS Secrets Manager
AWS Secrets Manager helps you protect secrets needed to access your applications, services, and IT resources. It enables you to easily rotate, manage, and retrieve database credentials, API keys, and other secrets throughout their lifecycle. We use it to securely store items like database access tokens or third-party API keys.

### Multi-Layered Security
These services work together to protect the system at multiple layers:
* **Perimeter Layer (WAF):** Blocks malicious traffic, SQL injection, and cross-site scripting attacks at the edge.
* **Identity Layer (IAM):** Ensures that only authorized entities (like specific ECS tasks or GitHub Actions) can interact with AWS resources.
* **Data Layer (Secrets Manager):** Prevents sensitive credentials from being hardcoded in source code or exposed in environment variables.

### Why Use These Services in Production?
A robust security posture is non-negotiable for production workloads. Relying solely on network boundaries (like VPCs) is insufficient. Applying defense-in-depth using WAF for application-layer threats, strict IAM roles for access control, and Secrets Manager for credential management minimizes the blast radius of any potential security incident.

### Deployment Considerations
* **IAM Role for ECS Task:** Create distinct IAM roles for your ECS tasks (Task Role for the application's permissions, Task Execution Role for the ECS agent to pull images and write logs). Ensure they only have access to the specific resources they need.
* **Secret Rotation:** Configure automatic rotation for secrets in Secrets Manager where applicable to enhance security.
* **WAF Rules & Managed Rule Groups:** Start with AWS Managed Rules (like the Core rule set) for WAF to immediately protect against common vulnerabilities.
* **Logging:** Enable WAF logging to monitor denied requests and refine your security rules over time.
