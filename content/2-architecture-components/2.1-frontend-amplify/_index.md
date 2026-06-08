---
title: "Frontend - AWS Amplify"
date: 2026-06-03
weight: 1
chapter: false
pre: " <b> 2.1 </b> "
---

# 2.1 Frontend - AWS Amplify

### What is AWS Amplify?
AWS Amplify is a complete solution that lets frontend web and mobile developers easily build, ship, and host full-stack applications on AWS. In this architecture, we use **Amplify Hosting**, a fully managed CI/CD and hosting service for fast, secure, and reliable delivery of static and server-side rendered apps.

### Role in the Frontend System
Amplify serves as the delivery mechanism for our React/Next.js frontend application. It automatically builds the application from our source repository and hosts the resulting static assets (HTML, CSS, JavaScript, images) on a globally distributed Content Delivery Network (CDN).

### How Users Access the Frontend
When a user navigates to the application's domain (e.g., `www.moneytrack.com`), the DNS request is resolved to the nearest CloudFront edge location managed by Amplify, ensuring fast load times regardless of the user's geographic location.

### Why Choose AWS Amplify?
* **Managed Hosting:** No servers to provision, patch, or maintain.
* **Integrated CI/CD:** Automatically triggers builds and deployments when code is pushed to connected branches.
* **Global CDN:** Built on Amazon CloudFront for low-latency content delivery worldwide.
* **Custom Domain Support:** Easily map custom domains and automatically provision free SSL/TLS certificates.

### Role in Multi-Region Architecture
While backend services are explicitly deployed in multiple regions for high availability, Amplify Hosting provides inherent multi-region benefits through its global CDN. It serves frontend assets from edge locations closest to the user, providing a resilient and globally accessible entry point to the application.

### Deployment Considerations
* **Environment Variables:** Securely manage configuration values (like API endpoints) for different environments (dev, staging, prod).
* **Branch-based Deployment:** Map Git branches to specific Amplify environments to easily test new features before releasing to production.
* **Build Settings:** Configure the `amplify.yml` file to specify build commands and the output directory for your specific framework.
* **Domain Mapping:** Ensure DNS records in Route 53 are correctly pointed to the Amplify app.
