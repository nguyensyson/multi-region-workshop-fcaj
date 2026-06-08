---
title: "High Availability"
date: 2026-06-03
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

# 6. High Availability

High Availability (HA - Tính sẵn sàng cao) trong phạm vi dự án MoneyTrack có nghĩa là đảm bảo ứng dụng luôn có thể truy cập và hoạt động bình thường ngay cả khi các thành phần riêng lẻ, toàn bộ một Availability Zone (AZ), hoặc thậm chí là cả một AWS Region gặp sự cố.

### HA ở tầng Frontend

*   Frontend được host trên **AWS Amplify**, một dịch vụ managed hosting cho web tĩnh được hỗ trợ bởi mạng CDN toàn cầu.
*   Việc tách riêng frontend khỏi backend giúp các tài nguyên tĩnh trở nên cực kỳ bền bỉ và có thể phục vụ người dùng độc lập với trạng thái tính toán của backend.

### HA ở tầng Backend

*   Backend API chạy trên **ECS Fargate** trong các private subnet. Fargate tự quản lý máy chủ vật lý bên dưới, giúp chúng ta không cần lo lắng về các lỗi ở cấp độ host.
*   **ECS Service** được cấu hình để duy trì một số lượng task mong muốn. Nếu một task bị crash, ECS sẽ tự động thay thế bằng một task mới.
*   **Application Load Balancer (ALB)** liên tục thực hiện health check trên các Fargate task. ALB đảm bảo chỉ route request đến những task đang khỏe mạnh (healthy), ngăn người dùng gặp lỗi từ các container hỏng.
*   Trong cùng một region, backend được triển khai rải rác trên nhiều Availability Zone, bảo vệ hệ thống khỏi lỗi sập một AZ cục bộ.

### HA ở tầng Database

*   **Amazon DynamoDB** bản thân nó đã là một managed database có tính HA rất cao trong phạm vi một region (dữ liệu được tự động đồng bộ qua nhiều AZ).
*   Để đạt được HA ở cấp độ multi-region, chúng ta sử dụng **DynamoDB Global Tables**. Tính năng này liên tục replicate (sao chép) dữ liệu bất đồng bộ giữa 2 region của chúng ta.
*   Khi một region gặp lỗi và sập hoàn toàn, region còn lại vẫn có đầy đủ dữ liệu cập nhật để tiếp tục phục vụ hệ thống.

### HA trong Networking

*   **VPC** được thiết kế với public subnet chứa ALB và NAT Gateway, cùng private subnet chứa các ECS Fargate task.
*   Cấu trúc này được nhân rộng ra nhiều AZ. Các NAT Gateway dự phòng (mỗi AZ một cái) đảm bảo các task trong private subnet luôn có thể kết nối ra internet.
*   **VPC Endpoint** cung cấp kết nối ổn định, an toàn và HA trực tiếp đến các dịch vụ AWS như DynamoDB và ECR, không phụ thuộc vào internet bên ngoài.

### Sức bật Multi-AZ và Multi-Region

*   **Multi-AZ:** Việc triển khai hạ tầng trên nhiều AZ trong mỗi region giúp giảm thiểu rủi ro khi một trung tâm dữ liệu của AWS mất điện, lỗi mạng hoặc lỗi hệ thống làm mát.
*   **Multi-Region:** Ở cấp độ toàn cầu, **Route 53** và **AWS Global Accelerator** hỗ trợ định tuyến traffic thông minh. Nếu một region bị đánh dấu unhealthy, Global Accelerator lập tức chuyển traffic API sang region còn healthy. Backend và database ở region dự phòng luôn sẵn sàng xử lý tải.

### Luồng Request High Availability

```txt
User → Route 53 / Global Accelerator → Healthy Region → ALB → ECS Fargate → DynamoDB Global Tables
```

### Kết luận

Khả năng High Availability của dự án MoneyTrack được xây dựng bằng cách kết hợp nhiều lớp bảo vệ: sử dụng các managed service (Amplify, Fargate, DynamoDB), thiết kế mạng multi-AZ, replicate cơ sở dữ liệu multi-region và định tuyến traffic toàn cầu có tích hợp health check.
