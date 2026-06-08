---
title: "Tổng quan Workshop"
weight: 1
chapter: false
pre: "<b>1. </b>"
---

# Tổng quan Workshop

## 1. Giới thiệu

Workshop này được xây dựng nhằm ghi lại quá trình thiết kế, triển khai và vận hành kiến trúc hệ thống cho dự án. Nội dung workshop cung cấp một cái nhìn tổng quan về toàn bộ dự án, bao gồm mục tiêu thiết kế, kiến trúc triển khai, các thành phần chính, luồng xử lý hệ thống, cũng như những bài toán kỹ thuật cần giải quyết trong môi trường thực tế.

Thông qua workshop này, người đọc có thể hiểu được hệ thống được xây dựng như thế nào, các dịch vụ cloud được sử dụng với vai trò gì, kiến trúc hiện tại giải quyết những vấn đề nào và vì sao các quyết định thiết kế đó được lựa chọn.

Bên cạnh đó, workshop cũng ghi lại các trade-offs trong quá trình thiết kế kiến trúc, bao gồm lợi thế, hạn chế, chi phí vận hành, mức độ phức tạp khi triển khai, cũng như các hướng cải tiến trong tương lai.

## 2. Mục tiêu của Workshop

Workshop này hướng đến các mục tiêu chính sau:

- Mô tả tổng quan kiến trúc hệ thống và các thành phần trong dự án.
- Giải thích vai trò của từng dịch vụ được sử dụng trong kiến trúc.
- Trình bày cách hệ thống được triển khai trên môi trường cloud.
- Mô tả luồng hoạt động của hệ thống từ phía người dùng đến backend và database.
- Phân tích các bài toán về CI/CD, monitoring, security, autoscaling, cost optimization và multi-region.
- Ghi lại các quyết định thiết kế quan trọng, trade-offs và lý do lựa chọn kiến trúc.
- Làm tài liệu tham khảo cho quá trình phát triển, vận hành và mở rộng hệ thống trong tương lai.

## 3. Phạm vi của Workshop

Workshop tập trung vào việc mô tả kiến trúc và quy trình vận hành hệ thống trên môi trường AWS. Nội dung không chỉ dừng lại ở việc triển khai ứng dụng, mà còn mở rộng sang các khía cạnh quan trọng trong thực tế như tự động hóa triển khai, giám sát hệ thống, bảo mật, khả năng mở rộng, tối ưu chi phí và khả năng phục hồi khi xảy ra sự cố.

Các nội dung chính bao gồm:

- Tổng quan kiến trúc hệ thống.
- Mô tả các thành phần frontend, backend, database và hạ tầng cloud.
- Quy trình CI/CD với GitHub Actions.
- Monitoring và observability với Amazon CloudWatch và Grafana.
- Real-time notification khi hệ thống phát sinh lỗi.
- Auto Scaling để đáp ứng tải tăng cao.
- Security với AWS WAF và các cơ chế bảo vệ hệ thống.
- Cost Optimization nhằm tối ưu chi phí vận hành.
- Đồng bộ dữ liệu giữa các Region.
- High Availability và Disaster Recovery cho kiến trúc multi-region.
- Backup, restore và các lưu ý vận hành.

## 4. Bài toán dự án cần giải quyết

Trong dự án này, hệ thống không chỉ cần chạy được ứng dụng mà còn cần đáp ứng các yêu cầu quan trọng về vận hành, bảo mật, khả năng mở rộng và độ ổn định. Dưới đây là các bài toán chính được xử lý trong workshop.

### 4.1 CI/CD với GitHub Actions

Dự án cần một quy trình triển khai tự động nhằm giảm thao tác thủ công, hạn chế lỗi trong quá trình deploy và tăng tốc độ phát hành tính năng mới.

GitHub Actions được sử dụng để xây dựng pipeline CI/CD, bao gồm các bước như kiểm tra source code, build ứng dụng, build Docker image, push image lên container registry và triển khai ứng dụng lên hạ tầng AWS.

Mục tiêu của phần này là giúp quá trình release trở nên nhất quán, dễ kiểm soát và có thể mở rộng cho nhiều môi trường như development, staging và production.

### 4.2 Monitoring với CloudWatch và Grafana

Để vận hành hệ thống ổn định, dự án cần có khả năng giám sát các chỉ số quan trọng như CPU, memory, request count, latency, error rate, log ứng dụng và trạng thái của các service.

Amazon CloudWatch được sử dụng để thu thập logs, metrics và tạo cảnh báo. Grafana được sử dụng để trực quan hóa dữ liệu giám sát thông qua dashboard, giúp dễ dàng theo dõi tình trạng hệ thống theo thời gian thực.

Phần này giúp người vận hành nhanh chóng phát hiện bất thường, phân tích nguyên nhân sự cố và đưa ra hành động xử lý kịp thời.

### 4.3 Real-time Notification khi phát sinh lỗi

Khi hệ thống xảy ra lỗi, việc phát hiện và phản hồi nhanh là yếu tố rất quan trọng. Dự án cần xây dựng cơ chế gửi thông báo lỗi theo thời gian thực đến người vận hành.

Các cảnh báo có thể được kích hoạt dựa trên log error, HTTP 5xx, latency cao, service không phản hồi hoặc tài nguyên vượt ngưỡng cho phép. Thông báo có thể được gửi qua email, Slack, Telegram hoặc các kênh communication khác.

Mục tiêu là giảm thời gian phát hiện sự cố, rút ngắn thời gian khắc phục và cải thiện độ tin cậy của hệ thống.

### 4.4 Auto Scaling

Hệ thống cần có khả năng tự động mở rộng hoặc thu hẹp tài nguyên dựa trên lưu lượng truy cập thực tế. Auto Scaling giúp đảm bảo ứng dụng vẫn hoạt động ổn định khi lượng người dùng tăng cao, đồng thời tránh lãng phí tài nguyên khi tải giảm.

Phần này tập trung vào cách thiết kế scaling cho các thành phần như compute, container workload hoặc các service backend. Các chỉ số như CPU utilization, memory usage, request count hoặc custom metrics có thể được sử dụng làm điều kiện scaling.

### 4.5 Security với AWS WAF

Bảo mật là một phần quan trọng trong kiến trúc hệ thống. AWS WAF được sử dụng để bảo vệ ứng dụng trước các loại tấn công phổ biến ở tầng web như SQL Injection, Cross-Site Scripting, malicious bots và các request bất thường.

Bên cạnh WAF, workshop cũng có thể mở rộng sang các khía cạnh bảo mật khác như IAM least privilege, security group, HTTPS, secret management, logging, audit và phân quyền truy cập.

Mục tiêu là thiết kế hệ thống có khả năng phòng thủ tốt hơn, giảm rủi ro bị tấn công và đảm bảo các thành phần quan trọng được bảo vệ đúng cách.

### 4.6 Cost Optimization

Một kiến trúc tốt không chỉ cần ổn định và bảo mật, mà còn cần tối ưu về chi phí. Workshop sẽ phân tích các thành phần có thể phát sinh chi phí cao như compute, NAT Gateway, data transfer, database, monitoring, storage và multi-region deployment.

Từ đó, workshop đề xuất các hướng tối ưu như lựa chọn đúng loại tài nguyên, sử dụng autoscaling, lifecycle policy, tối ưu log retention, giảm data transfer không cần thiết và cân nhắc trade-offs giữa chi phí với độ sẵn sàng của hệ thống.

### 4.7 Đồng bộ dữ liệu giữa các Region

Với kiến trúc multi-region, dữ liệu cần được đồng bộ giữa các Region để hỗ trợ high availability, disaster recovery và giảm rủi ro mất dữ liệu khi một Region gặp sự cố.

Phần này mô tả cách dữ liệu được replication giữa các Region, các vấn đề cần quan tâm như độ trễ đồng bộ, conflict dữ liệu, RTO, RPO và chiến lược failover.

Mục tiêu là giúp hệ thống có khả năng tiếp tục hoạt động hoặc khôi phục nhanh chóng trong các tình huống lỗi nghiêm trọng.

## 5. Giá trị của Workshop

Sau khi hoàn thành workshop, người đọc có thể nắm được:

- Kiến trúc tổng thể của dự án và cách các thành phần kết nối với nhau.
- Lý do lựa chọn các dịch vụ và mô hình triển khai trên AWS.
- Cách xây dựng pipeline CI/CD để tự động hóa quá trình deploy.
- Cách thiết kế monitoring, alerting và real-time notification cho hệ thống.
- Cách áp dụng security, autoscaling và cost optimization vào kiến trúc thực tế.
- Các trade-offs khi triển khai hệ thống theo hướng multi-region.
- Các rủi ro kỹ thuật và hướng cải tiến cho hệ thống trong tương lai.

## 6. Kết luận

Workshop này đóng vai trò như một tài liệu tổng hợp giúp mô tả đầy đủ quá trình thiết kế, triển khai và vận hành hệ thống. Thay vì chỉ tập trung vào việc ứng dụng chạy được, workshop hướng đến việc xây dựng một kiến trúc có khả năng mở rộng, dễ giám sát, bảo mật tốt hơn, tối ưu chi phí và có khả năng phục hồi khi xảy ra sự cố.

Thông qua các nội dung trong workshop, dự án được trình bày theo hướng gần với thực tế vận hành, giúp người đọc hiểu rõ hơn cách áp dụng các nguyên tắc Cloud, DevOps và System Design vào một hệ thống cụ thể.
