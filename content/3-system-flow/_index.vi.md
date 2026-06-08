---
title: "Luồng hoạt động của hệ thống"
date: 2026-06-03
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

# 3. Luồng hoạt động của hệ thống

Phần này giúp bạn hiểu cách request đi qua từng thành phần trong kiến trúc multi-region, từ lúc người dùng truy cập hệ thống cho đến khi dữ liệu được xử lý và phản hồi.

Chúng ta sẽ chia hệ thống thành 4 luồng chính:
- **Luồng request frontend:** Cách người dùng truy cập giao diện web.
- **Luồng request backend:** Cách các API request được định tuyến và xử lý.
- **Luồng truy cập database:** Cách backend đọc và ghi dữ liệu một cách bảo mật.
- **Luồng failover giữa các region:** Điều gì xảy ra khi toàn bộ một region gặp sự cố.

#### Nội dung
1. [Luồng request frontend](3.1-frontend-request-flow/)
2. [Luồng request backend](3.2-backend-request-flow/)
3. [Luồng truy cập database](3.3-database-access-flow/)
4. [Luồng failover giữa các region](3.4-region-failover-flow/)
