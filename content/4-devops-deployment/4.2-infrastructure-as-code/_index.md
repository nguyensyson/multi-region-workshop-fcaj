---
title: "Infrastructure as Code with Terraform"
date: 2026-06-03
weight: 2
chapter: false
pre: " <b> 4.2 </b> "
---

# 4.2 Infrastructure as Code with Terraform

### Role of Terraform

* Terraform is an open-source tool used to define and manage infrastructure as code (IaC).
* It allows you to create, update, reuse, and version-control your infrastructure changes safely and predictably.
* It is highly suitable for multi-region architectures because resources can be managed using modules and reused across different environments and regions.
* Using Terraform significantly reduces the risk of manual misconfigurations in the AWS Console.

### Components Managed by Terraform

You should manage the following groups of resources using Terraform:
* **Networking:** VPC, subnets, route tables, Internet Gateway (IGW), NAT Gateway, and VPC Endpoints.
* **Compute:** ECS Cluster, ECS Service, and Task Definitions.
* **Load Balancing:** Application Load Balancer (ALB), Target Groups, and Listeners.
* **Database:** DynamoDB Global Tables.
* **Security:** IAM Roles and Policies, Security Groups, AWS WAF, and Secrets Manager secrets.
* **Container Registry:** Amazon ECR repositories.
* **Monitoring:** CloudWatch Log Groups and CloudWatch Alarms.
* **Remote State:** S3 bucket for storing the Terraform state and DynamoDB table for state locking.

### Organizing Terraform Modules

A common and effective way to organize Terraform code is:

```txt
terraform/
├── modules/
│   ├── networking/
│   ├── ecs/
│   ├── alb/
│   ├── dynamodb/
│   ├── security/
│   └── monitoring/
└── environments/
    ├── dev/
    ├── staging/
    └── prod/
```

* The `modules/` directory contains reusable, generalized Terraform configurations.
* The `environments/` directory contains configurations specific to each environment (dev, staging, prod), which call the modules.
* Each environment can pass specific variables (like region, CIDR blocks, desired task count, domain names) into the modules.
* Multi-region deployments can be managed by using provider aliases or by calling the modules multiple times for each target region.

### Managing Terraform State

* The Terraform state file stores the current state of your deployed infrastructure and maps it to your configuration.
* **Do not** store the state file locally for team or production projects.
* Use a remote backend, such as Amazon S3, to store the state file securely.
* Enable versioning on the S3 bucket holding the state to recover from accidental deletions or corruptions.
* Use a DynamoDB table for state locking to prevent concurrent executions from corrupting the state file.
* Protect the state file strictly (e.g., using IAM policies), as it may contain sensitive information in plain text.

### Environment Management

* Separate dev, staging, and production environments into distinct directories or workspaces to prevent changes in one from affecting another.
* Each environment should use its own `tfvars` file for unique variables.
* Production deployments require strict controls over the `terraform apply` process.
* Always review the output of `terraform plan` before applying changes.
* Terraform can be integrated into a CI/CD pipeline, but deployments to production should typically require manual approval.

### Infrastructure Deployment Flow

```txt
Developer → Terraform Code → terraform plan → Review → terraform apply → AWS Infrastructure
```

1. **Update Code:** The developer updates the Terraform code in their local environment.
2. **Format/Validate:** Run `terraform fmt` and `terraform validate` to ensure code quality and syntax correctness.
3. **Plan:** Run `terraform plan` to preview the exact changes Terraform will make to the infrastructure.
4. **Review:** The team reviews the generated plan before proceeding.
5. **Apply:** Run `terraform apply` to execute the changes and create/update the AWS infrastructure.
6. **Verify:** Check the AWS Console and monitoring dashboards to confirm the resources were deployed correctly.

### Deployment Considerations

* **No Hard-coded Secrets:** Never hard-code secrets (like passwords or API keys) in Terraform code. Use Secrets Manager or pass them securely during runtime.
* **Use Variables:** Utilize variables and environment-specific configuration files extensively.
* **State Versioning:** Ensure S3 versioning is enabled for your state backend.
* **Least Privilege:** Ensure the IAM role executing Terraform has only the permissions required to manage the infrastructure.
* **Destructive Changes:** Be extremely careful with changes that result in resources being destroyed and recreated.
* **Lifecycle Rules:** For critical resources like databases, use Terraform lifecycle blocks like `prevent_destroy = true` as an extra safeguard.
* **Modularization:** Keep modules focused and clearly separated to ease maintenance.
* **Tagging:** Consistently tag resources to help manage costs and identify resource ownership.
