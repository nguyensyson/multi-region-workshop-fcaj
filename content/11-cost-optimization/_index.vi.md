---
title: "Cost Optimization"
date: 2026-06-03
weight: 11
chapter: false
pre: " <b> 11. </b> "
---

# 11. Cost Optimization

Phần này mô tả cách phân tích và tối ưu chi phí cho dự án MoneyTrack. Mặc dù kiến trúc multi-region mang lại khả năng High Availability (Sẵn sàng cao) và Disaster Recovery (Khôi phục thảm họa) tuyệt vời, nhưng chi phí sẽ cao hơn đáng kể so với mô hình single-region. Tối ưu chi phí giúp chúng ta tối đa hóa giá trị từng đồng bỏ ra mà không làm suy giảm năng lực vận hành.

## 11.1 Chi phí của Multi-region

Khi triển khai 2 region, rất nhiều thành phần phải được nhân đôi: VPC, subnet, NAT Gateway, ALB, ECS Fargate, VPC Endpoint, và hạ tầng monitoring. Chi phí không chỉ đến từ tài nguyên tính toán (compute) mà còn bị đội lên rất nhiều từ networking, truyền tải dữ liệu (data transfer) và đồng bộ dữ liệu (replication).

### Các nhóm chi phí chính trong MoneyTrack

*   **Frontend - AWS Amplify:** Chi phí phát sinh từ số phút build, dung lượng lưu trữ hosting và data transfer (dữ liệu phục vụ người dùng). Nếu build thường xuyên hoặc tạo nhiều branch preview, chi phí sẽ tăng.
*   **Backend - ECS Fargate:** Tính phí dựa trên số lượng vCPU và memory mà task sử dụng nhân với thời gian chạy. Việc duy trì số lượng task tối thiểu chạy 24/7 ở *cả hai* region sẽ làm tăng gấp đôi chi phí compute cơ sở. Đặt desired count quá cao gây lãng phí lớn.
*   **Load Balancing - ALB:** Phải trả phí thuê bao theo giờ cho mỗi ALB và phí xử lý dữ liệu (LCU). Tạo nhiều ALB cho các môi trường dev/staging sẽ làm chi phí tăng nhanh.
*   **Networking - NAT Gateway, VPC Endpoint, Data Transfer:** NAT Gateway là thủ phạm gây tốn kém phổ biến nhất (tính phí theo giờ và theo lượng GB dữ liệu đi qua). Mỗi AZ một NAT Gateway giúp tăng HA nhưng tăng chi phí. VPC Endpoint cũng tính phí giờ nhưng giúp giảm tải cho NAT Gateway. Data transfer giữa 2 region cũng phát sinh phí.
*   **Database - DynamoDB Global Tables:** Chi phí gồm số lượng request đọc/ghi, lưu trữ, backup và replication. Global Tables đắt hơn bảng single-region vì dữ liệu phải được ghi và đồng bộ qua lại giữa các region.
*   **Security - AWS WAF & Secrets Manager:** WAF tính phí theo số Web ACL, số rule và số lượng request được kiểm tra. Secrets Manager tính phí lưu trữ từng secret và theo số lần gọi API.
*   **Monitoring - CloudWatch, Prometheus, Grafana:** CloudWatch Logs tính phí dựa trên lượng log đẩy lên (ingest) và lưu trữ. Ghi log quá chi tiết sẽ rất tốn kém. Prometheus và Grafana tính phí theo lượng metric, workspace và số lượng user.
*   **ECR - Docker Image Storage:** Tính phí lưu trữ image. Nếu không có chính sách xóa (lifecycle policy), image cũ sẽ tích tụ và làm tăng chi phí lưu trữ theo thời gian.

### Bảng tóm tắt chi phí

| Thành phần | Nguyên nhân phát sinh chi phí | Mức độ cần chú ý | Ghi chú |
| :--- | :--- | :--- | :--- |
| **AWS Amplify** | Build minutes, data transfer | Thấp | Thường không đáng kể trừ khi traffic cực lớn. |
| **ECS Fargate** | vCPU, Memory, thời gian chạy | Cao | Cần right-size (chọn cấu hình vừa đủ) và tận dụng Auto Scaling. |
| **ALB** | Phí theo giờ, dữ liệu xử lý | Trung bình | Gom chung ALB cho các môi trường non-prod nếu có thể. |
| **NAT Gateway** | Phí theo giờ, dữ liệu xử lý | Cao | Rất dễ phát sinh chi phí "ẩn" khổng lồ nếu không kiểm soát traffic. |
| **VPC Endpoint** | Phí theo giờ, dữ liệu xử lý | Trung bình | Giúp bảo mật tốt hơn và có thể bù trừ chi phí NAT Gateway. |
| **DynamoDB Global Tables** | Request đọc/ghi, Storage, Replication | Cao | Cấu hình partition key tối ưu; chi phí ghi (write) bị nhân đôi. |
| **AWS WAF** | Web ACL, Rule, Lượng request | Trung bình | Chỉ bật các rule thật sự cần thiết. |
| **Secrets Manager** | Số lượng secret, số API call | Thấp | Cần cache secret ở tầng application để giảm API call. |
| **CloudWatch Logs/Metrics** | Lượng log ingest, thời gian lưu giữ | Cao | Tránh log mức độ DEBUG/TRACE trên production. |
| **Prometheus/Grafana** | Lượng metrics, Active user | Trung bình | Tránh scrape các metric không mang lại giá trị vận hành. |
| **Amazon ECR** | Dung lượng lưu trữ | Thấp | Cần thiết lập Lifecycle policy để tự động dọn dẹp. |

---

## 11.2 Chiến lược tối ưu chi phí

Dưới đây là các phương pháp tối ưu chi phí cho kiến trúc MoneyTrack.

### Tối ưu ECS Fargate
*   **Right-sizing:** Chọn mức CPU/memory sát với nhu cầu thực tế của application.
*   **Auto Scaling:** Dùng ECS Service Auto Scaling để tự động tăng/giảm task theo tải (CPU, memory, request count) thay vì cấp phát dư thừa.
*   **Môi trường Non-Production:** Tắt hoặc giảm desired count xuống 0 ngoài giờ làm việc đối với môi trường dev/staging.
*   **Compute Savings Plans:** Nếu traffic production ổn định lâu dài, hãy mua Savings Plans để được giảm giá sâu.

### Tối ưu NAT Gateway và Network
*   **VPC Endpoint:** Ưu tiên dùng Gateway Endpoint cho DynamoDB và Interface Endpoint cho ECR, CloudWatch để traffic đi thẳng trong mạng AWS, không qua NAT Gateway.
*   **Môi trường Non-Production:** Có thể chấp nhận rủi ro giảm HA bằng cách chỉ dùng 1 NAT Gateway cho cả VPC ở môi trường dev để tiết kiệm.
*   Kiểm tra kỹ Route Table để chặn traffic nội bộ không cần thiết đi ra ngoài internet.

### Tối ưu DynamoDB Global Tables
*   **Capacity Mode:** Dùng On-Demand cho các workload khó đoán. Khi traffic đã ổn định, hãy chuyển sang Provisioned Capacity kết hợp Auto Scaling để tiết kiệm chi phí.
*   **Time-to-Live (TTL):** Bật TTL để tự động xóa dữ liệu tạm/quá hạn, giúp giảm chi phí lưu trữ.
*   Chỉ sử dụng Global Tables cho những dữ liệu thật sự cần phải có mặt ở cả 2 region.

### Tối ưu Logging và Monitoring
*   **Log Retention:** Cấu hình thời gian lưu log (retention) hợp lý (ví dụ: dev lưu 7 ngày, prod lưu 30 ngày). Không bao giờ để "Never Expire".
*   **Log Level:** Cấu hình ứng dụng chỉ xuất log ở mức INFO, WARN, ERROR.
*   Chỉ tạo các metric và alarm thực sự quan trọng và có khả năng hành động (actionable).

### Tối ưu AWS WAF, Secrets Manager và ECR
*   **Cache Secret:** Ứng dụng nên lưu cache các giá trị từ Secrets Manager trong RAM một khoảng thời gian ngắn thay vì gọi API cho mỗi request.
*   **ECR Lifecycle Policy:** Cấu hình luật tự động xóa các image cũ (ví dụ: chỉ giữ lại 30 image được tag gần nhất).
*   **Tối ưu Docker Image:** Dùng multi-stage build và base image nhỏ (như Alpine) để giảm dung lượng, giúp giảm phí lưu trữ và thời gian pull image.

### Tối ưu Môi trường Multi-Region
*   **Active-Passive:** Nếu chưa có nhu cầu chạy Active-Active, hãy cân nhắc cấu hình Active-Passive. Region dự phòng chỉ chạy ECS Fargate với desired count là 1 (hoặc rất thấp) và tự động scale lên khi xảy ra failover, giúp tiết kiệm lượng lớn chi phí compute.

### Quản lý bằng Tagging và Budget
*   **Tagging:** Gắn tag đầy đủ cho mọi resource (ví dụ: `Project=MoneyTrack`, `Environment=prod`, `Region=primary`).
*   **AWS Budgets:** Thiết lập ngân sách và cảnh báo qua email/Slack khi chi phí vượt ngưỡng dự kiến.
*   **AWS Cost Explorer:** Thường xuyên sử dụng để phân tích chi phí theo từng service, region và tag.

**Luồng Tối ưu:**
```txt
Collect metrics → Analyze cost by service/region → Identify high-cost components → Apply optimization → Monitor impact
```

### Bảng đề xuất tối ưu và Trade-off

| Thành phần | Vấn đề chi phí | Cách tối ưu | Trade-off (Sự đánh đổi) |
| :--- | :--- | :--- | :--- |
| **ECS Fargate** | Chi phí compute nền (baseline) cao | Giảm task count tối thiểu, phụ thuộc nhiều vào Auto Scaling | Hệ thống phản ứng chậm hơn với đợt tăng traffic đột ngột (cold start). |
| **NAT Gateway** | Phí theo giờ và phí xử lý cao | Dùng 1 NAT Gateway cho toàn bộ VPC (thay vì mỗi AZ một cái) | Mất tính High Availability ở tầng mạng khi ra internet. |
| **DynamoDB** | Phí đồng bộ (replication) cao | Chuyển sang Provisioned Capacity + Auto Scaling | Đòi hỏi tinh chỉnh kỹ; traffic tăng quá nhanh có thể bị throttling. |
| **CloudWatch Logs**| Phí lưu trữ log cao | Giảm thời gian lưu log (VD: giữ 7 ngày) | Thiếu dữ liệu lịch sử nếu cần audit hoặc debug lỗi cũ. |
| **ECR** | Phí lưu trữ image cao | Dùng Lifecycle policy xóa mạnh tay image cũ | Hạn chế khả năng rollback về các version quá xa trong quá khứ. |

## Kết luận

Mô hình Multi-region giúp MoneyTrack đạt độ tin cậy tối đa, nhưng đòi hỏi sự quản lý chi phí nghiêm ngặt. Các thành phần "ngốn" tiền nhiều nhất thường là ECS Fargate, NAT Gateway, DynamoDB Global Tables và CloudWatch Logs.

Nguyên tắc tối quan trọng: **Tối ưu chi phí phải luôn được cân bằng với High Availability, Security và Disaster Recovery.** Đừng bao giờ tiết kiệm bằng cách tắt bỏ các thành phần bảo vệ cốt lõi trên môi trường production. Thay vào đó, hãy sử dụng metrics, tagging, AWS Budgets và review kiến trúc định kỳ để duy trì một hệ thống vừa mạnh mẽ vừa tối ưu chi phí.
