---
title: "Monitoring / Observability"
date: 2026-06-03
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# 5. Monitoring / Observability

The primary goal of monitoring and observability in the MoneyTrack project is to detect errors early, track system performance, and facilitate efficient troubleshooting. By having a clear view of our multi-region architecture, we can ensure a seamless experience for our users.

### Monitoring the CI/CD Pipeline

Tracking deployment pipelines helps identify issues before they reach production.

*   **Frontend:** We monitor build and deployment logs directly within the **AWS Amplify** console.
*   **Backend:** We track pipeline logs during the stages of building the Docker image, pushing it to Amazon ECR, and deploying it to ECS Fargate.

**Common Pipeline Errors:**
*   Frontend or backend build failures due to syntax errors or missing dependencies.
*   Failing unit or integration tests.
*   Docker build failures.
*   Failures when pushing images to ECR (often due to IAM permissions).
*   ECS deployment failures (e.g., tasks failing to start or failing ALB health checks).

![Pipeline Log](/images/pipeline-log.png)

### Tracking Application Logs

Application logs are crucial for understanding the internal state of our backend services.

*   Our backend containers running on **ECS Fargate** are configured to send their standard output and error streams directly to **Amazon CloudWatch Logs**.
*   These logs allow us to investigate incoming requests, track exceptions, identify application errors, and verify the status of business logic processing.

### Monitoring Key Metrics

We collect and monitor metrics across all layers of our architecture:

*   **ECS/Fargate:** CPU utilization, memory utilization, and the number of running tasks.
*   **Application Load Balancer (ALB):** Total request count, response latency, HTTP 4xx/5xx error rates, and the count of healthy vs. unhealthy targets.
*   **DynamoDB Global Tables:** Consumed read/write capacity units, throttled requests, request latency, and replication latency between regions.
*   **AWS WAF:** Count of allowed requests versus blocked requests (identifying potential attacks).

### Visualizing with Grafana Dashboards

To make sense of the collected data, we use dashboards for visualization.

*   **Amazon Managed Grafana** provides a centralized, secure platform to visualize our metrics.
*   We use **Amazon Managed Service for Prometheus** (where configured) to scrape detailed, application-level metrics from our backend services.
*   Our core dashboards include panels for request rates, error rates, latency percentiles, ECS CPU/memory usage, and DynamoDB performance metrics.

![Grafana Dashboard](/images/grafana-dashboard.png)

### Alerting Strategy

Proactive alerting ensures the team is notified immediately when something goes wrong. We configure alerts for critical thresholds, such as:

*   A sudden spike in HTTP 5xx errors at the ALB.
*   Unexpected ECS task terminations (tasks stopped).
*   Sustained high latency impacting user experience.
*   DynamoDB request throttling, indicating a need to adjust capacity.
*   ALB reporting unhealthy targets, suggesting backend application failures.
