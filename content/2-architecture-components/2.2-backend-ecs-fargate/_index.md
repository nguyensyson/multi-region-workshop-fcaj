---
title: "Backend - ECS Fargate"
date: 2026-06-03
weight: 2
chapter: false
pre: " <b> 2.2 </b> "
---

# 2.2 Backend - ECS Fargate

### What is ECS Fargate?
Amazon Elastic Container Service (ECS) is a fully managed container orchestration service. **Fargate** is a serverless compute engine for containers that works with ECS. It allows you to run containers without having to manage servers or clusters of Amazon EC2 instances.

### Role in the Backend System
ECS Fargate hosts our containerized backend API application. It pulls the Docker image from Amazon ECR and runs it as a task, processing incoming API requests and interacting with the DynamoDB database.

### How Backend Receives Requests
Traffic is routed from the Application Load Balancer (ALB) to the ECS Fargate tasks. The ALB serves as a single point of contact, distributing incoming application traffic across multiple targets (our ECS tasks) in multiple Availability Zones.

### Running in a Private Subnet
For security, ECS Fargate tasks are deployed in **private subnets**. They do not have public IP addresses and cannot be accessed directly from the internet. They receive traffic only from the ALB (which resides in the public subnet) and access external services (like pulling images from ECR) via a NAT Gateway or VPC Endpoints.

### Why Choose ECS Fargate?
* **Serverless Compute:** Focus on building applications, not managing infrastructure. No EC2 instances to patch, scale, or secure.
* **Flexible Scaling:** Easily scale up or down based on CPU or memory usage using ECS Service Auto Scaling.
* **Security:** Tasks run in their own isolated compute environment, improving security.
* **Cost-Effective:** Pay only for the vCPU and memory resources consumed by your application.

### Deployment Considerations
* **Task Definition:** Specifies the Docker image, CPU, memory, environment variables, and logging configuration for your application.
* **Service:** Manages a specified number (desired count) of task definition instances and registers them with the ALB.
* **Desired Count & Auto Scaling:** Set a baseline number of tasks and configure auto-scaling rules to handle traffic spikes.
* **Health Check:** Configure the ALB health check path so it can accurately determine if a task is healthy before sending traffic to it.
* **IAM Task Role:** Grant the task specific permissions, such as the ability to read from Secrets Manager or write to DynamoDB, adhering to least privilege.
* **Security Group:** Configure rules to allow inbound traffic only from the ALB's security group.
