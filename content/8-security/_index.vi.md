---
title: "Security"
date: 2026-06-03
weight: 8
chapter: false
pre: " <b> 8. </b> "
---

# 8. Security

Phần này mô tả cách dự án MoneyTrack bảo mật hệ thống theo nhiều lớp. Một tư thế bảo mật vững chắc đạt được bằng cách áp dụng nguyên tắc phòng thủ theo chiều sâu (defense-in-depth) trên các tầng mạng, ứng dụng, định danh và dữ liệu.

## 8.1 Network Security

Bảo mật mạng giúp giới hạn bề mặt tấn công bằng cách kiểm soát chặt chẽ luồng traffic đi vào và di chuyển bên trong môi trường AWS.

*   **VPC độc lập:** Một VPC được thiết kế riêng cho từng region để cung cấp ranh giới mạng biệt lập.
*   **Phân chia Subnet:** Hệ thống mạng được chia thành public subnet và private subnet.
*   **Vị trí ALB:** Application Load Balancer (ALB) được đặt trong public subnet để nhận request từ internet.
*   **Cô lập Backend:** ECS Fargate chạy ứng dụng backend hoàn toàn trong private subnet, không expose trực tiếp ra internet.
*   **Security Group:** Hoạt động như các firewall stateful kiểm soát traffic vào/ra giữa ALB, ECS Fargate và các dịch vụ AWS khác.
    *   ALB chỉ cho phép traffic HTTP/HTTPS từ bên ngoài internet.
    *   ECS Fargate được cấu hình để *chỉ* nhận traffic từ Security Group của ALB, tuân thủ nguyên tắc quyền tối thiểu.
*   **NAT Gateway:** Cho phép các tài nguyên trong private subnet truy cập internet khi cần (ví dụ: pull package hoặc gọi external API), mà không cho phép kết nối inbound từ internet đi vào.
*   **VPC Endpoint:** Giúp ECS Fargate truy cập DynamoDB thông qua một private network path, hạn chế traffic đi ra public internet.
*   **NACLs:** Network Access Control Lists (NACL) có thể được dùng như một lớp kiểm soát stateless bổ sung ở cấp độ subnet.
*   **Multi-AZ:** Thiết kế mạng Multi-AZ giúp tăng tính sẵn sàng và giảm rủi ro khi một AZ gặp lỗi.

**Luồng Network:**
```txt
Internet → ALB Public Subnet → ECS Fargate Private Subnet → VPC Endpoint → DynamoDB
```

**Điểm cần nhấn mạnh:**
*   Backend không bao giờ public trực tiếp.
*   Chỉ ALB là entry point (điểm vào) chính cho các request API tới backend.
*   Security Group phải được cấu hình tuân thủ nghiêm ngặt nguyên tắc quyền tối thiểu (least privilege).

---

## 8.2 Application Security với AWS WAF

Bảo mật ứng dụng bảo vệ logic của ứng dụng khỏi các payload và cuộc tấn công độc hại.

*   **Vị trí AWS WAF:** AWS WAF được đặt trước ALB để kiểm tra các request HTTP/HTTPS trước khi chúng đi vào ứng dụng backend.
*   **Giảm thiểu mối đe dọa:** WAF giúp giảm rủi ro từ các request độc hại như SQL injection, XSS (Cross-Site Scripting), bad bots hoặc các luồng traffic bất thường.
*   **Managed Rules:** Chúng ta sử dụng AWS Managed Rules để áp dụng nhanh các rule bảo vệ cơ bản chống lại các lỗ hổng phổ biến.
*   **Rate Limiting:** Rate-based rule được cấu hình để giới hạn số lượng request từ một địa chỉ IP trong một khoảng thời gian cụ thể, bảo vệ hệ thống khỏi các nỗ lực brute-force hoặc DDoS.
*   **Khả năng quan sát:** WAF log có thể được gửi về CloudWatch hoặc S3 để phân tích chi tiết. Có thể kết hợp WAF metrics với CloudWatch/Grafana để theo dõi số lượng request bị block hoặc allow.
*   **Trách nhiệm chung:** WAF là một lớp bảo vệ biên quan trọng, nhưng application backend vẫn cần validate input và xử lý authentication/authorization ở tầng code. WAF không thay thế hoàn toàn các thực hành viết code bảo mật.

**Luồng Application Security:**
```txt
User → Route 53 / Global Accelerator → AWS WAF → ALB → ECS Fargate
```

**Điểm cần nhấn mạnh:**
*   WAF là lớp bảo vệ đầu tiên trước backend.
*   ALB chỉ nhận request đã đi qua lớp kiểm tra của WAF.
*   WAF giúp giảm tải và giảm nguy cơ tấn công trực tiếp vào application.

---

## 8.3 Data Encryption & Secrets Management

Dữ liệu cần được bảo vệ cả khi đang truyền tải và khi lưu trữ.

**Encryption in Transit (Mã hóa khi truyền tải):**
*   Người dùng truy cập cả frontend và backend hoàn toàn thông qua HTTPS.
*   ALB sử dụng SSL/TLS certificate (quản lý bởi ACM) để mã hóa traffic từ client.
*   API domain `api.moneytrack.com` bắt buộc sử dụng HTTPS.
*   Kết nối giữa các service và AWS API đều dùng giao thức HTTPS.

**Encryption at Rest (Mã hóa khi lưu trữ):**
*   **DynamoDB:** Mã hóa dữ liệu mặc định khi lưu trữ. Khi DynamoDB Global Tables replicate dữ liệu giữa các region, dữ liệu vẫn luôn được bảo vệ bằng encryption.
*   **Amazon ECR:** Lưu trữ Docker image. Nên bật tính năng image scanning và encryption mặc định.
*   **CloudWatch Logs:** Lưu trữ application log an toàn, cần cấu hình retention (thời gian lưu giữ) phù hợp.
*   **Terraform State:** S3 bucket chứa Terraform state cần được bật encryption và versioning, vì state file có thể chứa thông tin nhạy cảm.

**Quản lý Secrets:**
*   **AWS Secrets Manager:** Lưu trữ an toàn các secret, token, hoặc thông tin nhạy cảm.
*   **Không Hard-code:** Không bao giờ hard-code secret trong source code, Docker image, hoặc code Terraform.
*   **Truy xuất an toàn:** ECS Fargate lấy secret động trong lúc chạy (runtime) thông qua quyền được cấp bởi IAM Task Role.
*   **Rotation:** Có thể cấu hình secret rotation để tự động cập nhật thông tin xác thực định kỳ.

**Identity and Access Management (IAM):**
*   **IAM Roles:** ECS task sử dụng IAM Role để truy cập các dịch vụ AWS, thay vì dùng access key hard-code có rủi ro cao.
*   **CI/CD Pipeline:** Pipeline chỉ được cấp các quyền tối thiểu cần thiết để build, push image và deploy.
*   **Least Privilege:** Áp dụng nghiêm ngặt nguyên tắc quyền tối thiểu cho tất cả các service roles.
*   **Phân tách môi trường:** Tách biệt quyền IAM theo từng environment như dev, staging, production để tránh ảnh hưởng chéo.

**Luồng Identity:**
```txt
ECS Fargate → IAM Task Role → Secrets Manager / DynamoDB / CloudWatch
```

---

## Security Checklist

Bảng checklist dưới đây tóm tắt cách áp dụng security ở từng lớp trong kiến trúc MoneyTrack.

| Layer | Service | Cách áp dụng trong MoneyTrack |
| :--- | :--- | :--- |
| **Network** | VPC, Private subnet, Security Group, VPC Endpoint | VPC riêng biệt, backend giấu trong private subnet, SG rule chặt chẽ (chỉ nhận từ ALB), kết nối private tới DynamoDB qua Endpoint. |
| **Application** | AWS WAF, ALB, HTTPS | WAF lọc request độc hại trước ALB, bắt buộc dùng HTTPS cho mọi giao tiếp client. |
| **IAM** | IAM Roles | Dùng IAM Task Role cho ECS, áp dụng least privilege cho mọi service và CI/CD pipeline. |
| **Secret** | AWS Secrets Manager | Lưu trữ tập trung và an toàn API key, token; task tự động lấy khi cần. |
| **Data** | DynamoDB, S3, CloudWatch | DynamoDB encryption at rest, mã hóa S3 state bucket, cấu hình CloudWatch log retention hợp lý. |
| **Monitoring** | WAF log, CloudWatch Logs, Grafana dashboard | Theo dõi lượng request bị block qua WAF, phân tích log để tìm hành vi bất thường. |

## Kết luận

Tóm tắt:
*   Security của dự án MoneyTrack được thiết kế theo nguyên tắc nhiều lớp.
*   **Network security** giúp giới hạn bề mặt tấn công thông qua việc cô lập mạng.
*   **AWS WAF** bảo vệ application trước các request độc hại.
*   **IAM và Secrets Manager** giúp quản lý quyền truy cập và thông tin nhạy cảm một cách an toàn.
*   **Encryption** bảo vệ dữ liệu cả khi truyền tải và khi lưu trữ.
*   **Monitoring** cung cấp khả năng quan sát để phát hiện sớm các hành vi bất thường.
