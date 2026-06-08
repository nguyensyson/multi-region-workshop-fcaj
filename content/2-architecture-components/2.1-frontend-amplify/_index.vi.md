---
title: "Frontend - AWS Amplify"
date: 2026-06-03
weight: 1
chapter: false
pre: " <b> 2.1 </b> "
---

# 2.1 Frontend - AWS Amplify

### AWS Amplify là gì?
AWS Amplify là một giải pháp toàn diện giúp các lập trình viên frontend web và mobile dễ dàng xây dựng, phát hành và lưu trữ các ứng dụng full-stack trên AWS. Trong kiến trúc này, chúng ta sử dụng **Amplify Hosting**, một dịch vụ quản lý CI/CD và hosting hoàn toàn managed để phân phối các ứng dụng static và server-side rendered một cách nhanh chóng, bảo mật và đáng tin cậy.

### Vai trò của Amplify trong hệ thống Frontend
Amplify đóng vai trò là cơ chế phân phối cho ứng dụng frontend React/Next.js của chúng ta. Nó tự động build ứng dụng từ repository mã nguồn và lưu trữ các tài nguyên tĩnh (HTML, CSS, JavaScript, hình ảnh) trên một Mạng phân phối nội dung (CDN) toàn cầu.

### Cách user truy cập frontend
Khi người dùng truy cập vào domain của ứng dụng (ví dụ: `www.moneytrack.com`), request DNS sẽ được phân giải đến edge location của CloudFront gần nhất được quản lý bởi Amplify, đảm bảo thời gian tải trang nhanh chóng bất kể vị trí địa lý của người dùng.

### Lý do chọn AWS Amplify
* **Managed Hosting:** Không cần cấp phát, vá lỗi hay bảo trì máy chủ.
* **Tích hợp CI/CD:** Tự động kích hoạt quá trình build và deploy khi có code mới đẩy lên các branch được kết nối.
* **CDN Toàn cầu:** Xây dựng trên nền tảng Amazon CloudFront giúp phân phối nội dung với độ trễ thấp trên toàn thế giới.
* **Hỗ trợ Custom Domain:** Dễ dàng cấu hình domain tùy chỉnh và tự động cung cấp chứng chỉ SSL/TLS miễn phí.

### Vai trò trong kiến trúc Multi-Region
Mặc dù các dịch vụ backend được triển khai rõ ràng ở nhiều region để đạt High Availability, Amplify Hosting mang lại lợi ích multi-region vốn có thông qua CDN toàn cầu của nó. Nó phục vụ các tài nguyên frontend từ các edge location gần người dùng nhất, cung cấp một điểm truy cập ứng dụng có tính đàn hồi và khả dụng trên toàn cầu.

### Lưu ý triển khai
* **Environment Variables:** Quản lý an toàn các giá trị cấu hình (như API endpoints) cho các môi trường khác nhau (dev, staging, prod).
* **Branch-based Deployment:** Ánh xạ các branch Git vào các môi trường Amplify cụ thể để dễ dàng kiểm thử tính năng mới trước khi phát hành lên production.
* **Build Settings:** Cấu hình file `amplify.yml` để chỉ định các lệnh build và thư mục đầu ra cho framework cụ thể của bạn.
* **Domain Mapping:** Đảm bảo các bản ghi DNS trong Route 53 trỏ đúng đến ứng dụng Amplify.
