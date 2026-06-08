---
title: "Monitoring / Observability"
date: 2026-06-03
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# 5. Monitoring / Observability

Mục tiêu chính của việc giám sát (monitoring) và quan sát (observability) trong dự án MoneyTrack là phát hiện lỗi sớm, theo dõi hiệu năng hệ thống và hỗ trợ khắc phục sự cố (troubleshooting) một cách hiệu quả. Bằng cách có cái nhìn rõ ràng về kiến trúc multi-region, chúng ta có thể đảm bảo trải nghiệm liền mạch cho người dùng.

### Theo dõi CI/CD Pipeline Log

Việc theo dõi các pipeline triển khai giúp xác định các vấn đề trước khi chúng lên môi trường production.

*   **Frontend:** Chúng ta theo dõi build và deploy log trực tiếp trên console của **AWS Amplify**.
*   **Backend:** Chúng ta theo dõi pipeline log trong các giai đoạn build Docker image, push image lên Amazon ECR và deploy lên ECS Fargate.

**Các lỗi thường gặp trong pipeline:**
*   Build fail ở frontend hoặc backend do lỗi cú pháp hoặc thiếu dependency.
*   Test fail (unit test, integration test).
*   Docker build fail.
*   Lỗi push image lên ECR (thường do thiếu quyền IAM).
*   Lỗi ECS deployment fail (ví dụ: task không khởi động được hoặc không vượt qua ALB health check).

![Pipeline Log](/images/pipeline-log.png)

### Theo dõi Application Log

Application log rất quan trọng để hiểu trạng thái bên trong của các dịch vụ backend.

*   Các backend container chạy trên **ECS Fargate** được cấu hình để gửi luồng output và error trực tiếp về **Amazon CloudWatch Logs**.
*   Log này giúp chúng ta kiểm tra chi tiết các request, lần theo các exception, xác định error và xác minh trạng thái xử lý nghiệp vụ.

### Theo dõi Metrics

Chúng ta thu thập và theo dõi các metric ở mọi tầng của kiến trúc:

*   **ECS/Fargate:** Mức sử dụng CPU, memory và số lượng task đang chạy (running task count).
*   **Application Load Balancer (ALB):** Số lượng request (request count), độ trễ (latency), tỷ lệ lỗi HTTP 4xx/5xx và số lượng target khỏe mạnh (healthy target).
*   **DynamoDB Global Tables:** Dung lượng read/write đã tiêu thụ (capacity), số lượng request bị chặn lại do quá tải (throttled requests), độ trễ truy vấn và độ trễ đồng bộ (replication) giữa các region.
*   **AWS WAF:** Số lượng request được cho phép (allowed) so với các request bị chặn (blocked), giúp nhận diện các cuộc tấn công.

### Trực quan hóa với Grafana Dashboard

Để dễ dàng phân tích dữ liệu thu thập được, chúng ta sử dụng dashboard để trực quan hóa.

*   **Amazon Managed Grafana** cung cấp một nền tảng bảo mật, tập trung để trực quan hóa các metric.
*   Chúng ta sử dụng **Amazon Managed Service for Prometheus** (nếu được cấu hình) để thu thập các metric chi tiết ở cấp độ ứng dụng từ các backend service.
*   Dashboard cốt lõi của chúng ta bao gồm các biểu đồ về request rate, error rate, latency, mức sử dụng CPU/memory của ECS và các metric hiệu suất của DynamoDB.

![Dashboard Grafana](/images/grafana-dashboard.png)

### Chiến lược Alerting (Cảnh báo)

Cảnh báo chủ động đảm bảo team vận hành được thông báo ngay lập tức khi có sự cố. Chúng ta thiết lập cảnh báo cho các ngưỡng tới hạn, chẳng hạn như:

*   Tỷ lệ lỗi HTTP 5xx tăng đột biến tại ALB.
*   Các ECS task bị dừng đột ngột (task stopped).
*   Độ trễ cao kéo dài ảnh hưởng đến trải nghiệm người dùng.
*   Hiện tượng throttling trên DynamoDB, báo hiệu cần điều chỉnh capacity.
*   ALB báo cáo có unhealthy target, cho thấy ứng dụng backend đang gặp lỗi.
