---
title: "Disaster Recovery"
date: 2026-06-03
weight: 9
chapter: false
pre: " <b> 9. </b> "
---

# 9. Disaster Recovery

Phần này mô tả chiến lược Khôi phục sau thảm họa (Disaster Recovery - DR) của dự án MoneyTrack, trình bày chi tiết cách hệ thống phản ứng khi xảy ra lỗi và quy trình khôi phục dịch vụ.

## 9.1 Kịch bản một AZ bị lỗi

**Mô tả:**
*   Một Availability Zone (AZ) trong một region gặp sự cố (ví dụ: mất điện, đứt cáp).
*   Các ECS Fargate task hoặc subnet nằm trong AZ đó có thể bị ảnh hưởng và ngừng hoạt động.
*   Application Load Balancer (ALB) health check phát hiện các target trong AZ bị lỗi không còn healthy.
*   ALB ngừng route traffic đến các ECS task bị lỗi và tự động chuyển toàn bộ request sang các task healthy ở AZ còn lại.
*   ECS Service sẽ cố gắng tạo lại task mới ở AZ còn hoạt động nếu AZ đó còn đủ capacity (tài nguyên).
*   DynamoDB là managed service nên vẫn duy trì khả năng phục vụ bình thường trong region đó.
*   *Lưu ý:* Nếu NAT Gateway chỉ được triển khai ở một AZ để tiết kiệm chi phí, nó sẽ trở thành điểm lỗi (single point of failure). Trong môi trường production, nên triển khai NAT Gateway ở mọi AZ.

**Luồng xử lý lỗi:**
```txt
AZ A lỗi → ALB health check phát hiện target unhealthy → Traffic chuyển sang ECS task ở AZ B → Hệ thống tiếp tục phục vụ
```

**Kết quả:**
*   Người dùng có thể chỉ bị ảnh hưởng rất nhẹ hoặc hoàn toàn không nhận thấy sự gián đoạn nếu hệ thống có đủ số lượng task dự phòng ở AZ còn lại.
*   Đây là kịch bản được xử lý hoàn toàn tự động nhờ vào thiết kế kiến trúc Multi-AZ.

---

## 9.2 Kịch bản một Region bị lỗi

**Mô tả:**
*   Một region chính gặp sự cố nghiêm trọng, ảnh hưởng đến ALB, ECS Fargate, VPC hoặc các service cốt lõi khác trong region đó.
*   AWS Global Accelerator hoặc Route 53 health check phát hiện endpoint của region bị lỗi không còn healthy.
*   Traffic mới từ người dùng được tự động chuyển sang region còn hoạt động.
*   Hạ tầng backend (ECS tasks) đã được triển khai sẵn ở region dự phòng sẽ tiếp tục xử lý request.
*   DynamoDB Global Tables đảm bảo dữ liệu đã được replicate sang region còn lại để sẵn sàng phục vụ.
*   *Lưu ý:* Để quá trình khôi phục diễn ra suôn sẻ, các thành phần phụ trợ như Secrets Manager (chứa secret), ECR (chứa Docker image) hoặc cấu hình Terraform phải được đồng bộ sẵn ở region dự phòng.

**Luồng xử lý lỗi:**
```txt
Region A lỗi → Global Accelerator đánh dấu endpoint unhealthy → Traffic chuyển sang Region B → ALB Region B → ECS Fargate Region B → DynamoDB Global Tables
```

**Kết quả:**
*   Hệ thống tiếp tục phục vụ người dùng từ region còn healthy.
*   Một số request đang được xử lý dở dang tại region lỗi ngay tại thời điểm xảy ra sự cố có thể bị thất bại, yêu cầu client (frontend) phải retry.
*   Dữ liệu có thể có độ trễ replicate nhỏ (vài phần trăm giây), nhưng DynamoDB Global Tables đảm bảo dữ liệu ở region dự phòng gần như là mới nhất.

---

## 9.3 RTO / RPO dự kiến

*   **RTO (Recovery Time Objective):** Thời gian tối đa hệ thống cần để khôi phục dịch vụ sau sự cố.
*   **RPO (Recovery Point Objective):** Lượng dữ liệu tối đa có thể bị mất hoặc chưa kịp đồng bộ tại thời điểm xảy ra sự cố.

Với dự án MoneyTrack, mục tiêu RTO/RPO dự kiến như sau:

| Kịch bản | RTO dự kiến | RPO dự kiến | Giải thích |
| :--- | :--- | :--- | :--- |
| **Một ECS task bị lỗi** | Vài giây đến vài phút | Gần như 0 | ECS tạo task mới thay thế, ALB chỉ route đến các target healthy. |
| **Một AZ bị lỗi** | Vài phút | Gần như 0 | ALB tự động chuyển traffic sang các AZ còn lại trong cùng một region. |
| **Một Region bị lỗi** | Vài phút | Vài giây đến vài phút | Global Accelerator chuyển traffic sang region healthy, DynamoDB Global Tables cung cấp dữ liệu đã được replicate. |

*Lưu ý: Các giá trị RTO/RPO này là mục tiêu thiết kế dự kiến, không phải cam kết tuyệt đối (SLA). RTO/RPO thực tế phụ thuộc vào cấu hình health check interval, chiến lược failover, độ trễ replication, mức độ sẵn sàng của CI/CD và bản chất thực tế của sự cố.*

---

## 9.4 Chiến lược failover

Dự án có thể áp dụng các chiến lược failover khác nhau tùy thuộc vào chi phí và mục tiêu vận hành:

*   **Active-Active:** Cả 2 region cùng phục vụ traffic đồng thời. Global Accelerator định tuyến user đến region phù hợp nhất. Khi một region lỗi, traffic được chuyển hết sang region còn lại.
    *   *Ưu điểm:* RTO thấp nhất, tận dụng tối đa tài nguyên ở cả 2 region.
    *   *Hạn chế:* Chi phí cao hơn, đòi hỏi xử lý tính nhất quán (consistency) và xung đột dữ liệu (conflict) phức tạp hơn.
*   **Active-Passive:** Một region chính phục vụ toàn bộ traffic. Region còn lại ở trạng thái chờ (dự phòng). Khi region chính lỗi, traffic mới được chuyển sang.
    *   *Ưu điểm:* Dễ kiểm soát hơn, có thể tối ưu chi phí nếu chạy quy mô nhỏ ở region dự phòng.
    *   *Hạn chế:* RTO có thể cao hơn nếu backend ở region dự phòng cần thời gian để scale up (tăng capacity) để gánh toàn bộ tải.

**Định hướng của MoneyTrack:**
Với kiến trúc hiện tại, backend và database được triển khai sẵn ở cả 2 region. Global Accelerator hỗ trợ điều hướng traffic linh hoạt và DynamoDB Global Tables đồng bộ dữ liệu hai chiều. Kiến trúc này hỗ trợ cả hai mô hình trên. Failover nên được kiểm tra định kỳ thông qua các kịch bản diễn tập giả lập sự cố (Game Days).

**Luồng Failover:**
```txt
Health check failed → Endpoint marked unhealthy → Stop routing to failed region → Route traffic to healthy region → Monitor recovery
```

---

## 9.5 Quy trình khôi phục hệ thống

Khi xảy ra sự cố, quy trình xử lý chung diễn ra theo các bước sau:

### Bước 1: Phát hiện sự cố
*   Phát hiện bất thường thông qua CloudWatch Alarm, ALB health check, event của ECS service, metrics của WAF/Global Accelerator hoặc trên Grafana dashboard (ví dụ: lỗi HTTP 5xx tăng vọt, ALB báo unhealthy target, ECS task bị stopped, DynamoDB bị throttling, hoặc endpoint của region bị unhealthy).

### Bước 2: Xác định phạm vi ảnh hưởng
*   Kiểm tra xem nguyên nhân cốt lõi nằm ở application code, một ECS task cụ thể, ALB, vấn đề mạng, database, lỗi một AZ hay sập toàn bộ region.
*   So sánh metrics giữa 2 region để xác định rõ region nào còn đang healthy.

### Bước 3: Kích hoạt failover (nếu cần)
*   **Lỗi AZ:** Để ALB và ECS tự động chuyển traffic và thay thế task.
*   **Lỗi Region:** Xác nhận Global Accelerator đã chuyển traffic sang region còn healthy.
*   Nếu dùng Active-Passive, có thể cần tăng `desired_count` của ECS task ở region dự phòng để đảm bảo đủ khả năng xử lý tải.

### Bước 4: Khôi phục tài nguyên lỗi
*   Nếu lỗi do application: Rollback Docker image về phiên bản cũ ổn định hoặc redeploy ECS service.
*   Nếu lỗi network: Kiểm tra lại cấu hình Security Group, route table, NAT Gateway, VPC Endpoint.
*   Nếu lỗi dữ liệu: Kiểm tra trạng thái replication của DynamoDB Global Tables.
*   Nếu infrastructure bị xóa nhầm hoặc sửa sai: Dùng Terraform để apply lại cấu hình chuẩn.

### Bước 5: Xác minh hệ thống sau khôi phục
*   Kiểm tra các API health check endpoint.
*   Kiểm tra ALB target group xem các instance đã healthy chưa.
*   Kiểm tra trạng thái ECS task có đang RUNNING ổn định không.
*   Kiểm tra application log trong CloudWatch để tìm lỗi ẩn.
*   Kiểm tra các dashboard trên Grafana xem metrics đã bình thường chưa.
*   Kiểm tra dữ liệu DynamoDB ở cả 2 region.

### Bước 6: Failback (Khôi phục về trạng thái ban đầu)
*   Khi region bị lỗi đã được sửa chữa và hoạt động bình thường, **không** chuyển toàn bộ traffic về ngay lập tức.
*   Cần kiểm tra kỹ backend, database replication, log và metrics ở region vừa được sửa.
*   Sau khi ổn định, chuyển traffic lại từ từ (ví dụ: dùng traffic dials của Global Accelerator) và theo dõi sát sao.

### Bước 7: Post-incident review (Đánh giá sau sự cố)
*   Ghi chép lại nguyên nhân gốc rễ (root cause) của sự cố.
*   Đánh giá xem RTO/RPO thực tế có đạt mục tiêu thiết kế không.
*   Cập nhật lại tài liệu runbook, điều chỉnh alert threshold hoặc sửa code/infrastructure Terraform nếu cần để khắc phục triệt để.
*   Bổ sung test case để tránh lặp lại lỗi tương tự trong tương lai.

## Kết luận

Chiến lược Disaster Recovery của MoneyTrack dựa trên sự kết hợp giữa kiến trúc Multi-AZ, Multi-Region, sức mạnh định tuyến của Global Accelerator, ALB health check, tính bền bỉ của ECS Fargate và khả năng đồng bộ của DynamoDB Global Tables.
*   Khi một AZ lỗi, hệ thống ưu tiên tự động chuyển traffic nội bộ sang AZ còn healthy.
*   Khi một Region lỗi, traffic được chuyển hướng ở cấp độ toàn cầu sang region còn hoạt động.
*   Một quy trình khôi phục thành công đòi hỏi sự kết hợp chặt chẽ giữa monitoring, cơ chế failover tự động, quy trình kiểm tra, khả năng rollback và việc đúc kết kinh nghiệm sau mỗi sự cố.
