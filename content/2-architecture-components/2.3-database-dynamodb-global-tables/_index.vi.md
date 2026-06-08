---
title: "Database - DynamoDB Global Tables"
date: 2026-06-03
weight: 3
chapter: false
pre: " <b> 2.3 </b> "
---

# 2.3 Database - DynamoDB Global Tables

### DynamoDB là gì?
Amazon DynamoDB là một cơ sở dữ liệu NoSQL key-value được quản lý hoàn toàn, serverless, được thiết kế để chạy các ứng dụng hiệu năng cao ở mọi quy mô. Nó cung cấp tính năng bảo mật tích hợp, sao lưu liên tục, tự động sao chép multi-region, caching in-memory và các công cụ import/export dữ liệu.

### DynamoDB Global Tables là gì?
**DynamoDB Global Tables** cung cấp một giải pháp được quản lý hoàn toàn để triển khai một cơ sở dữ liệu multi-region, multi-active. Bạn có thể chỉ định các AWS Region mà bạn muốn bảng của mình khả dụng và DynamoDB sẽ tự động sao chép các thay đổi dữ liệu đang diễn ra tới tất cả các region đó.

### Vai trò của DynamoDB trong hệ thống
DynamoDB đóng vai trò là kho lưu trữ dữ liệu chính cho ứng dụng MoneyTrack, lưu giữ hồ sơ người dùng, hồ sơ giao dịch và số dư tài khoản. Global Tables đảm bảo rằng dữ liệu này luôn khả dụng và được đồng bộ hóa nhất quán trên các region triển khai chính và phụ của chúng ta.

### Cách dữ liệu được replicate giữa các region
Khi một ECS Fargate task ở Region A ghi dữ liệu vào bản sao DynamoDB cục bộ của nó, DynamoDB sẽ tự động truyền thay đổi đó đến bản sao ở Region B, thường trong vòng chưa tới một giây. Việc sao chép multi-active này có nghĩa là các ứng dụng có thể đọc và ghi dữ liệu vào bản sao cục bộ của chúng, giúp giảm độ trễ.

### Lý do chọn DynamoDB Global Tables
* **Độ trễ thấp (Low latency):** Ứng dụng truy cập dữ liệu từ region gần nhất, cung cấp hiệu suất mili-giây có một chữ số.
* **Multi-region Replication:** Sao chép active-active được quản lý hoàn toàn mà không cần xây dựng hoặc duy trì logic sao chép tùy chỉnh.
* **High Availability & Disaster Recovery:** Nếu một region bị cô lập hoặc suy giảm hiệu năng, ứng dụng có thể chuyển đổi dự phòng (failover) liền mạch sang một region khác và tiếp tục sử dụng bản sao cục bộ, cập nhật của nó.
* **Serverless:** Không có máy chủ nào cần cấp phát hoặc quản lý. Cơ sở dữ liệu tự động mở rộng dung lượng dựa trên pattern lưu lượng truy cập.

### Lưu lưu ý triển khai
* **Partition Key & Sort Key:** Thiết kế schema cẩn thận là rất quan trọng đối với hiệu suất NoSQL. Chọn các partition key giúp phân phối lưu lượng truy cập đồng đều để tránh các "hot partition".
* **Capacity Mode:** Chọn giữa chế độ On-Demand (trả tiền cho mỗi request) hoặc Provisioned capacity dựa trên khả năng dự đoán của workload.
* **Conflict Handling:** Cần hiểu rằng Global Tables sử dụng cơ chế đối chiếu "last writer wins" (người ghi cuối cùng thắng) cho các bản cập nhật đồng thời vào cùng một item trên các region khác nhau.
* **Backup & Recovery:** Sử dụng Point-in-Time Recovery (PITR) để bảo vệ khỏi các thao tác ghi hoặc xóa vô tình.
* **Encryption:** Kích hoạt mã hóa ở trạng thái nghỉ (encryption at rest) bằng AWS KMS để bảo mật dữ liệu tài chính nhạy cảm.
