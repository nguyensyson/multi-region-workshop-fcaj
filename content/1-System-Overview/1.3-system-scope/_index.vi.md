---
title: "Phạm vi hệ thống"
date: 2026-06-03
weight: 3
chapter: false
pre: " <b> 1.3 </b> "
---

# 1.3 Phạm vi hệ thống

### Trong phạm vi workshop

Workshop này bao gồm đầy đủ các thành phần sau:

| Thành phần | Mô tả |
|------------|-------|
| **Frontend** | Ứng dụng web được host trên **AWS Amplify**, phân phối nội dung tĩnh toàn cầu qua CDN tích hợp sẵn. |
| **Backend** | Container API chạy trên **ECS Fargate** trong Private Subnet, không expose trực tiếp ra internet. |
| **Database** | **DynamoDB Global Tables** đồng bộ dữ liệu giữa hai Region theo thời gian thực. |
| **Networking** | VPC với Public/Private Subnet, NAT Gateway, Internet Gateway, VPC Endpoint (Interface & Gateway). |
| **Routing & Traffic Management** | **Route 53** phân giải DNS; **AWS Global Accelerator** định tuyến thông minh đến Region có độ trễ thấp nhất. |
| **Security** | **AWS WAF** tại tầng ALB; **IAM** phân quyền tối thiểu cho từng dịch vụ; **AWS Secrets Manager** và **AWS Systems Manager** quản lý secret và tham số cấu hình. |
| **Container Registry** | **Amazon ECR** lưu trữ và quản lý phiên bản Docker image của backend. |
| **CI/CD** | **GitHub Actions** tự động hóa toàn bộ quy trình build và deploy. |
| **Infrastructure as Code** | **Terraform** quản lý toàn bộ hạ tầng AWS theo cách khai báo. |
| **Monitoring & Observability** | **CloudWatch**, **Amazon Managed Service for Prometheus**, **Amazon Managed Grafana**. |

### Ngoài phạm vi workshop

Để tập trung vào kiến trúc và vận hành hệ thống, workshop này **không** đi sâu vào:

- **Mã nguồn ứng dụng**: Workshop cung cấp sẵn ứng dụng mẫu đã được build. Chi tiết triển khai business logic không thuộc phạm vi này.
- **Tối ưu chi phí nâng cao**: Các chiến lược như Reserved Instances, Savings Plans, Spot Instances cho Fargate, hay Cost Anomaly Detection nằm ngoài phạm vi workshop.
- **Kiểm thử phần mềm**: Unit test, integration test và phương pháp load test chi tiết không được đề cập.

{{% notice tip %}}
Nếu bạn quan tâm đến tối ưu chi phí sau khi hoàn thành workshop, hãy tham khảo [AWS Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html) trong AWS Well-Architected Framework.
{{% /notice %}}
