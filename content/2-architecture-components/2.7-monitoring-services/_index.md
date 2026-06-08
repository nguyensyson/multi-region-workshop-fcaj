---
title: "Monitoring services - CloudWatch, Prometheus, Grafana"
date: 2026-06-03
weight: 7
chapter: false
pre: " <b> 2.7 </b> "
---

# 2.7 Monitoring services - CloudWatch, Prometheus, Grafana

### CloudWatch
Amazon CloudWatch is a monitoring and observability service. In our architecture, it is primarily used to collect logs (from ECS tasks and ALB), gather standard AWS infrastructure metrics, and set up alarms to notify operators when specific thresholds are breached.

### Prometheus (Amazon Managed Service for Prometheus)
Prometheus is an open-source systems monitoring and alerting toolkit. Amazon Managed Service for Prometheus provides a highly available, secure, and managed environment for Prometheus. We use it to scrape detailed, custom application-level metrics directly from our backend services.

### Grafana (Amazon Managed Grafana)
Grafana is an open-source analytics and interactive visualization web application. Amazon Managed Grafana is a fully managed and secure data visualization service. We use it to create comprehensive dashboards that visualize metrics from both CloudWatch and Prometheus in a single pane of glass.

![Grafana Dashboard](/images/grafana-dashboard.png)

### The Role of Monitoring in Operations
Effective monitoring is crucial for operating a multi-region system. It provides the visibility needed to:
* **Detect Errors:** Quickly identify failing requests or crashing application containers.
* **Analyze Performance:** Understand latency bottlenecks and resource utilization trends.
* **Operate the System:** Make informed decisions about scaling, capacity planning, and incident response.

### Examples of Metrics to Monitor
* **Infrastructure:** CPU and Memory utilization of ECS tasks.
* **Application/Traffic:** Request count, error rate (HTTP 4xx/5xx), and latency (response time).
* **Health:** ECS task health status and restart counts.
* **Database:** DynamoDB read/write capacity consumed, throttling events, and replication latency.

### Deployment Considerations
* **Log Retention:** Configure log retention policies in CloudWatch Logs to balance troubleshooting needs with storage costs. Don't keep logs indefinitely unless required by compliance.
* **Dashboards:** Design Grafana dashboards that present a clear, high-level overview of system health before diving into granular metrics.
* **Alerting:** Set up meaningful alerts (e.g., via CloudWatch Alarms or Grafana Alerting) that notify the team of actionable issues, avoiding "alert fatigue" from noisy, non-critical notifications.
* **Metric Naming:** Adopt a consistent metric naming convention in your application code for Prometheus scraping to make dashboard creation and querying easier.
* **Centralized Logging:** Ensure logs from all regions are easily accessible and searchable, potentially using CloudWatch Logs Insights or a centralized logging account.
