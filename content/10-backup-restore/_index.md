---
title: "Backup & Restore"
date: 2026-06-03
weight: 10
chapter: false
pre: " <b> 10. </b> "
---

# 10. Backup & Restore

This section details the backup and restore strategies for critical components of the MoneyTrack project. Having robust backups ensures we can recover from accidental deletions, bad deployments, or data corruption.

## 10.1 Backup DynamoDB

**Description:**
*   DynamoDB is the primary data store for the MoneyTrack system.
*   While we use DynamoDB Global Tables to replicate data across regions for high availability, replication is *not* a substitute for backups. (If a script accidentally deletes a record, that deletion is immediately replicated).
*   We enable **Point-in-Time Recovery (PITR)** to restore data to any specific second within the preceding 35 days. This protects against accidental write or delete operations.
*   We also take **On-Demand Backups** before executing major changes, such as schema migrations, bulk data imports, or critical feature releases.
*   Backups should be managed using AWS Backup with appropriate lifecycle and retention policies to balance cost and compliance.

**Restore Procedure:**
1.  When a data issue is detected (e.g., incorrect data written), use PITR to restore the table to a point in time just before the error occurred.
2.  The restored data is always written to a *new* DynamoDB table.
3.  Carefully verify the data in the newly restored table.
4.  If the entire table was corrupted, you can re-point the application to the new table. If only specific records were affected, export the corrected records from the restored table and import them back into the active production table.
5.  After a full restore, re-establish Global Tables replication to the secondary region.

**Restore Flow:**
```txt
Data issue detected → Restore DynamoDB to selected point in time → Verify restored table → Switch application or import recovered data → Monitor replication
```

**Operational Considerations:**
*   Never rely solely on Global Tables as a backup mechanism; it replicates mistakes just as quickly as valid data.
*   Ensure encryption at rest is enabled for all DynamoDB tables and backups.
*   Monitor DynamoDB metrics (throttling, latency, replication latency).
*   Periodically test the restore process to ensure the team knows how to recover data quickly.
*   Strictly limit IAM permissions for who can perform backup and restore operations.

---

## 10.2 Backup Terraform State

**Description:**
*   The Terraform state file tracks the current status of the deployed AWS infrastructure. It is critical; without it, Terraform cannot accurately manage or update resources.
*   In the MoneyTrack project, we store the state file in a secure **S3 Remote Backend** rather than locally.
*   We must enable **Versioning** on this S3 bucket. If the state file becomes corrupted or someone accidentally applies a destructive change, we can revert to a previous version of the state.
*   Encryption must be enabled on the S3 bucket, as the state file often contains sensitive information (secrets, database passwords) in plain text.
*   We use a **DynamoDB table for state locking** to prevent race conditions when multiple engineers or CI/CD pipelines attempt to run `terraform apply` concurrently.

**Restore Procedure:**
1.  If the Terraform state becomes corrupted or out of sync, review the version history of the S3 state bucket.
2.  Select a known good version of the state from before the issue occurred.
3.  Restore that specific S3 object version to become the current state file.
4.  Run `terraform plan` to analyze the differences between the restored state and the actual infrastructure running in AWS.
5.  Review the plan carefully before running `terraform apply` to realign the infrastructure with the state.

**Restore Flow:**
```txt
State issue detected → Check S3 version history → Restore previous state version → Run terraform plan → Review → Apply if needed
```

**Operational Considerations:**
*   Never manually edit the state file unless absolutely necessary and only by a Terraform expert.
*   Never commit the `terraform.tfstate` file to GitHub or any source control.
*   Always enable versioning and encryption on the S3 backend bucket.
*   Apply the principle of least privilege via IAM for users or pipelines accessing the state bucket.
*   For critical resources like DynamoDB tables, use Terraform's `prevent_destroy = true` lifecycle rule to prevent accidental deletion.

---

## 10.3 Versioning Docker Images in ECR

**Description:**
*   The MoneyTrack backend is packaged as a Docker image and stored in **Amazon ECR**.
*   Every time the CI/CD pipeline builds the backend, the resulting image must be tagged with a clear, unique version.
*   We avoid using the generic `latest` tag for production deployments because it makes it incredibly difficult to know exactly what version is running and complicates the rollback process.
*   Recommended tagging strategies include:
    *   Git commit SHA (e.g., `a1b2c3d`)
    *   Semantic Release version (e.g., `v1.2.0`)
    *   Environment/Timestamp tag (e.g., `prod-2026-06-03`)
*   Strict versioning allows for instant rollbacks when a new deployment introduces a bug.

**Restore/Rollback Procedure:**
1.  If a newly deployed backend version causes errors, locate the tag of the previously stable image in Amazon ECR.
2.  Update the ECS Task Definition to point to this previous, stable image version.
3.  Redeploy the ECS Service using the updated Task Definition.
4.  The ALB health checks will ensure traffic is only routed to the newly started (rolled-back) tasks once they are healthy.
5.  Monitor CloudWatch Logs and Grafana dashboards to confirm the rollback resolved the issue.

**Restore Flow:**
```txt
New deployment failed → Select previous stable image in ECR → Update ECS Task Definition → Redeploy ECS Service → Verify health check and logs
```

**Operational Considerations:**
*   Do not delete old images too quickly.
*   Configure ECR Lifecycle Policies to automatically clean up very old images to balance rollback capability with storage costs.
*   Enable ECR Image Scanning to automatically detect vulnerabilities in your Docker images.
*   Always log the specific deployed image version in release notes or deployment tracking tools.
*   For production, only ever rollback to an image version that has previously passed all testing.

---

## Backup & Restore Summary

| Component | Backup / Versioning Strategy | Restore / Rollback Strategy | Operational Notes |
| :--- | :--- | :--- | :--- |
| **DynamoDB** | Enable Point-in-Time Recovery (PITR); take on-demand backups before major changes. | Restore to a new table via PITR; switch application to new table or migrate recovered data. | Global Tables do not replace backups; test restores periodically; restrict IAM access. |
| **Terraform State** | Store in S3 with Versioning and Encryption enabled; use DynamoDB for state locking. | Restore previous version from S3 history; run `terraform plan` to verify differences. | Never commit state to Git; do not edit state manually; use `prevent_destroy` for core DBs. |
| **ECR Docker Images** | Tag images uniquely (Commit SHA, Semantic Version) upon CI/CD build; avoid `latest`. | Update ECS Task Definition to use the previous stable image tag and redeploy the service. | Use ECR Lifecycle policies to manage storage; rollback only to verified stable images. |

## Conclusion

A comprehensive Backup & Restore strategy significantly reduces risk when dealing with data corruption, infrastructure misconfigurations, or faulty application deployments.
*   **DynamoDB** requires both Point-in-Time Recovery and targeted on-demand backups.
*   **Terraform State** must be protected using an S3 backend with versioning, encryption, and state locking.
*   **Docker Images in ECR** require strict versioning to enable rapid, reliable rollbacks.
*   Most importantly, restore procedures must be periodically tested in a safe environment, not just documented on paper.
