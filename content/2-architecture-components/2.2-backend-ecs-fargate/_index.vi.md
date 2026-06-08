---
title: "Backend - ECS Fargate"
date: 2026-06-03
weight: 2
chapter: false
pre: " <b> 2.2 </b> "
---

# 2.2 Backend - ECS Fargate

### ECS Fargate là gì?
Amazon Elastic Container Service (ECS) là một dịch vụ điều phối container được quản lý hoàn toàn. **Fargate** là một engine tính toán serverless dành cho container hoạt động cùng với ECS. Nó cho phép bạn chạy các container mà không cần phải quản lý máy chủ hoặc cluster các Amazon EC2 instance.

### Vai trò của ECS Fargate trong hệ thống Backend
ECS Fargate đóng vai trò host ứng dụng backend API đã được container hóa của chúng ta. Nó pull Docker image từ Amazon ECR và chạy dưới dạng một task, xử lý các API request đến và tương tác với cơ sở dữ liệu DynamoDB.

### Cách backend nhận request
Lưu lượng truy cập được định tuyến từ Application Load Balancer (ALB) đến các ECS Fargate task. ALB đóng vai trò là một điểm tiếp xúc duy nhất, phân phối lưu lượng ứng dụng đến qua nhiều target (các ECS task của chúng ta) trong nhiều Availability Zone.

### Chạy container trong private subnet
Vì lý do bảo mật, các ECS Fargate task được triển khai trong **private subnet**. Chúng không có địa chỉ IP public và không thể truy cập trực tiếp từ internet. Chúng chỉ nhận request từ ALB (nằm trong public subnet) và truy cập các dịch vụ bên ngoài (như pull image từ ECR) thông qua NAT Gateway hoặc VPC Endpoints.

### Lý do chọn ECS Fargate
* **Serverless Compute:** Tập trung vào việc xây dựng ứng dụng, không phải quản lý hạ tầng. Không có EC2 instance nào cần vá lỗi, scale hay bảo mật.
* **Scale linh hoạt:** Dễ dàng mở rộng hoặc thu hẹp dựa trên mức sử dụng CPU hoặc bộ nhớ bằng cách sử dụng ECS Service Auto Scaling.
* **Bảo mật:** Các task chạy trong môi trường tính toán biệt lập của riêng chúng, tăng cường bảo mật.
* **Tối ưu chi phí:** Chỉ trả tiền cho tài nguyên vCPU và bộ nhớ mà ứng dụng của bạn tiêu thụ.

### Lưu ý triển khai
* **Task Definition:** Chỉ định Docker image, CPU, bộ nhớ, biến môi trường và cấu hình logging cho ứng dụng của bạn.
* **Service:** Quản lý một số lượng (desired count) các bản sao của task definition và đăng ký chúng với ALB.
* **Desired Count & Auto Scaling:** Đặt số lượng task cơ bản và cấu hình các rule auto-scaling để xử lý các đợt tăng đột biến lưu lượng.
* **Health Check:** Cấu hình đường dẫn health check của ALB để nó có thể xác định chính xác xem một task có khỏe mạnh hay không trước khi gửi request đến.
* **IAM Task Role:** Cấp cho task các quyền cụ thể, chẳng hạn như khả năng đọc từ Secrets Manager hoặc ghi vào DynamoDB, tuân thủ nguyên tắc quyền tối thiểu (least privilege).
* **Security Group:** Cấu hình các rule để chỉ cho phép lưu lượng inbound từ security group của ALB.
