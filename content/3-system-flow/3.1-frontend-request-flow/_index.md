---
title: "Frontend Request Flow"
date: 2026-06-03
weight: 1
chapter: false
pre: " <b> 3.1 </b> "
---

# 3.1 Frontend Request Flow

This flow describes how users load the web application's user interface.

### Flow Breakdown

```txt
User → www.moneytrack.com → AWS Amplify → Frontend UI → API call to api.moneytrack.com
```

1. **User Request:** The user opens their browser and navigates to the domain `www.moneytrack.com`.
2. **DNS Resolution:** Route 53 resolves the domain name, directing the user to the frontend hosted on **AWS Amplify**.
3. **Serving Static Assets:** Amplify serves the static assets of the frontend application (HTML, CSS, JavaScript) from its global CDN to the user's browser.
4. **API Interaction:** Once the frontend UI is loaded in the browser, the application makes API calls to the backend via the domain `api.moneytrack.com` to fetch or submit data.
5. **Result:** The user sees the application interface and can interact with the system.

### Key Concepts

* **Separation of Frontend and Backend:** Keeping the frontend static and decoupled from the backend allows us to host it cheaply and deliver it globally via a CDN, independent of where the backend computes the business logic.
* **Role of Amplify:** AWS Amplify provides a fully managed hosting service for static web apps. It handles the CDN, custom domain configuration, and SSL/TLS certificates automatically.
* **When Frontend Calls Backend API:** The frontend only calls the backend API when it needs dynamic data, such as authenticating a user, retrieving a list of transactions, or saving new data.
* **Deployment Considerations:** When deploying the frontend, environment variables are used to inject the correct API endpoint URL (`api.moneytrack.com`). The custom domain mapping in Amplify ensures users can access the site using the branded domain.
