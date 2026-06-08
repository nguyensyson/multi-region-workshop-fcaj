---
title: "Giới thiệu bài toán"
date: 2026-06-03
weight: 1
chapter: false
pre: " <b> 1.1 </b> "
---

# 1.1 Giới thiệu bài toán

Hầu hết các ứng dụng web hiện đại đều có yêu cầu ngày càng cao về **tính sẵn sàng**, **tốc độ phản hồi** và **khả năng chịu tải**. Người dùng đến từ nhiều khu vực địa lý khác nhau — Đông Nam Á, châu Âu, Bắc Mỹ — và đều kỳ vọng ứng dụng hoạt động ổn định, phản hồi nhanh bất kể họ đang ở đâu.

Trong bài toán này, chúng ta xây dựng hệ thống backend cho ứng dụng **MoneyTrack** — một ứng dụng quản lý tài chính cá nhân phục vụ người dùng toàn cầu. Hệ thống cần đảm bảo:

- Người dùng truy cập API từ bất kỳ quốc gia nào đều nhận được phản hồi nhanh và ổn định.
- Dữ liệu không bị mất khi xảy ra sự cố tại một khu vực.
- Hệ thống tự động phục hồi mà không cần can thiệp thủ công.

### Vấn đề của kiến trúc Single-Region

Nếu toàn bộ hệ thống chỉ được triển khai trên một AWS Region duy nhất, sẽ phát sinh các rủi ro sau:

| Rủi ro | Mô tả |
|--------|-------|
| **Downtime** | Khi Region gặp sự cố (thiên tai, lỗi hạ tầng), toàn bộ dịch vụ ngừng hoạt động. |
| **Độ trễ cao** | Người dùng ở xa Region phải chịu độ trễ mạng lớn, ảnh hưởng đến trải nghiệm. |
| **Điểm lỗi duy nhất** | Mọi thành phần tập trung ở một nơi tạo ra *Single Point of Failure* (SPOF). |
| **Mất khả năng phục vụ** | Không có cơ chế tự động chuyển lưu lượng sang vùng dự phòng khi xảy ra sự cố. |

Để giải quyết các vấn đề này, workshop này hướng dẫn bạn xây dựng một kiến trúc **Multi-Region** trên AWS — nơi hệ thống được triển khai song song ở nhiều Region, đảm bảo tính liên tục và hiệu năng cao.
