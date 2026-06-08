---
title: "Disaster Recovery"
date: 2026-06-03
weight: 9
chapter: false
pre: " <b> 9. </b> "
---

# 9. Disaster Recovery

This section describes the Disaster Recovery (DR) strategy for the MoneyTrack project, detailing how the system reacts to failures and how we restore service.

## 9.1 Scenario: Single Availability Zone Failure

**Description:**
*   An entire Availability Zone (AZ) within a region experiences an outage.
*   ECS Fargate tasks or subnets residing in that specific AZ may become unreachable or fail.
*   The Application Load Balancer (ALB) health checks detect that the targets in the affected AZ are unhealthy.
*   The ALB automatically stops routing traffic to the failing tasks and shifts all incoming requests to the healthy tasks in the remaining AZs.
*   The ECS Service will attempt to provision replacement tasks in the healthy AZs if there is sufficient underlying capacity.
*   DynamoDB, being a fully managed regional service, continues to serve requests seamlessly.
*   *Note:* If NAT Gateways are only deployed in a single AZ to save costs, they become a single point of failure. In a production environment, NAT Gateways should be deployed in every AZ.

**Failure Flow:**
```txt
AZ A fails → ALB health check detects unhealthy targets → Traffic shifts to ECS tasks in AZ B → System continues serving requests
```

**Outcome:**
*   Users may experience minimal to no disruption if the system has adequately provisioned capacity in the surviving AZs.
*   This scenario is handled almost entirely automatically by our Multi-AZ architectural design.

---

## 9.2 Scenario: Full Region Failure

**Description:**
*   A major, region-wide outage occurs, affecting the ALB, ECS Fargate, the VPC, or other core services in the primary region.
*   AWS Global Accelerator and Route 53 health checks detect that the endpoint group for the failed region is completely unhealthy.
*   New incoming traffic is automatically rerouted by Global Accelerator to the surviving region.
*   The pre-provisioned backend infrastructure (ECS tasks) in the backup region takes over request processing.
*   DynamoDB Global Tables ensures that the backup region already has the replicated data necessary to serve users.
*   *Note:* For a smooth recovery, auxiliary components like Secrets Manager secrets, ECR images, and Terraform configurations must be pre-synchronized to the backup region.

**Failure Flow:**
```txt
Region A fails → Global Accelerator marks endpoint unhealthy → Traffic shifts to Region B → ALB Region B → ECS Fargate Region B → DynamoDB Global Tables
```

**Outcome:**
*   The system continues to serve users from the healthy region.
*   In-flight requests being processed in the failed region during the exact moment of failure might be dropped, requiring a client-side retry.
*   Data might experience a tiny replication latency, but DynamoDB Global Tables typically keeps this under a second.

---

## 9.3 Expected RTO and RPO

*   **RTO (Recovery Time Objective):** The maximum acceptable time the system can be down before service is restored.
*   **RPO (Recovery Point Objective):** The maximum acceptable amount of data loss measured in time.

| Scenario | Expected RTO | Expected RPO | Explanation |
| :--- | :--- | :--- | :--- |
| **Single ECS Task Failure** | Seconds to a few minutes | Near Zero | ECS replaces the task; ALB only routes to healthy targets. |
| **Single AZ Failure** | A few minutes | Near Zero | ALB shifts traffic to the remaining AZ(s) in the same region. |
| **Full Region Failure** | A few minutes | Seconds to a few minutes | Global Accelerator routes traffic to the healthy region; DynamoDB Global Tables provides the replicated data. |

*Disclaimer: These RTO/RPO values are target design objectives, not absolute guarantees. Actual recovery times depend on health check intervals, DNS caching, application readiness, and the nature of the outage.*

---

## 9.4 Failover Strategy

Failover strategies generally fall into two categories:

*   **Active-Active:** Both regions serve traffic simultaneously. Global Accelerator routes users to the optimal region. If one fails, the other absorbs all traffic.
    *   *Pros:* Lowest RTO, fully utilizes all provisioned resources.
    *   *Cons:* Higher cost, requires careful handling of data consistency and conflict resolution.
*   **Active-Passive:** One primary region handles all traffic while the backup region remains on standby.
    *   *Pros:* Easier to manage, potentially lower cost if the passive region is scaled down.
    *   *Cons:* Higher RTO as the passive region may need time to scale up to handle the full production load.

**MoneyTrack's Approach:**
The MoneyTrack architecture provisions backend and database resources in both regions, supported by Global Accelerator and DynamoDB Global Tables. This setup can act as Active-Active or Active-Passive depending on how traffic weights are configured. To ensure readiness, failover procedures should be periodically tested via simulated outages.

**Failover Flow:**
```txt
Health check failed → Endpoint marked unhealthy → Stop routing to failed region → Route traffic to healthy region → Monitor recovery
```

---

## 9.5 System Recovery Procedure

When a disaster strikes, follow this general procedure:

### Step 1: Incident Detection
*   Anomalies are detected via CloudWatch Alarms, ALB health checks, ECS events, WAF/Global Accelerator metrics, or Grafana dashboards (e.g., spike in HTTP 5xx errors, ALB reporting unhealthy targets, region endpoint failing).

### Step 2: Determine Impact Scope
*   Investigate whether the issue is isolated to the application code, a specific ECS task, the ALB, a network component, the database, a single AZ, or the entire region.
*   Compare metrics between the two regions to confirm which one remains healthy.

### Step 3: Trigger Failover (if necessary)
*   **AZ Level:** Allow the ALB and ECS to handle the traffic shift automatically.
*   **Region Level:** Verify that Global Accelerator has successfully shifted traffic to the healthy region.
*   If operating in an Active-Passive mode with scaled-down capacity, manually or automatically increase the `desired_count` of ECS tasks in the backup region.

### Step 4: Remediate the Failing Resources
*   If the issue is application-related, rollback the Docker image or redeploy the ECS service.
*   If it's a network issue, inspect Security Groups, route tables, NAT Gateways, and VPC Endpoints.
*   If it's a data issue, check the DynamoDB Global Tables replication status.
*   If infrastructure was accidentally deleted or modified, use Terraform to re-apply the correct state.

### Step 5: Verify System Health
*   Test the API health check endpoints.
*   Verify that the ALB target groups show healthy instances.
*   Confirm ECS tasks are running stably.
*   Review application logs in CloudWatch.
*   Check the Grafana dashboards for normal metrics.
*   Validate data accessibility in DynamoDB.

### Step 6: Failback (When resolved)
*   Once the failed region is completely stable, *do not* instantly shift all traffic back.
*   Thoroughly verify backend health, database replication, and metrics in the recovered region.
*   Gradually shift traffic back (e.g., using Global Accelerator traffic dials) while monitoring closely.

### Step 7: Post-Incident Review (PIR)
*   Document the root cause of the incident.
*   Evaluate the actual RTO and RPO against the design objectives.
*   Update runbooks, alert thresholds, or Terraform configurations based on lessons learned.
*   Create new test cases to prevent similar outages in the future.

---

## Conclusion

The Disaster Recovery plan for MoneyTrack relies on a combination of Multi-AZ deployments, Multi-Region architecture, Global Accelerator, ALB health checks, ECS Fargate resilience, and DynamoDB Global Tables.
*   AZ failures trigger automatic, localized traffic shifting.
*   Region failures trigger global traffic routing to a standby region.
*   A successful recovery requires robust monitoring, predefined failover steps, data validation, and continuous post-incident learning.
