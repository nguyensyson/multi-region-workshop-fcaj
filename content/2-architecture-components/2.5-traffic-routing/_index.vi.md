---
title: "Traffic routing - Route 53, Global Accelerator, ALB"
date: 2026-06-03
weight: 5
chapter: false
pre: " <b> 2.5 </b> "
---

# 2.5 Traffic routing - Route 53, Global Accelerator, ALB

### Route 53
Amazon Route 53 là một dịch vụ web Hệ thống phân giải tên miền (DNS) trên cloud có tính sẵn sàng cao và khả năng mở rộng tốt. Nó được dùng để quản lý các bản ghi DNS và định tuyến người dùng đến các ứng dụng Internet. Trong kiến trúc của chúng ta, nó đóng vai trò là điểm vào ban đầu, biên dịch tên miền dễ đọc (như `api.moneytrack.com`) thành các địa chỉ IP của AWS Global Accelerator.

### AWS Global Accelerator
AWS Global Accelerator là một dịch vụ mạng giúp cải thiện tính sẵn sàng và hiệu suất của các ứng dụng có người dùng toàn cầu. Nó cung cấp các địa chỉ IP tĩnh đóng vai trò là một điểm vào cố định cho các application endpoint của bạn ở một hoặc nhiều AWS Region.
* Nó định tuyến traffic qua mạng toàn cầu của AWS đến region tối ưu dựa trên vị trí địa lý và tình trạng sức khỏe của region đó.

### Application Load Balancer (ALB)
Application Load Balancer hoạt động ở cấp độ request (Layer 7), định tuyến traffic đến các target (ECS Fargate tasks) trong một Amazon VPC dựa trên nội dung của request. Nó phân phối các request ứng dụng đến trên nhiều target ở nhiều Availability Zone.

### Luồng traffic tổng quan
1. **User** khởi tạo một request đến `api.moneytrack.com`.
2. **Route 53** phân giải domain thành các địa chỉ IP tĩnh được cung cấp bởi **Global Accelerator**.
3. **Global Accelerator** hướng traffic đến endpoint group của region tối ưu (chính là ALB của chúng ta).
4. **ALB** nhận request và phân phối nó đến một **ECS Fargate** task đang khỏe mạnh (healthy) chạy backend API.

### Vai trò của health check trong failover
Health check rất quan trọng đối với tính sẵn sàng cao (High Availability).
* **ALB** liên tục kiểm tra sức khỏe của các ECS task. Nếu một task bị lỗi, ALB sẽ ngừng định tuyến traffic đến nó.
* **Global Accelerator** liên tục kiểm tra sức khỏe của các endpoint group của nó (các ALB ở mỗi region). Nếu toàn bộ một region bị sập hoặc trở nên "unhealthy", Global Accelerator sẽ lập tức failover và định tuyến toàn bộ traffic sang region đang khỏe mạnh.

### Lý do kết hợp Route 53, Global Accelerator và ALB
* **Route 53** cung cấp phân giải DNS đáng tin cậy.
* **Global Accelerator** cung cấp định tuyến xác định qua mạng AWS, cải thiện hiệu suất cho người dùng toàn cầu và cung cấp khả năng failover nhanh chóng, không phụ thuộc IP giữa các region.
* **ALB** cung cấp load balancing ở layer 7 và health check chi tiết trong một region cụ thể.
Việc kết hợp chúng mang lại một kiến trúc định tuyến mạnh mẽ, hiệu suất cao và sẵn sàng cao.

### Lưu ý triển khai
* **DNS Record:** Cấu hình bản ghi Alias trong Route 53 để trỏ đến DNS name của Global Accelerator.
* **Listener và Target Group:** Cấu hình ALB listener (ví dụ: port 443 cho HTTPS) và các target group trỏ đến ECS service.
* **Health Check:** Cấu hình đường dẫn health check chính xác và phản hồi nhanh trên ALB để nó có thể nhanh chóng xác định các task bị lỗi.
* **Chứng chỉ SSL/TLS:** Sử dụng AWS Certificate Manager (ACM) để cấp phát và quản lý chứng chỉ SSL/TLS cho ALB nhằm kích hoạt giao tiếp HTTPS bảo mật.
