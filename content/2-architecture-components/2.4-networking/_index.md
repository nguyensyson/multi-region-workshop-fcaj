---
title: "Networking - VPC, Subnet, IGW, NAT Gateway, VPC Endpoint"
date: 2026-06-03
weight: 4
chapter: false
pre: " <b> 2.4 </b> "
---

# 2.4 Networking - VPC, Subnet, IGW, NAT Gateway, VPC Endpoint

### Role of VPC in the Architecture
An Amazon Virtual Private Cloud (VPC) provides an isolated virtual network for your AWS resources. In this architecture, each region has its own VPC acting as the secure foundation. It controls the network boundaries, traffic flow, and IP addressing for all components deployed within that region.

### Public and Private Subnets
Subnets divide the VPC's IP address range.
* **Public Subnets:** Contain resources that need to be accessible from the internet, such as the Application Load Balancer (ALB) and NAT Gateways. These subnets have a route to the Internet Gateway.
* **Private Subnets:** House our core application logic (ECS Fargate tasks) and databases. They have no direct route to the internet, providing a strong security boundary against direct external access.

### Internet Gateway (IGW)
The Internet Gateway is a horizontally scaled, redundant, and highly available VPC component that allows communication between instances in your VPC and the internet. In our architecture, it enables traffic from the internet to reach the ALB in the public subnets.

### NAT Gateway
The NAT (Network Address Translation) Gateway allows resources in a private subnet (like our ECS tasks) to connect to the internet (e.g., to download Docker images from ECR or patches) while preventing the internet from initiating connections to those instances. It resides in the public subnet and routes traffic from the private subnet to the IGW.

### VPC Endpoints
VPC Endpoints enable private connections between your VPC and supported AWS services without requiring an Internet Gateway, NAT device, VPN connection, or AWS Direct Connect connection.
* They optimize the network path, keeping traffic on the AWS global network.
* In this architecture, we use them for services like DynamoDB (Gateway Endpoint), ECR, Secrets Manager, and CloudWatch (Interface Endpoints) to ensure our backend services can securely communicate with AWS APIs without traversing the public internet.

### Why Design the Network Across Multiple AZs?
Deploying resources across multiple Availability Zones (AZs) within a region provides fault tolerance and high availability. If one AZ experiences an outage (e.g., due to power or network issues), the ALB can route traffic to healthy ECS tasks running in the other AZs, ensuring uninterrupted service.

### Deployment Considerations
* **CIDR Planning:** Ensure your VPC and subnet CIDR blocks are large enough to accommodate future growth and do not overlap with other networks you might connect to (e.g., via peering or VPN).
* **Route Tables:** Carefully manage route tables to ensure public subnets route internet traffic to the IGW and private subnets route outbound traffic to the NAT Gateway or VPC Endpoints.
* **Security Groups and NACLs:** Use Security Groups as stateful firewalls at the instance/task level and Network ACLs (NACLs) as stateless firewalls at the subnet level to enforce strict security boundaries.
* **NAT Gateway Costs:** Be aware that NAT Gateways incur an hourly charge and a data processing fee. Consider using VPC Endpoints where possible to reduce data processing costs.
* **Endpoint Policies:** Attach policies to VPC Endpoints to restrict access to only specific API actions or specific AWS resources (e.g., only allow access to a specific DynamoDB table).
