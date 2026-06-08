---
title: "Database Access Flow"
date: 2026-06-03
weight: 3
chapter: false
pre: " <b> 3.3 </b> "
---

# 3.3 Database Access Flow

This flow illustrates how backend tasks securely interact with the database and handle multi-region replication.

### Flow Breakdown

```txt
ECS Fargate → IAM Task Role → VPC Endpoint → DynamoDB Global Tables → Replication to another Region → Response
```

1. **Initiate Request:** An **ECS Fargate** task processes an API request that requires reading from or writing to the database.
2. **Retrieve Secrets (Optional):** If the application requires external API keys or specific database credentials, it retrieves them securely from AWS Secrets Manager.
3. **Authorization:** The ECS task uses its assigned **IAM Task Role** to authenticate and authorize its request to DynamoDB, ensuring it has the correct permissions.
4. **Private Access:** The database request is routed through a **VPC Endpoint** (specifically, a Gateway Endpoint for DynamoDB). This keeps the traffic entirely within the AWS private network, avoiding the public internet.
5. **Data Storage & Replication:** The request reaches **DynamoDB Global Tables**. The data is written to or read from the local region's table. If it's a write operation, DynamoDB automatically replicates the changes to the other region asynchronously.
6. **Result:** The response from DynamoDB is returned to the ECS Fargate task, which then formats the final response to send back to the frontend/user.

### Key Concepts

* **Why use IAM Role instead of hard-coded credentials:** Hard-coding credentials poses a severe security risk. Using an IAM Task Role grants temporary, securely managed permissions directly to the ECS task, adhering to best security practices.
* **Why use VPC Endpoint to access DynamoDB:** Accessing DynamoDB via a VPC Endpoint ensures that traffic between your VPC and DynamoDB does not leave the Amazon network. This improves security and reduces data transfer costs.
* **How DynamoDB Global Tables support multi-region:** Global Tables provide a fully managed, multi-active database. Writes performed in one region are automatically propagated to replicas in other regions, ensuring high availability and local read/write performance globally.
* **Deployment Considerations:** When using Global Tables, consider *eventual consistency* (replicas might have a slight delay). Understand the *conflict handling* mechanism (typically "last writer wins"). Properly design the *partition key* to ensure even data distribution. Ensure data is protected with *encryption at rest* and point-in-time *backup*.
