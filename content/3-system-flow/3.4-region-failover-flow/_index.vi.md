---
title: "Luồng failover giữa các region"
date: 2026-06-03
weight: 4
chapter: false
pre: " <b> 3.4 </b> "
---

# 3.4 Luồng failover giữa các region

Luồng này giải thích cách hệ thống xử lý khi một region gặp sự cố để duy trì tính sẵn sàng cao.

### Luồng hoạt động bình thường

Trong điều kiện bình thường, traffic được định tuyến đến region gần nhất hoặc region được ưu tiên.

```txt
User → Global Accelerator → Health Check → Healthy Region → ALB → ECS Fargate → DynamoDB Global Tables
```

1. **Định tuyến:** Traffic được **Global Accelerator** định tuyến đến endpoint group ở region chính (hoặc gần nhất).
2. **Theo dõi sức khỏe:** Global Accelerator và Route 53 health check liên tục theo dõi trạng thái của các endpoint (ALB) ở từng region.

### Luồng hoạt động khi Failover

Khi một region gặp sự cố hoặc trở nên unhealthy, quá trình sau sẽ diễn ra:

```txt
Region A unhealthy → Global Accelerator stops routing to Region A → Traffic routed to Region B → Application continues serving users
```

1. **Phát hiện lỗi:** Nếu Region A bị sập, các health check liên kết với endpoint group của Region A sẽ bị failed.
2. **Định tuyến lại (Rerouting):** **Global Accelerator** đánh dấu Region A là unhealthy và tự động chuyển toàn bộ traffic mới sang Region B đang healthy.
3. **Tiếp tục hoạt động:** Backend (ECS Fargate tasks) ở region dự phòng (Region B) nhận các request và tiếp tục xử lý chúng.
4. **Đảm bảo dữ liệu:** Vì **DynamoDB Global Tables** đã liên tục replicate dữ liệu ở background trước đó, ứng dụng ở Region B có quyền truy cập vào dữ liệu cập nhật nhất và có thể tiếp tục phục vụ người dùng mà không bị gián đoạn.
5. **Failback (Phục hồi):** Khi sự cố ở Region A được khắc phục và health check pass trở lại, Global Accelerator có thể dần dần đưa traffic quay lại Region A dựa theo chiến lược failback của bạn.

### Các khái niệm chính

* **Failover xảy ra ở tầng routing:** Quá trình failover được quản lý mượt mà bởi Global Accelerator ở cấp độ định tuyến mạng, nghĩa là không cần thay đổi DNS thủ công và tốc độ failover diễn ra cực kỳ nhanh.
* **Triển khai độc lập:** Hạ tầng backend ở mỗi region cần được triển khai và scale độc lập để đảm bảo nó có thể gánh được toàn bộ tải nếu failover xảy ra.
* **Dữ liệu ở nhiều region:** DynamoDB Global Tables là thành phần then chốt đảm bảo dữ liệu luôn có mặt ở cả hai region, cho phép ứng dụng hoạt động bất kể region nào đang xử lý request.
* **RTO và RPO:** Kiến trúc này hướng tới Recovery Time Objective (RTO - thời gian phục hồi) và Recovery Point Objective (RPO - lượng dữ liệu có thể mất) rất thấp, mang lại khả năng sẵn sàng gần như liên tục.
* **Active-Active và Active-Passive:** Kiến trúc này hỗ trợ mô hình Active-Active (cả 2 region cùng phục vụ traffic) nhưng cũng có thể hoạt động như Active-Passive nếu cấu hình dồn toàn bộ traffic vào một region cho đến khi có lỗi.
