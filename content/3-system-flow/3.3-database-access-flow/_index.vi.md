---
title: "Luồng truy cập database"
date: 2026-06-03
weight: 3
chapter: false
pre: " <b> 3.3 </b> "
---

# 3.3 Luồng truy cập database

Luồng này minh họa cách các backend task tương tác bảo mật với cơ sở dữ liệu và xử lý việc sao chép dữ liệu giữa các region.

### Chi tiết luồng

```txt
ECS Fargate → IAM Task Role → VPC Endpoint → DynamoDB Global Tables → Replication to another Region → Response
```

1. **Khởi tạo Request:** Một **ECS Fargate** task xử lý request cần đọc hoặc ghi dữ liệu.
2. **Lấy Secret (Tùy chọn):** Nếu application cần external API keys hoặc credential cơ sở dữ liệu cụ thể, nó sẽ lấy các secret này một cách bảo mật từ AWS Secrets Manager.
3. **Phân quyền (Authorization):** ECS task sử dụng **IAM Task Role** được gán cho nó để xác thực và cấp quyền cho request đến DynamoDB, đảm bảo nó có đúng quyền hạn.
4. **Truy cập Private:** Request đến cơ sở dữ liệu được định tuyến thông qua **VPC Endpoint** (cụ thể là Gateway Endpoint cho DynamoDB). Điều này giữ cho traffic hoàn toàn nằm trong mạng private của AWS thay vì đi ra public internet.
5. **Lưu trữ & Replicate dữ liệu:** Request đến được **DynamoDB Global Tables**. Dữ liệu được đọc hoặc ghi vào table ở region cục bộ. Nếu đó là thao tác ghi, DynamoDB sẽ tự động replicate (sao chép) các thay đổi sang region còn lại một cách bất đồng bộ.
6. **Kết quả:** Response từ DynamoDB được trả về cho ECS Fargate, sau đó task này định dạng response cuối cùng để trả về cho frontend/user.

### Các khái niệm chính

* **Vì sao dùng IAM Role thay vì hard-code credential:** Hard-code credential gây ra rủi ro bảo mật nghiêm trọng. Việc sử dụng IAM Task Role cấp các quyền tạm thời, được quản lý an toàn trực tiếp cho ECS task, tuân thủ các best practice về bảo mật.
* **Vì sao dùng VPC Endpoint để truy cập DynamoDB:** Truy cập DynamoDB thông qua VPC Endpoint đảm bảo traffic giữa VPC và DynamoDB không đi ra khỏi mạng Amazon. Điều này giúp tăng cường bảo mật và giảm chi phí truyền tải dữ liệu.
* **DynamoDB Global Tables hỗ trợ multi-region như thế nào:** Global Tables cung cấp một cơ sở dữ liệu multi-active được quản lý hoàn toàn. Các thao tác ghi thực hiện ở một region sẽ tự động được truyền đến các bản sao ở các region khác, đảm bảo tính sẵn sàng cao và hiệu suất đọc/ghi cục bộ trên toàn cầu.
* **Lưu ý triển khai:** Khi sử dụng Global Tables, cần lưu ý về *eventual consistency* (nhất quán cuối cùng - các bản sao có thể có độ trễ nhỏ). Hiểu rõ cơ chế *conflict handling* (xử lý xung đột - thường là "last writer wins"). Thiết kế *partition key* hợp lý để đảm bảo phân phối dữ liệu đồng đều. Đảm bảo dữ liệu được bảo vệ bằng *encryption* (mã hóa) và có tính năng *backup* (sao lưu).
