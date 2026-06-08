---
title: "Mục tiêu thiết kế"
date: 2026-06-03
weight: 2
chapter: false
pre: " <b> 1.2 </b> "
---

# 1.2 Mục tiêu thiết kế

Hệ thống được thiết kế hướng đến sáu mục tiêu cốt lõi:

| Mục tiêu | Mô tả |
|----------|-------|
| **High Availability** | Hệ thống luôn sẵn sàng phục vụ, kể cả khi một Region gặp sự cố. Route 53 và Global Accelerator tự động chuyển lưu lượng sang Region còn hoạt động. |
| **Scalability** | ECS Fargate tự động scale out/in theo tải thực tế. DynamoDB Global Tables xử lý hàng triệu request mà không cần cấu hình sharding thủ công. |
| **Security** | AWS WAF lọc request độc hại. IAM kiểm soát quyền truy cập tối thiểu. Secrets Manager quản lý thông tin nhạy cảm thay vì lưu trong code hay biến môi trường. |
| **Disaster Recovery** | DynamoDB Global Tables đồng bộ dữ liệu giữa các Region theo thời gian thực. Khi một Region gặp sự cố, Region còn lại tiếp tục phục vụ với dữ liệu đầy đủ. |
| **Automation (CI/CD & IaC)** | Toàn bộ hạ tầng được định nghĩa bằng **Terraform**. Pipeline CI/CD trên GitHub Actions tự động build, push image lên ECR và deploy lên ECS khi có thay đổi mã nguồn. |
| **Observability** | CloudWatch thu thập logs và metrics từ ECS, ALB, và các dịch vụ AWS. Amazon Managed Service for Prometheus và Amazon Managed Grafana cung cấp dashboard trực quan để theo dõi sức khoẻ hệ thống và thiết lập cảnh báo. |

### Chi tiết từng mục tiêu

#### High Availability
Một hệ thống có tính sẵn sàng cao sẽ chịu được lỗi ở từng thành phần riêng lẻ, thậm chí toàn bộ một Region, mà không ảnh hưởng đến người dùng cuối. Trong kiến trúc này, **Route 53** liên tục kiểm tra sức khoẻ (health check) của từng Region. Nếu một Region không còn hoạt động, **AWS Global Accelerator** sẽ tự động chuyển toàn bộ lưu lượng sang Region còn lại — hoàn toàn tự động, không cần can thiệp thủ công.

#### Scalability
**ECS Fargate** loại bỏ sự cần thiết phải quản lý EC2 instance. Các task tự động scale ngang (horizontal scaling) dựa trên chỉ số CPU và bộ nhớ thông qua Application Auto Scaling. **DynamoDB Global Tables** là cơ sở dữ liệu serverless, fully managed, có khả năng tự điều chỉnh dung lượng theo nhu cầu mà không cần cấu hình trước.

#### Security
Bảo mật được áp dụng theo nhiều tầng:
- **Tầng mạng**: AWS WAF kiểm tra toàn bộ HTTP/HTTPS request đến ALB.
- **Tầng định danh**: IAM role với chính sách quyền tối thiểu kiểm soát những gì mỗi dịch vụ có thể truy cập.
- **Tầng dữ liệu**: AWS Secrets Manager lưu trữ và tự động xoay vòng secret; SSM Parameter Store quản lý các tham số cấu hình không nhạy cảm.

#### Disaster Recovery
Hệ thống áp dụng mô hình **Active-Active** multi-region. Cả hai Region đều phục vụ lưu lượng thực tế cùng lúc. **DynamoDB Global Tables** sử dụng cơ chế replication multi-active với độ trễ dưới một giây, đảm bảo dữ liệu nhất quán giữa các Region mọi lúc.

#### Automation (CI/CD & IaC)
- **Terraform** định nghĩa toàn bộ tài nguyên AWS dưới dạng code, giúp triển khai nhất quán và có thể kiểm tra lại.
- **GitHub Actions** tự động hóa toàn bộ pipeline: build Docker image → push lên ECR → cập nhật ECS service — kích hoạt mỗi khi có merge vào nhánh main.

#### Observability
- **CloudWatch** cung cấp thu thập log tập trung và các chỉ số đo lường từ cả hai Region.
- **Amazon Managed Prometheus** và **Amazon Managed Grafana** cung cấp chỉ số ở tầng ứng dụng và dashboard trực quan.
- Cảnh báo thông báo ngay cho đội on-call khi độ trễ, tỷ lệ lỗi hoặc mức sử dụng tài nguyên vượt ngưỡng đã định.
