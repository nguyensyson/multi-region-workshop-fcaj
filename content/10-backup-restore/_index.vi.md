---
title: "Backup & Restore"
date: 2026-06-03
weight: 10
chapter: false
pre: " <b> 10. </b> "
---

# 10. Backup & Restore

Phần này mô tả chiến lược backup (sao lưu) và restore (khôi phục) cho các thành phần quan trọng của dự án MoneyTrack. Việc có một chiến lược backup vững chắc giúp chúng ta phục hồi nhanh chóng sau các sự cố như xóa nhầm dữ liệu, deploy mã nguồn lỗi, hoặc hỏng cấu hình hạ tầng.

## 10.1 Backup DynamoDB

**Mô tả:**
*   DynamoDB là nơi lưu trữ dữ liệu chính của hệ thống MoneyTrack.
*   Dự án sử dụng DynamoDB Global Tables để replicate dữ liệu giữa nhiều region.
*   Mặc dù Global Tables hỗ trợ tính sẵn sàng và khả năng phục hồi ở cấp độ region, nó **không thể thay thế hoàn toàn backup**. Nếu ai đó vô tình xóa một record, thao tác xóa đó cũng sẽ lập tức được replicate sang region kia.
*   Chúng ta cần bật **Point-in-Time Recovery (PITR)** để có thể khôi phục dữ liệu về một thời điểm cụ thể (bất kỳ giây nào trong vòng 35 ngày qua) khi có lỗi thao tác, xóa nhầm hoặc ghi sai dữ liệu.
*   Chúng ta cũng có thể tạo **On-demand backup** (sao lưu theo yêu cầu) trước khi thực hiện các thay đổi lớn, ví dụ: trước khi migrate schema, import lượng lớn dữ liệu, hoặc release tính năng quan trọng.
*   Backup nên được quản lý theo lifecycle và retention policy (chính sách lưu giữ) phù hợp thông qua AWS Backup.

**Cách restore:**
1.  Khi phát hiện dữ liệu bị lỗi (xóa nhầm/ghi sai), sử dụng PITR để restore table về thời điểm ngay trước khi xảy ra lỗi.
2.  Table được restore luôn là một table **mới** (không ghi đè lên table cũ).
3.  Cần kiểm tra kỹ dữ liệu trong table mới sau khi restore.
4.  Nếu cần khôi phục toàn bộ, có thể chuyển cấu hình application sang sử dụng table mới. Nếu chỉ cần khôi phục một phần dữ liệu, có thể export các record cần thiết từ table mới và import lại vào table production hiện tại.
5.  Sau khi restore hoàn tất, cần thiết lập lại quá trình replicate Global Tables giữa các region.

**Luồng Restore:**
```txt
Data issue detected → Restore DynamoDB to selected point in time → Verify restored table → Switch application or import recovered data → Monitor replication
```

**Lưu ý vận hành:**
*   Không phụ thuộc hoàn toàn vào Global Tables như một cơ chế backup.
*   Bật mã hóa (encryption at rest) cho DynamoDB và cả bản backup.
*   Theo dõi DynamoDB metrics như throttling, latency, replication latency.
*   Kiểm tra định kỳ quy trình restore (diễn tập) để đảm bảo team biết cách khôi phục khi cần.
*   Phân quyền IAM cực kỳ chặt chẽ cho thao tác backup/restore.

---

## 10.2 Backup Terraform State

**Mô tả:**
*   Terraform state lưu trạng thái hiện tại của infrastructure trên AWS.
*   Đây là thành phần cốt lõi vì Terraform dựa vào state để biết resource nào đang được quản lý.
*   Trong dự án MoneyTrack, Terraform state được lưu ở **S3 remote backend** thay vì lưu local trên máy developer.
*   S3 bucket chứa state cần được bật **Versioning** để có thể khôi phục phiên bản state trước đó khi state bị hỏng hoặc ai đó apply cấu hình sai.
*   Nên bật encryption cho bucket chứa state vì state file thường chứa các thông tin nhạy cảm (như mật khẩu DB) dưới dạng plain text.
*   Dùng **DynamoDB để state locking**, tránh tình trạng nhiều người hoặc nhiều pipeline chạy `terraform apply` cùng một lúc làm hỏng state.
*   Cần hạn chế quyền truy cập vào state file.

**Cách restore:**
1.  Khi Terraform state bị lỗi, truy cập vào version history của S3 bucket chứa state.
2.  Chọn phiên bản state ổn định ngay trước thời điểm xảy ra lỗi.
3.  Restore object version đó để biến nó thành state hiện tại.
4.  Chạy lệnh `terraform plan` để kiểm tra sự khác biệt giữa state vừa được khôi phục và infrastructure thực tế đang chạy trên AWS.
5.  Chỉ chạy `terraform apply` sau khi đã review plan một cách cực kỳ cẩn thận.

**Luồng Restore:**
```txt
State issue detected → Check S3 version history → Restore previous state version → Run terraform plan → Review → Apply if needed
```

**Lưu ý vận hành:**
*   Tuyệt đối không chỉnh sửa state file thủ công nếu không thật sự am hiểu.
*   Không bao giờ commit file `terraform.tfstate` lên GitHub.
*   Luôn bật versioning và encryption cho S3 backend.
*   Dùng IAM least privilege cho người dùng hoặc pipeline truy cập state.
*   Với các resource đặc biệt quan trọng như DynamoDB, nên dùng lifecycle rule `prevent_destroy = true` trong Terraform để tránh vô tình xóa.

---

## 10.3 Versioning Docker Image trong ECR

**Mô tả:**
*   Backend của MoneyTrack được đóng gói thành các Docker image.
*   Docker image được lưu trữ trong **Amazon ECR**.
*   Mỗi lần CI/CD pipeline build backend, image cần được gắn tag với version rõ ràng.
*   **Không nên** chỉ dùng tag `latest` cho production vì nó khiến việc xác định phiên bản đang chạy rất khó khăn và làm quá trình rollback trở nên phức tạp.
*   Nên dùng các kiểu tag như:
    *   Git commit SHA (ví dụ: `a1b2c3d`)
    *   Release version (ví dụ: `v1.0.0`)
    *   Environment tag kết hợp timestamp (ví dụ: `prod-2026-06-03`)
*   Versioning Docker image tốt giúp rollback hệ thống ngay lập tức khi bản deploy mới gặp lỗi.

**Cách restore/rollback:**
1.  Khi phát hiện version backend mới gây lỗi, tìm lại tag của version image cũ ổn định trong ECR.
2.  Update ECS Task Definition để sử dụng lại image version cũ đó.
3.  Deploy lại ECS service với Task Definition vừa cập nhật.
4.  ALB health check sẽ đảm bảo traffic chỉ được route đến các task cũ đã khởi động thành công (healthy).
5.  Theo dõi CloudWatch Logs và Grafana dashboard sau khi rollback để đảm bảo hệ thống đã ổn định trở lại.

**Luồng Rollback:**
```txt
New deployment failed → Select previous stable image in ECR → Update ECS Task Definition → Redeploy ECS Service → Verify health check and logs
```

**Lưu ý vận hành:**
*   Không xóa các image cũ quá sớm.
*   Cấu hình ECR lifecycle policy để tự động dọn dẹp các image quá cũ, cân bằng giữa khả năng rollback và chi phí lưu trữ.
*   Bật tính năng image scanning để phát hiện lỗ hổng (vulnerability) trong Docker image.
*   Luôn ghi lại version image được deploy trong release note hoặc deployment log.
*   Với môi trường production, chỉ được phép rollback bằng image version đã được kiểm thử trước đó.

---

## Bảng tóm tắt Backup & Restore

| Thành phần | Cách backup / Versioning | Cách restore / Rollback | Lưu ý vận hành |
| :--- | :--- | :--- | :--- |
| **DynamoDB** | Bật Point-in-Time Recovery (PITR); tạo on-demand backup trước các thay đổi lớn. | Restore ra một table mới qua PITR; đổi config trỏ app sang table mới hoặc import dữ liệu bị mất lại. | Global Tables không thay thế backup; kiểm tra quy trình restore định kỳ; phân quyền IAM chặt chẽ. |
| **Terraform State** | Lưu trên S3 Remote Backend, bật Versioning và Encryption; dùng DynamoDB state locking. | Lấy lại phiên bản cũ từ S3 version history; chạy `terraform plan` để review kỹ khác biệt. | Không commit state lên Git; không sửa state thủ công; dùng `prevent_destroy` cho DB. |
| **Docker Image (ECR)** | Tag image theo Commit SHA hoặc Release Version khi build CI/CD; không dùng tag `latest`. | Update ECS Task Definition trỏ về image tag cũ ổn định và redeploy ECS Service. | Dùng ECR Lifecycle policy dọn dẹp image thừa; chỉ rollback về các bản đã được test. |

## Kết luận

Tóm tắt:
*   Backup & Restore giúp MoneyTrack giảm thiểu tối đa rủi ro khi dữ liệu, infrastructure hoặc quy trình deployment gặp sự cố.
*   **DynamoDB** cần cả Point-in-Time Recovery và on-demand backup.
*   **Terraform state** bắt buộc phải được lưu trên S3 có cấu hình versioning, encryption và khóa state (state locking).
*   **Docker image** trong ECR cần versioning rõ ràng để hỗ trợ rollback nhanh chóng và an toàn.
*   Quan trọng nhất: Quy trình restore cần được diễn tập và kiểm tra định kỳ trong thực tế, không chỉ nằm trên tài liệu.
