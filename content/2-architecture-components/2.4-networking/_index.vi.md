---
title: "Networking - VPC, Subnet, IGW, NAT Gateway, VPC Endpoint"
date: 2026-06-03
weight: 4
chapter: false
pre: " <b> 2.4 </b> "
---

# 2.4 Networking - VPC, Subnet, IGW, NAT Gateway, VPC Endpoint

### Vai trò của VPC trong kiến trúc
Amazon Virtual Private Cloud (VPC) cung cấp một mạng ảo bị cô lập cho các tài nguyên AWS của bạn. Trong kiến trúc này, mỗi region có VPC riêng đóng vai trò là nền tảng bảo mật. Nó kiểm soát ranh giới mạng, luồng traffic và địa chỉ IP cho tất cả các thành phần được triển khai trong region đó.

### Public subnet và Private subnet
Subnet chia nhỏ dải địa chỉ IP của VPC.
* **Public Subnet:** Chứa các tài nguyên cần được truy cập từ internet, chẳng hạn như Application Load Balancer (ALB) và NAT Gateways. Các subnet này có route đến Internet Gateway.
* **Private Subnet:** Chứa logic ứng dụng cốt lõi của chúng ta (ECS Fargate tasks) và cơ sở dữ liệu. Chúng không có route trực tiếp ra internet, cung cấp một ranh giới bảo mật vững chắc chống lại truy cập từ bên ngoài.

### Internet Gateway (IGW)
Internet Gateway là một thành phần VPC được mở rộng theo chiều ngang (horizontally scaled), dự phòng và có tính sẵn sàng cao, cho phép giao tiếp giữa các instance trong VPC của bạn và internet. Trong kiến trúc của chúng ta, nó cho phép traffic từ internet đến được ALB trong các public subnet.

### NAT Gateway
NAT (Network Address Translation) Gateway cho phép các tài nguyên trong private subnet (như các ECS task của chúng ta) kết nối với internet (ví dụ: để tải xuống Docker image từ ECR hoặc các bản vá lỗi) đồng thời ngăn không cho internet khởi tạo kết nối đến các instance đó. Nó nằm trong public subnet và định tuyến traffic từ private subnet đến IGW.

### VPC Endpoint
VPC Endpoint cho phép kết nối private giữa VPC của bạn và các dịch vụ AWS được hỗ trợ mà không cần Internet Gateway, thiết bị NAT, kết nối VPN hoặc kết nối AWS Direct Connect.
* Chúng giúp truy cập AWS services private hơn và tối ưu network path, giữ traffic trên mạng toàn cầu của AWS.
* Trong kiến trúc này, chúng ta sử dụng chúng cho các dịch vụ như DynamoDB (Gateway Endpoint), ECR, Secrets Manager và CloudWatch (Interface Endpoints) để đảm bảo các backend service có thể giao tiếp an toàn với các AWS API mà không cần đi qua internet.

### Lý do cần thiết kế network theo nhiều AZ
Triển khai tài nguyên trên nhiều Availability Zone (AZ) trong một region cung cấp khả năng chịu lỗi và tính sẵn sàng cao (High Availability). Nếu một AZ gặp sự cố (ví dụ: do sự cố mất điện hoặc mạng), ALB có thể định tuyến traffic đến các ECS task khỏe mạnh đang chạy ở các AZ khác, đảm bảo dịch vụ không bị gián đoạn.

### Lưu ý triển khai
* **CIDR Planning:** Đảm bảo các khối CIDR của VPC và subnet đủ lớn để đáp ứng sự phát triển trong tương lai và không trùng lặp với các mạng khác mà bạn có thể kết nối tới (ví dụ: qua peering hoặc VPN).
* **Route Tables:** Quản lý cẩn thận các route table để đảm bảo public subnet định tuyến traffic internet đến IGW và private subnet định tuyến traffic outbound đến NAT Gateway hoặc VPC Endpoints.
* **Security Groups và NACLs:** Sử dụng Security Group như các firewall stateful ở cấp độ instance/task và Network ACL (NACL) như các firewall stateless ở cấp độ subnet để thực thi các ranh giới bảo mật nghiêm ngặt.
* **Chi phí NAT Gateway:** Cần lưu ý rằng NAT Gateways phát sinh phí hàng giờ và phí xử lý dữ liệu. Hãy cân nhắc sử dụng VPC Endpoints ở những nơi có thể để giảm chi phí xử lý dữ liệu.
* **Endpoint Policy:** Đính kèm policy vào VPC Endpoints để giới hạn quyền truy cập chỉ cho phép các API action cụ thể hoặc các tài nguyên AWS cụ thể (ví dụ: chỉ cho phép truy cập vào một bảng DynamoDB cụ thể).
