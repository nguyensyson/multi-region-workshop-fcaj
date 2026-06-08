---
title: "Security"
date: 2026-06-03
weight: 8
chapter: false
pre: " <b> 8. </b> "
---

# 8. Security

This section describes how the MoneyTrack project secures the system across multiple layers. A robust security posture is achieved by applying defense-in-depth principles across network, application, identity, and data layers.

## 8.1 Network Security

Network security limits the attack surface by tightly controlling how traffic flows into and within the AWS environment.

*   **VPC Isolation:** A dedicated VPC is designed for each region to provide an isolated network boundary.
*   **Subnet Segmentation:** The network is divided into public and private subnets.
*   **ALB Placement:** The Application Load Balancer (ALB) is placed in the public subnet to receive requests from the internet.
*   **Backend Isolation:** ECS Fargate tasks run the backend application strictly within the private subnet, meaning they are not directly exposed to the internet.
*   **Security Groups:** Security Groups act as stateful firewalls controlling traffic between the ALB, ECS Fargate, and other AWS services.
    *   The ALB only allows incoming HTTP/HTTPS traffic from the outside.
    *   ECS Fargate is configured to *only* accept traffic originating from the ALB's Security Group, enforcing the principle of least privilege.
*   **NAT Gateway:** Allows resources in the private subnet to access the internet when necessary (e.g., to pull packages or call external APIs), without allowing inbound connections from the internet.
*   **VPC Endpoint:** Enables ECS Fargate to access DynamoDB through a private network path, keeping database traffic off the public internet.
*   **NACLs:** Network Access Control Lists (NACLs) can be used as an additional stateless security layer at the subnet level.
*   **Multi-AZ:** Multi-AZ networking increases availability and reduces risk if one Availability Zone fails.

**Network Flow:**
```txt
Internet → ALB Public Subnet → ECS Fargate Private Subnet → VPC Endpoint → DynamoDB
```

**Key Takeaways:**
*   The backend is never publicly accessible.
*   The ALB is the sole entry point for backend API traffic.
*   Security Groups must strictly adhere to the principle of least privilege.

---

## 8.2 Application Security with AWS WAF

Application security protects the application logic from malicious payloads and attacks.

*   **AWS WAF Placement:** AWS WAF is attached to the ALB, inspecting incoming HTTP/HTTPS requests before they ever reach the backend application.
*   **Threat Mitigation:** WAF helps mitigate risks from malicious requests such as SQL injection, Cross-Site Scripting (XSS), bad bots, and anomalous traffic patterns.
*   **Managed Rules:** We leverage AWS Managed Rules to quickly apply baseline protection against common web vulnerabilities.
*   **Rate Limiting:** Rate-based rules are configured to limit the number of requests from a single IP address within a specific timeframe, protecting against brute-force or DDoS attempts.
*   **Visibility:** WAF logs can be sent to CloudWatch or S3 for detailed analysis. WAF metrics are integrated with CloudWatch/Grafana to monitor the volume of allowed versus blocked requests.
*   **Shared Responsibility:** WAF is a critical perimeter defense, but the application backend must still perform input validation and handle authentication/authorization in code. WAF does not replace secure coding practices.

**Application Security Flow:**
```txt
User → Route 53 / Global Accelerator → AWS WAF → ALB → ECS Fargate
```

**Key Takeaways:**
*   WAF is the first line of defense before the backend.
*   The ALB only processes requests that pass WAF inspection.
*   WAF reduces the load and attack risk on the application itself.

---

## 8.3 Data Encryption & Secrets Management

Data must be protected both when it is moving and when it is stored.

**Encryption in Transit:**
*   Users interact with both the frontend and backend exclusively via HTTPS.
*   The ALB uses SSL/TLS certificates (managed by ACM) to encrypt traffic from the client.
*   The API domain `api.moneytrack.com` enforces HTTPS.
*   Connections between AWS services and the AWS API are secured via HTTPS.

**Encryption at Rest:**
*   **DynamoDB:** Encrypts all data at rest by default. While Global Tables replicate data across regions, the data remains protected by encryption in all locations.
*   **Amazon ECR:** Stores Docker images securely. Image scanning and encryption should be enabled by default.
*   **CloudWatch Logs:** Application logs are stored securely, and appropriate retention policies should be configured.
*   **Terraform State:** The S3 bucket storing the Terraform state file must have encryption and versioning enabled, as the state file can contain sensitive information.

**Secrets Management:**
*   **AWS Secrets Manager:** Securely stores secrets, tokens, and sensitive credentials.
*   **No Hard-coding:** Secrets are never hard-coded in source code, Docker images, or Terraform configuration files.
*   **Secure Retrieval:** ECS Fargate tasks retrieve necessary secrets dynamically at runtime using permissions granted by their IAM Task Role.
*   **Rotation:** Secret rotation can be configured to periodically update credentials for enhanced security.

**Identity and Access Management (IAM):**
*   **IAM Roles:** ECS tasks use IAM Roles to access AWS services, eliminating the need for long-term access keys.
*   **CI/CD Pipeline:** The pipeline is granted only the permissions strictly necessary to build, push images, and deploy updates.
*   **Least Privilege:** All service roles strictly adhere to the principle of least privilege.
*   **Environment Separation:** IAM permissions are separated across environments (dev, staging, production) to prevent unintended cross-environment access.

**Identity Flow:**
```txt
ECS Fargate → IAM Task Role → Secrets Manager / DynamoDB / CloudWatch
```

---

## Security Checklist

This table summarizes how security is applied across different layers in the MoneyTrack architecture.

| Layer | Service | Application in MoneyTrack |
| :--- | :--- | :--- |
| **Network** | VPC, Subnets, Security Groups, VPC Endpoints | Dedicated VPC, backend hidden in private subnets, strict SG rules (ALB to ECS only), private routing to DynamoDB via Endpoints. |
| **Application** | AWS WAF, ALB, HTTPS | WAF filters malicious requests before ALB, HTTPS enforced for all client communication. |
| **IAM** | IAM Roles | IAM Task Roles for ECS, least privilege policies for all services and CI/CD pipelines. |
| **Secrets** | AWS Secrets Manager | Centralized, secure storage for API keys and tokens; dynamically retrieved by tasks. |
| **Data** | DynamoDB, S3, CloudWatch | DynamoDB encryption at rest, S3 state bucket encryption/versioning, appropriate log retention. |
| **Monitoring** | WAF Logs, CloudWatch, Grafana | Tracking blocked requests via WAF metrics, monitoring application logs for anomalous behavior. |

## Conclusion

The security of the MoneyTrack project is designed with a defense-in-depth approach.
*   **Network security** limits the attack surface by isolating the backend.
*   **AWS WAF** protects the application from malicious web traffic.
*   **IAM and Secrets Manager** ensure secure access control and sensitive data handling.
*   **Encryption** safeguards data both in transit and at rest.
*   **Monitoring** provides visibility to detect and respond to anomalous activities early.
