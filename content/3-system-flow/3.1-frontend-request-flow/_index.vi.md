---
title: "Luồng request frontend"
date: 2026-06-03
weight: 1
chapter: false
pre: " <b> 3.1 </b> "
---

# 3.1 Luồng request frontend

Luồng này mô tả cách người dùng tải giao diện của ứng dụng web.

### Chi tiết luồng

```txt
User → www.moneytrack.com → AWS Amplify → Frontend UI → API call to api.moneytrack.com
```

1. **User Request:** Người dùng mở trình duyệt và truy cập vào domain `www.moneytrack.com`.
2. **Phân giải DNS:** DNS (quản lý bởi Route 53) trỏ người dùng đến frontend đang được host trên **AWS Amplify**.
3. **Phục vụ Static Assets:** Amplify phục vụ các tài nguyên tĩnh (static assets) của frontend như HTML, CSS, JavaScript từ CDN toàn cầu của nó xuống trình duyệt của người dùng.
4. **Tương tác API:** Sau khi giao diện frontend được tải trên trình duyệt, ứng dụng sẽ gọi API backend thông qua domain `api.moneytrack.com` để lấy hoặc gửi dữ liệu.
5. **Kết quả:** Người dùng nhìn thấy giao diện ứng dụng và có thể thao tác với hệ thống.

### Các khái niệm chính

* **Vì sao frontend tách riêng với backend:** Việc giữ frontend ở dạng static và tách biệt khỏi backend cho phép chúng ta host nó với chi phí thấp và phân phối toàn cầu qua CDN, độc lập với nơi backend xử lý business logic.
* **Vai trò của Amplify:** AWS Amplify cung cấp dịch vụ managed hosting cho các ứng dụng web tĩnh. Nó tự động quản lý CDN, cấu hình custom domain và chứng chỉ SSL/TLS.
* **Khi nào frontend cần gọi backend API:** Frontend chỉ gọi backend API khi cần dữ liệu động, chẳng hạn như xác thực người dùng, lấy danh sách giao dịch hoặc lưu dữ liệu mới.
* **Lưu ý triển khai:** Khi deploy frontend, environment variables được sử dụng để cấu hình đúng API endpoint URL (`api.moneytrack.com`). Việc mapping custom domain trong Amplify đảm bảo người dùng có thể truy cập trang web bằng tên miền thương hiệu.
