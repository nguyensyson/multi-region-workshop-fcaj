---
title: "Region Failover Flow"
date: 2026-06-03
weight: 4
chapter: false
pre: " <b> 3.4 </b> "
---

# 3.4 Region Failover Flow

This flow explains how the system handles a regional outage to maintain high availability.

### Normal Operation Flow

Under normal conditions, traffic is routed to the closest or preferred region.

```txt
User → Global Accelerator → Health Check → Healthy Region → ALB → ECS Fargate → DynamoDB Global Tables
```

1. **Routing:** Traffic is routed by **Global Accelerator** to the endpoint group in the primary (or closest) region.
2. **Health Monitoring:** Global Accelerator and Route 53 health checks continuously monitor the status of the endpoints (ALBs) in each region.

### Failover Operation Flow

When a region becomes unavailable or unhealthy, the following happens:

```txt
Region A unhealthy → Global Accelerator stops routing to Region A → Traffic routed to Region B → Application continues serving users
```

1. **Failure Detection:** If Region A goes down, the health checks associated with Region A's endpoint group fail.
2. **Traffic Rerouting:** **Global Accelerator** marks Region A as unhealthy and automatically reroutes all new incoming traffic to the healthy Region B.
3. **Continuous Operation:** The backend (ECS Fargate tasks) in the backup region (Region B) receives the requests and continues to process them.
4. **Data Availability:** Because **DynamoDB Global Tables** has already been replicating data in the background, the application in Region B has access to the most up-to-date data and can serve users without interruption.
5. **Failback (Recovery):** Once the issue in Region A is resolved and health checks pass again, Global Accelerator can gradually restore traffic back to Region A according to your failback strategy.

### Key Concepts

* **Failover occurs at the routing layer:** The failover is managed seamlessly by Global Accelerator at the network routing level, meaning no manual DNS changes are required, and the failover is incredibly fast.
* **Independent Deployments:** The backend infrastructure in each region must be deployed and scaled independently to ensure it can handle the full load if a failover occurs.
* **Data in Multiple Regions:** DynamoDB Global Tables is the critical piece that ensures stateful data is present in both regions, allowing the application to function regardless of which region is serving the request.
* **RTO and RPO:** While we won't dive deep here, this architecture aims for a very low Recovery Time Objective (RTO - how fast you recover) and Recovery Point Objective (RPO - how much data you might lose), providing near-continuous availability.
* **Active-Active vs. Active-Passive:** This architecture supports an Active-Active setup (both regions serve traffic simultaneously) but can also act as Active-Passive if traffic is weighted entirely to one region until a failure occurs.
