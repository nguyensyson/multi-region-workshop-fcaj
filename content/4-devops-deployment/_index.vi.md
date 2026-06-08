---
title: "DevOps & Deployment"
date: 2026-06-03
weight: 4
chapter: false
pre: " <b> 4. </b> "
---

# 4. DevOps & Deployment

Phần này giới thiệu vai trò của các thực hành và công cụ DevOps trong việc quản lý kiến trúc multi-region của chúng ta. Việc áp dụng văn hóa DevOps mạnh mẽ là rất quan trọng để vận hành một hệ thống phức tạp một cách đáng tin cậy ở quy mô lớn.

DevOps trong kiến trúc này phục vụ một số mục đích chính:
* **Tự động hóa:** Tự động hóa quá trình build, test và deploy cho cả ứng dụng frontend và backend.
* **Giảm thao tác thủ công:** Giảm thiểu lỗi do con người và sự can thiệp thủ công khi triển khai các bản cập nhật.
* **Tính nhất quán:** Đảm bảo infrastructure được quản lý nhất quán giữa các environment (dev, staging, prod) và các region.
* **Kiểm soát triển khai Multi-Region:** Hỗ trợ triển khai các thay đổi đến nhiều region một cách an toàn và có kiểm soát.
* **Tích hợp:** Kết nối các công cụ và dịch vụ khác nhau lại với nhau, bao gồm CI/CD pipeline, Terraform cho hạ tầng, Amazon ECR làm container registry, ECS Fargate để tính toán, Amplify để host frontend và các công cụ monitoring để giám sát.

#### Nội dung chính
1. [CI/CD pipeline cho frontend và backend](4.1-cicd-pipeline/)
2. [Infrastructure as Code với Terraform](4.2-infrastructure-as-code/)
