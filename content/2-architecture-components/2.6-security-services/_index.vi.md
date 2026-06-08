---
title: "Security services - WAF, IAM, Secrets Manager"
date: 2026-06-03
weight: 6
chapter: false
pre: " <b> 2.6 </b> "
---

# 2.6 Security services - WAF, IAM, Secrets Manager

### AWS WAF
AWS WAF (Web Application Firewall) giúp bảo vệ các ứng dụng web hoặc API của bạn chống lại các web exploit và bot phổ biến có thể ảnh hưởng đến tính khả dụng, làm giảm bảo mật hoặc tiêu thụ quá mức tài nguyên. Trong kiến trúc này, WAF được liên kết với Application Load Balancer để kiểm tra các request HTTP/HTTPS đến trước khi chúng tiếp cận ứng dụng backend.

### IAM (Identity and Access Management)
AWS IAM cho phép bạn quản lý quyền truy cập vào các dịch vụ và tài nguyên AWS một cách an toàn. Nó được dùng để quản lý quyền theo nguyên tắc quyền tối thiểu (least privilege)—chỉ cấp những quyền cần thiết để thực hiện một công việc.

### AWS Secrets Manager
AWS Secrets Manager giúp bạn bảo vệ các secret cần thiết để truy cập ứng dụng, dịch vụ và tài nguyên CNTT của mình. Nó cho phép bạn dễ dàng xoay vòng (rotate), quản lý và truy xuất thông tin xác thực cơ sở dữ liệu, API key và các secret khác trong suốt vòng đời của chúng. Chúng ta sử dụng nó để lưu trữ an toàn các item như access token của cơ sở dữ liệu hoặc API key của bên thứ ba.

### Bảo vệ hệ thống theo nhiều lớp
Các dịch vụ này phối hợp với nhau để bảo vệ hệ thống ở nhiều lớp:
* **Lớp biên (WAF):** Chặn traffic độc hại, SQL injection và các cuộc tấn công cross-site scripting ngay tại biên (edge).
* **Lớp định danh (IAM):** Đảm bảo rằng chỉ các thực thể được ủy quyền (như các ECS task cụ thể hoặc GitHub Actions) mới có thể tương tác với các tài nguyên AWS.
* **Lớp dữ liệu (Secrets Manager):** Ngăn chặn việc các thông tin xác thực nhạy cảm bị hardcode trong source code hoặc bị lộ trong các biến môi trường.

### Lý do sử dụng các security services này trong kiến trúc production
Một thế trận bảo mật vững chắc là điều bắt buộc đối với các workload production. Việc chỉ dựa vào các ranh giới mạng (như VPC) là không đủ. Áp dụng defense-in-depth (bảo vệ chiều sâu) bằng cách sử dụng WAF cho các mối đe dọa ở lớp ứng dụng, các role IAM nghiêm ngặt để kiểm soát truy cập và Secrets Manager để quản lý thông tin xác thực giúp giảm thiểu phạm vi ảnh hưởng (blast radius) của bất kỳ sự cố bảo mật tiềm ẩn nào.

### Lưu ý triển khai
* **IAM role cho ECS task:** Tạo các IAM role riêng biệt cho ECS task của bạn (Task Role cho các quyền của ứng dụng, Task Execution Role để ECS agent pull image và ghi log). Đảm bảo chúng chỉ có quyền truy cập vào các tài nguyên cụ thể mà chúng cần.
* **Secret rotation:** Cấu hình tự động xoay vòng cho các secret trong Secrets Manager ở những nơi có thể để tăng cường bảo mật.
* **WAF rule & Managed rule group:** Bắt đầu với AWS Managed Rules (như Core rule set) cho WAF để ngay lập tức bảo vệ hệ thống khỏi các lỗ hổng phổ biến.
* **Logging:** Kích hoạt tính năng logging của WAF để giám sát các request bị từ chối và tinh chỉnh các rule bảo mật của bạn theo thời gian.
