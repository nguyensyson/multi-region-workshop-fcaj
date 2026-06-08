---
title: "Infrastructure as Code với Terraform"
date: 2026-06-03
weight: 2
chapter: false
pre: " <b> 4.2 </b> "
---

# 4.2 Infrastructure as Code với Terraform

### Vai trò của Terraform

* Terraform dùng để khai báo và quản lý infrastructure bằng code (IaC).
* Giúp tạo, cập nhật, tái sử dụng và kiểm soát thay đổi infrastructure một cách an toàn và có thể dự đoán được.
* Phù hợp với kiến trúc multi-region vì có thể quản lý tài nguyên theo module và tái sử dụng cho nhiều environment và region khác nhau.
* Giảm thiểu rủi ro từ các lỗi thao tác thủ công trên AWS Console.

### Các thành phần nên quản lý bằng Terraform

Nên quản lý các nhóm resource sau bằng Terraform:
* **Networking:** VPC, subnet, route table, IGW, NAT Gateway, VPC Endpoint.
* **Compute:** ECS Cluster, ECS Service, Task Definition.
* **Load Balancing:** ALB, Target Group, Listener.
* **Database:** DynamoDB Global Tables.
* **Security:** IAM Role, Security Group, WAF, Secrets Manager.
* **Container Registry:** ECR.
* **Monitoring:** CloudWatch Log Group, Alarm.
* **Remote state:** S3 backend và DynamoDB state lock (nếu có).

### Cách tổ chức Terraform module

Cách tổ chức code Terraform phổ biến và hiệu quả:

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

* `modules/` chứa các module cấu hình chung, có thể tái sử dụng.
* `environments/` chứa cấu hình riêng cho từng môi trường (dev, staging, prod), các thư mục này sẽ gọi đến các module ở trên.
* Mỗi environment có thể truyền vào các biến riêng như region, block CIDR, desired count, domain name.
* Kiến trúc multi-region có thể được quản lý bằng cách sử dụng provider alias hoặc gọi module nhiều lần cho từng region.

### Quản lý Terraform state

* Terraform state lưu trạng thái infrastructure hiện tại và map nó với cấu hình code của bạn.
* Không nên lưu state local cho các team hoặc project thực tế.
* Nên dùng remote backend như Amazon S3 để lưu trữ state file an toàn.
* Nên bật versioning cho S3 bucket chứa state để có thể phục hồi nếu lỡ xóa hoặc state bị hỏng.
* Có thể dùng DynamoDB để state locking, tránh tình trạng nhiều người apply cùng lúc làm hỏng state.
* Cần bảo vệ state file cẩn thận vì nó có thể chứa thông tin nhạy cảm (secrets) ở dạng plain text.

### Quản lý environment

* Tách biệt cấu hình dev, staging, production ra các thư mục để tránh thay đổi ở môi trường này ảnh hưởng đến môi trường khác.
* Mỗi environment có biến (variables) riêng.
* Production cần được kiểm soát chặt chẽ quá trình `terraform apply`.
* Luôn review kết quả `terraform plan` trước khi apply.
* Có thể tích hợp Terraform vào CI/CD pipeline nhưng cần thiết lập bước approval thủ công trước khi apply lên production.

### Luồng triển khai infrastructure

```txt
Developer → Terraform Code → terraform plan → Review → terraform apply → AWS Infrastructure
```

1. **Cập nhật code:** Developer cập nhật Terraform code.
2. **Format/Validate:** Chạy format (`terraform fmt`) và validate để kiểm tra cú pháp.
3. **Plan:** Chạy `terraform plan` để xem chi tiết các thay đổi mà Terraform sẽ thực hiện.
4. **Review:** Team review các thay đổi này trước khi apply.
5. **Apply:** Chạy `terraform apply` để tạo hoặc cập nhật infrastructure trên AWS.
6. **Kiểm tra:** Kiểm tra resource trên AWS Console và monitoring.

### Lưu ý triển khai

* Không hard-code secret trong Terraform code.
* Sử dụng biến và file cấu hình riêng cho từng environment.
* Bật versioning cho S3 bucket lưu Terraform state.
* Cấu hình IAM permission cho Terraform thực thi theo nguyên tắc least privilege.
* Đặc biệt cẩn thận với các thay đổi khiến resource bị destroy và tạo lại.
* Với database quan trọng, có thể dùng lifecycle rule như `prevent_destroy` làm chốt chặn an toàn.
* Chia module rõ ràng để dễ bảo trì.
* Tagging resource đầy đủ để quản lý cost và xác định ownership.
