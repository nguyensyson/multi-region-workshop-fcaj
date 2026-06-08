---
title: "Problem Statement"
date: 2026-06-03
weight: 1
chapter: false
pre: " <b> 1.1 </b> "
---

# 1.1 Problem Statement

Modern web applications face increasingly demanding requirements around **availability**, **response time**, and **scalability**. Users from different geographic regions — Southeast Asia, Europe, North America — all expect consistent performance regardless of where they are.

In this workshop, we build the backend infrastructure for **MoneyTrack** — a personal finance management application serving a global user base. The system must ensure:

- Users calling the API from any country receive fast, reliable responses.
- Data is not lost when a failure occurs in one region.
- The system automatically recovers without manual intervention.

### The Problem with Single-Region Architecture

When an entire system is deployed in only one AWS Region, the following risks arise:

| Risk | Description |
|------|-------------|
| **Downtime** | If the Region experiences an outage (natural disaster, infrastructure failure), all services become unavailable. |
| **High Latency** | Users located far from the Region experience increased network latency, degrading the user experience. |
| **Single Point of Failure** | All components concentrated in one location create a *Single Point of Failure* (SPOF). |
| **No Failover** | There is no automatic mechanism to redirect traffic to a backup region during an incident. |

To address these issues, this workshop guides you through building a **Multi-Region architecture** on AWS — where the system is deployed in parallel across multiple Regions, ensuring high availability and consistent performance.
