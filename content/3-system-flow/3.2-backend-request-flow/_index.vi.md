---
title: "Luồng request backend"
date: 2026-06-03
weight: 2
chapter: false
pre: " <b> 3.2 </b> "
---

# 3.2 Luồng request backend

Luồng này trình bày chi tiết cách các API request động được định tuyến và xử lý bởi các dịch vụ backend.

### Chi tiết luồng

```txt
User → api.moneytrack.com → Route 53 → Global Accelerator → AWS WAF → ALB → ECS Fargate → Response
```

1. **User Action:** Người dùng thao tác trên frontend (ví dụ: đăng nhập hoặc xem giao dịch), khiến ứng dụng gửi HTTP request đến `api.moneytrack.com`.
2. **Phân giải DNS:** **Route 53** phân giải DNS cho API domain thành các địa chỉ anycast IP được cung cấp bởi AWS Global Accelerator.
3. **Định tuyến toàn cầu:** **Global Accelerator** nhận traffic và định tuyến nó qua mạng toàn cầu của AWS đến region tối ưu (thường là region gần nhất) hoặc region đang healthy.
4. **Kiểm tra bảo mật:** Trước khi đến load balancer, request đi qua **AWS WAF**. WAF kiểm tra request dựa trên các rule bảo mật (ví dụ: kiểm tra SQL injection hoặc IP độc hại).
5. **Load Balancing:** Nếu request hợp lệ, WAF cho phép nó đi qua đến Application Load Balancer (**ALB**) nằm trong public subnet.
6. **Xử lý:** ALB forward request đến một **ECS Fargate** task đang rảnh chạy trong private subnet. Fargate task thực thi business logic của ứng dụng.
7. **Kết quả:** ECS Fargate tạo response và gửi ngược trở lại qua ALB, Global Accelerator, và cuối cùng về cho người dùng.

### Các khái niệm chính

* **Vai trò của Route 53 trong DNS:** Route 53 hoạt động như dịch vụ DNS, hướng người dùng muốn truy cập `api.moneytrack.com` đến điểm vào chính xác (Global Accelerator).
* **Vai trò của Global Accelerator:** Nó cải thiện hiệu suất bằng cách định tuyến traffic qua mạng private của AWS thay vì public internet, và cung cấp khả năng failover nhanh chóng giữa các region.
* **Vai trò của WAF:** WAF đóng vai trò như một lá chắn, lọc bỏ các request độc hại trước khi chúng tiêu thụ tài nguyên tính toán hoặc xâm phạm ứng dụng.
* **Vai trò của ALB:** ALB phân phối các request đến đồng đều trên nhiều ECS task trong một region, đảm bảo không có task nào bị quá tải.
* **Vì sao ECS Fargate chạy trong private subnet:** Chạy backend task trong private subnet đảm bảo chúng không thể bị truy cập trực tiếp từ internet, giảm thiểu đáng kể bề mặt tấn công (attack surface). Chúng chỉ nhận traffic từ ALB.
