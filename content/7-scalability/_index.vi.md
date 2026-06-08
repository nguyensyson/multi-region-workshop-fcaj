---
title: "Scalability"
date: 2026-06-03
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

# 7. Scalability

Khả năng mở rộng (Scalability) trong dự án MoneyTrack đề cập đến khả năng hệ thống xử lý mượt mà lượng traffic và dữ liệu ngày càng tăng mà không làm giảm hiệu suất, đồng thời có thể thu hẹp lại để tiết kiệm chi phí khi nhu cầu giảm.

### Khả năng mở rộng Frontend

*   Frontend là một ứng dụng web tĩnh được host trên **AWS Amplify**.
*   Frontend có thể phục vụ lượng truy cập tăng vọt mà chúng ta không cần tự quản lý hay cấp phát bất kỳ server nào, nhờ vào sức mạnh của hệ thống CDN bên dưới Amplify.
*   Có thể kết hợp với custom domain để tối ưu tốc độ và khả năng phục vụ người dùng trên diện rộng.

### Khả năng mở rộng Backend

*   Tầng compute của chúng ta sử dụng **ECS Fargate**, hỗ trợ scale các container backend một cách dễ dàng.
*   Chúng ta sử dụng **ECS Service Auto Scaling** để tự động tăng/giảm số lượng task đang chạy dựa trên các metric từ CloudWatch như mức sử dụng CPU, memory hoặc số lượng request.
*   Điều kiện tiên quyết là backend nên được thiết kế theo dạng **stateless** (không lưu trạng thái). Nhờ đó, bất kỳ task nào cũng có thể xử lý request, giúp việc scale ngang (scale out) bằng cách thêm task trở nên vô cùng đơn giản.

### Khả năng mở rộng ở tầng Load Balancing (ALB)

*   **Application Load Balancer (ALB)** có khả năng tự động mở rộng tài nguyên ngầm của chính nó để đáp ứng sự gia tăng traffic từ bên ngoài.
*   Khi backend scale thêm các ECS task mới, chúng sẽ tự động đăng ký vào Target Group. ALB sẽ lập tức phân phối traffic đến các target mới đang healthy này.

### Khả năng mở rộng Database

*   **Amazon DynamoDB** được sinh ra cho khả năng mở rộng cực lớn.
*   Nó hỗ trợ hai chế độ: On-Demand Capacity (tự động điều chỉnh theo tải thực tế, không cần cấu hình trước) hoặc Provisioned Capacity kết hợp với Auto Scaling.
*   Để database scale tốt, chúng ta cần thiết kế partition key tối ưu để phân tán đều lưu lượng đọc/ghi, tránh hiện tượng "hot partition" (nút cổ chai tại một vùng dữ liệu).
*   Chúng ta cần liên tục theo dõi các chỉ số throttling, latency và consumed capacity để điều chỉnh chiến lược scale.

### Khả năng mở rộng Multi-region

*   **DynamoDB Global Tables** giúp dữ liệu có mặt ở nhiều region, mang dữ liệu đến gần người dùng hơn và phân tán tải database.
*   **Global Accelerator** có khả năng phân phối traffic đến region phù hợp nhất.
*   Khi traffic tăng mạnh, chúng ta có thể mở rộng backend (scale ECS task) một cách độc lập ở từng region tùy thuộc vào lượng tải ở region đó.

### Luồng Auto Scaling

```txt
Traffic tăng → CloudWatch Metrics → ECS Auto Scaling → Thêm Fargate Tasks → ALB phân phối request
```

### Kết luận

Dự án MoneyTrack có kiến trúc linh hoạt, cho phép scale độc lập ở nhiều tầng khác nhau: frontend scale bằng CDN, backend scale ngang với ECS và ALB, database tự động scale với DynamoDB, và toàn bộ hệ thống mở rộng về mặt địa lý thông qua multi-region.
