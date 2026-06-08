---
title: "Monitoring services - CloudWatch, Prometheus, Grafana"
date: 2026-06-03
weight: 7
chapter: false
pre: " <b> 2.7 </b> "
---

# 2.7 Monitoring services - CloudWatch, Prometheus, Grafana

### CloudWatch
Amazon CloudWatch là một dịch vụ giám sát và observability. Trong kiến trúc của chúng ta, nó chủ yếu được sử dụng để thu thập log (từ các ECS task và ALB), thu thập các metric hạ tầng AWS tiêu chuẩn và thiết lập alarm để thông báo cho người vận hành khi các ngưỡng cụ thể bị vi phạm.

### Prometheus (Amazon Managed Service for Prometheus)
Prometheus là một bộ công cụ mã nguồn mở dùng để giám sát hệ thống và cảnh báo. Amazon Managed Service for Prometheus cung cấp một môi trường có tính sẵn sàng cao, bảo mật và được quản lý hoàn toàn cho Prometheus. Chúng ta sử dụng nó để scrape (thu thập) các metric chi tiết, tùy chỉnh ở cấp độ ứng dụng trực tiếp từ các backend service của mình.

### Grafana (Amazon Managed Grafana)
Grafana là một ứng dụng web mã nguồn mở dùng để phân tích và trực quan hóa tương tác. Amazon Managed Grafana là một dịch vụ trực quan hóa dữ liệu bảo mật và được quản lý hoàn toàn. Chúng ta sử dụng nó để tạo các dashboard toàn diện giúp trực quan hóa các metric từ cả CloudWatch và Prometheus trên một giao diện thống nhất.

![Dashboard Grafana](/images/grafana-dashboard.png)

### Vai trò của monitoring trong vận hành
Giám sát hiệu quả là điều cốt lõi để vận hành một hệ thống multi-region. Nó cung cấp khả năng quan sát cần thiết để:
* **Phát hiện lỗi:** Nhanh chóng xác định các request bị lỗi hoặc các container ứng dụng bị crash.
* **Phân tích hiệu năng:** Hiểu rõ các điểm nghẽn về độ trễ và xu hướng sử dụng tài nguyên.
* **Vận hành hệ thống:** Đưa ra các quyết định sáng suốt về việc mở rộng (scaling), hoạch định dung lượng (capacity planning) và ứng phó với sự cố (incident response).

### Các ví dụ về metric cần theo dõi
* **Hạ tầng:** Mức sử dụng CPU và Memory của các ECS task.
* **Ứng dụng/Traffic:** Request count (Số lượng request), error rate (Tỷ lệ lỗi HTTP 4xx/5xx) và latency (độ trễ, thời gian phản hồi).
* **Sức khỏe:** Trạng thái health của ECS task và số lần khởi động lại (restart count).
* **Cơ sở dữ liệu:** DynamoDB read/write capacity đã tiêu thụ, các sự kiện throttling và replication latency (độ trễ sao chép).

### Lưu ý triển khai
* **Log retention:** Cấu hình chính sách lưu giữ log (retention policy) trong CloudWatch Logs để cân bằng giữa nhu cầu khắc phục sự cố và chi phí lưu trữ. Đừng giữ log vô thời hạn trừ khi có yêu cầu tuân thủ.
* **Dashboard:** Thiết kế các Grafana dashboard trình bày cái nhìn tổng quan, rõ ràng về sức khỏe hệ thống trước khi đi sâu vào các metric chi tiết.
* **Alerting:** Thiết lập các cảnh báo có ý nghĩa (ví dụ: qua CloudWatch Alarms hoặc Grafana Alerting) để thông báo cho team về các vấn đề cần xử lý, tránh tình trạng "alert fatigue" (mệt mỏi vì cảnh báo) do các thông báo ồn ào, không quan trọng.
* **Metric naming:** Áp dụng quy ước đặt tên metric nhất quán trong code ứng dụng của bạn cho Prometheus scrape để giúp việc tạo dashboard và truy vấn dễ dàng hơn.
* **Centralized logging:** Đảm bảo log từ tất cả các region đều dễ dàng truy cập và tìm kiếm, có thể sử dụng CloudWatch Logs Insights hoặc một tài khoản centralized logging.
