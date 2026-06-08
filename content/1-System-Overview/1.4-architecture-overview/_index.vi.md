---
title: "Sơ đồ kiến trúc tổng quan"
date: 2026-06-03
weight: 4
chapter: false
pre: " <b> 1.4 </b> "
---

# 1.4 Sơ đồ kiến trúc tổng quan

![Sơ đồ kiến trúc tổng quan](/images/architecture-overview.png)

Sơ đồ trên mô tả **kiến trúc Multi-Region** của ứng dụng MoneyTrack. Hệ thống chạy đồng thời trên hai AWS Region, đảm bảo tính sẵn sàng cao và độ trễ thấp cho người dùng toàn cầu.

---

### Luồng truy cập chính (API)

Người dùng truy cập API của ứng dụng tại `api.moneytrack.com`. Request đi qua các tầng sau:

1. **User** gửi request đến `api.moneytrack.com`.
2. **Route 53** phân giải DNS và chuyển request đến **AWS Global Accelerator**.
3. **Global Accelerator** xác định Region có độ trễ thấp nhất (Primary hoặc Secondary Region) và chuyển tiếp request đến **Application Load Balancer (ALB)** tương ứng.
4. **AWS WAF** được gắn vào ALB kiểm tra và lọc request, chặn các mối đe dọa như SQL Injection, XSS, hoặc request từ IP độc hại.
5. ALB phân phối request đến các **ECS Fargate** container đang chạy trong **Private Subnet**.
6. ECS Fargate kết nối với **DynamoDB Global Tables** thông qua **VPC Endpoint** (Gateway Endpoint) — không đi qua internet.
7. **DynamoDB Global Tables** đồng bộ dữ liệu giữa hai Region theo thời gian thực, đảm bảo tính nhất quán.

---

### Frontend

- **User** truy cập `www.moneytrack.com` — được phân giải bởi Route 53 đến **AWS Amplify**.
- Amplify host ứng dụng frontend (React/Next.js) và phân phối nội dung toàn cầu qua CDN tích hợp sẵn — không cần hạ tầng bổ sung.

---

### Networking nội bộ

Mỗi Region có một **VPC** riêng biệt với cấu trúc hai tầng subnet:

| Subnet | Chứa | Mục đích |
|--------|------|----------|
| **Public Subnet** | ALB, Internet Gateway (IGW), NAT Gateway | Điểm tiếp nhận lưu lượng từ internet vào |
| **Private Subnet** | ECS Fargate container | Chạy workload ứng dụng; không expose ra internet |

- **Interface Endpoint** cho phép ECS Fargate giao tiếp với các dịch vụ AWS (ECR, Secrets Manager, SSM, CloudWatch) mà không ra internet.
- **Gateway Endpoint** cho phép ECS Fargate kết nối với DynamoDB qua mạng nội bộ riêng của AWS.

---

### CI/CD Pipeline

- Mỗi khi có thay đổi mã nguồn được đẩy lên **GitHub**, pipeline CI/CD tự động kích hoạt.
- Pipeline build Docker image, đẩy lên **Amazon ECR**, sau đó deploy phiên bản mới lên ECS Fargate ở **cả hai Region** đồng thời.

---

### Security

| Tầng | Dịch vụ | Vai trò |
|------|---------|---------|
| **Mạng** | AWS WAF | Kiểm tra và chặn request HTTP/HTTPS độc hại tại ALB |
| **Định danh** | IAM | Phân quyền tối thiểu cho từng dịch vụ (ECS Task Role, GitHub Actions Role) |
| **Secret** | AWS Secrets Manager | Lưu trữ và tự động xoay vòng secret (API key, thông tin xác thực database) |
| **Cấu hình** | AWS Systems Manager (SSM) | Quản lý tham số cấu hình theo từng môi trường |

---

### Monitoring & Observability

| Dịch vụ | Vai trò |
|---------|---------|
| **Amazon CloudWatch** | Thu thập log tập trung và chỉ số đo lường; CloudWatch Alarms kích hoạt cảnh báo khi vượt ngưỡng |
| **Amazon Managed Service for Prometheus** | Thu thập metrics theo chuẩn Prometheus từ tầng ứng dụng |
| **Amazon Managed Grafana** | Dashboard thời gian thực để quan sát hiệu năng hệ thống; tích hợp cảnh báo |

{{% notice info %}}
Trong các phần tiếp theo, bạn sẽ được hướng dẫn từng bước triển khai từng thành phần của kiến trúc này — từ việc cấp phát hạ tầng bằng Terraform, đến cấu hình CI/CD pipeline và thiết lập giám sát hệ thống.
{{% /notice %}}
