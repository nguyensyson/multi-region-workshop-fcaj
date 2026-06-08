---
title: "Database - DynamoDB Global Tables"
date: 2026-06-03
weight: 3
chapter: false
pre: " <b> 2.3 </b> "
---

# 2.3 Database - DynamoDB Global Tables

### What is DynamoDB?
Amazon DynamoDB is a fully managed, serverless, key-value NoSQL database designed to run high-performance applications at any scale. It offers built-in security, continuous backups, automated multi-region replication, in-memory caching, and data import/export tools.

### What are DynamoDB Global Tables?
**DynamoDB Global Tables** provide a fully managed solution for deploying a multi-region, multi-active database. They allow you to specify the AWS Regions where you want your table to be available, and DynamoDB automatically replicates ongoing data changes to all of them.

### Role in the System
DynamoDB serves as the primary data store for the MoneyTrack application, holding user profiles, transaction records, and account balances. Global Tables ensure that this data is consistently available and synchronized across our primary and secondary deployment regions.

### Data Replication Across Regions
When an ECS Fargate task in Region A writes data to its local DynamoDB replica, DynamoDB automatically propagates that write to the replica in Region B, typically within a second. This multi-active replication means applications can read and write data to their local replica, reducing latency.

### Why Choose DynamoDB Global Tables?
* **Low Latency:** Applications access data from the closest region, providing single-digit millisecond performance.
* **Multi-Region Replication:** Fully managed, active-active replication without needing to build or maintain custom replication logic.
* **High Availability & Disaster Recovery:** If one region becomes isolated or degraded, the application can seamlessly failover to another region and continue using its local, up-to-date replica.
* **Serverless:** No servers to provision or manage. The database automatically scales capacity based on traffic patterns.

### Deployment Considerations
* **Partition Key & Sort Key:** Careful schema design is crucial for NoSQL performance. Choose partition keys that distribute traffic evenly to avoid hot partitions.
* **Capacity Mode:** Choose between On-Demand (pay per request) or Provisioned capacity based on the predictability of your workload.
* **Conflict Handling:** Understand that Global Tables use a "last writer wins" reconciliation mechanism for concurrent updates to the same item across different regions.
* **Backup & Recovery:** Utilize Point-in-Time Recovery (PITR) to protect against accidental writes or deletes.
* **Encryption:** Enable encryption at rest using AWS KMS to secure sensitive financial data.
