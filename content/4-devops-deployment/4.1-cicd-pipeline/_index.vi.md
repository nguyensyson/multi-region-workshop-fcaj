---
title: "CI/CD pipeline cho frontend và backend"
date: 2026-06-03
weight: 1
chapter: false
pre: " <b> 4.1 </b> "
---

# 4.1 CI/CD pipeline cho frontend và backend

### Frontend CI/CD với AWS Amplify

**Luồng triển khai:**
```txt
Developer → GitHub → AWS Amplify → Build Frontend → Deploy → User truy cập www.moneytrack.com
```

* **Quá trình:** Developer push code frontend lên GitHub. AWS Amplify tự động detect thay đổi trên branch được cấu hình. Amplify chạy build command để build static frontend. Sau khi build thành công, Amplify deploy frontend lên hosting environment. User truy cập frontend thông qua domain `www.moneytrack.com`.
* **Quản lý môi trường:** Có thể quản lý nhiều environment như dev, staging, production bằng branch-based deployment (triển khai dựa trên nhánh).

**Các bước chính:**
1. Push code frontend lên GitHub.
2. Amplify trigger pipeline.
3. Install dependencies.
4. Build frontend.
5. Deploy static assets.
6. Kiểm tra frontend sau deploy.

**Lưu ý:**
* Cấu hình environment variables cho API endpoint chính xác cho từng môi trường.
* Tách branch cho từng environment.
* Kiểm tra build log khi deploy lỗi.
* Không lưu secret trực tiếp trong source code.
* Mapping custom domain cho frontend.

---

### Backend CI/CD với ECS Fargate và ECR

**Luồng triển khai:**
```txt
Developer → GitHub → CI/CD Pipeline → Build Docker Image → Push to ECR → Deploy to ECS Fargate → ALB → User
```

* **Quá trình:** Developer push code backend lên GitHub. Pipeline chạy test và build application. Docker image được build từ source code backend. Image được tag theo version, commit SHA hoặc release tag. Image được push lên Amazon ECR. ECS service cập nhật task definition để sử dụng image mới. ECS Fargate triển khai task mới trong private subnet. ALB health check đảm bảo chỉ route traffic đến task healthy.

**Các bước chính:**
1. Checkout source code.
2. Run unit test/build backend.
3. Build Docker image.
4. Login Amazon ECR.
5. Tag image.
6. Push image lên ECR.
7. Update ECS task definition.
8. Deploy ECS service.
9. Kiểm tra health check và log.

**Lưu ý:**
* Không dùng tag `latest` cho production, nên dùng version hoặc commit SHA.
* Cấu hình IAM permission cho pipeline theo least privilege.
* Dùng rolling deployment để giảm downtime.
* Theo dõi CloudWatch Logs sau khi deploy.
* Có thể deploy lần lượt từng region để giảm rủi ro.
* Có thể rollback bằng cách deploy lại image version cũ.

---

### Tóm tắt
* Frontend deploy qua Amplify.
* Backend deploy qua ECR và ECS Fargate.
* CI/CD giúp triển khai nhanh, ổn định và dễ rollback.
