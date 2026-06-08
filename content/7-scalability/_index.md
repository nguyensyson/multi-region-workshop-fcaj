---
title: "Scalability"
date: 2026-06-03
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

# 7. Scalability

Scalability in the MoneyTrack project refers to the system's ability to gracefully handle increasing amounts of traffic and data without degrading performance, and conversely, to scale down to save costs when demand is low.

### Frontend Scalability

*   The frontend is a static web application hosted on **AWS Amplify**.
*   Because Amplify leverages a global Content Delivery Network (CDN), it can serve massive spikes in traffic without requiring us to provision or manage any backend servers for the UI.
*   Combining this with a custom domain on Route 53 ensures fast, scalable access globally.

### Backend Scalability

*   Our compute layer is powered by **ECS Fargate**, which allows us to scale backend containers effortlessly.
*   We use **ECS Service Auto Scaling** to automatically increase or decrease the number of running Fargate tasks based on CloudWatch metrics like average CPU utilization, memory usage, or request count.
*   Crucially, the backend application is designed to be **stateless**. This means any task can handle any request, making it easy to scale out horizontally simply by adding more tasks.

### Load Balancing Scalability

*   The **Application Load Balancer (ALB)** automatically scales its underlying resources to handle varying levels of incoming traffic.
*   As ECS Auto Scaling adds new Fargate tasks, they register with the ALB's Target Group. The ALB seamlessly begins distributing incoming requests across the newly expanded pool of healthy targets.

### Database Scalability

*   **Amazon DynamoDB** is built for extreme scalability. It can handle massive workloads with consistent, single-digit millisecond latency.
*   We can configure DynamoDB with **On-Demand Capacity** (automatically adapting to unpredictable workloads) or **Provisioned Capacity with Auto Scaling** (automatically adjusting capacity limits based on utilization targets).
*   To ensure smooth scaling, we must design a good partition key to distribute read and write operations evenly across the table, avoiding "hot partitions." We monitor throttling, latency, and consumed capacity to adjust our scaling strategies.

### Multi-Region Scalability

*   **DynamoDB Global Tables** inherently scales data presence by replicating data across multiple regions, bringing data closer to global users and reducing read latency.
*   **Global Accelerator** can distribute traffic to the most appropriate region based on performance and user location.
*   As global traffic increases, we can independently scale the backend ECS tasks in each specific region to meet localized demand.

### Auto Scaling Flow

```txt
Traffic Increases → CloudWatch Metrics Trigger Alarm → ECS Auto Scaling kicks in → Adds more Fargate Tasks → ALB distributes requests to new tasks
```

### Conclusion

The MoneyTrack architecture is designed to scale independently at multiple layers. The frontend scales via CDN, the backend scales horizontally via ECS Fargate and ALB, the database scales elastically via DynamoDB, and the entire system scales geographically through multi-region deployment.
