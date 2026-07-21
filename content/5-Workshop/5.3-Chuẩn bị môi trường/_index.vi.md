---
title : "Chuẩn bị môi trường"
date : 2026-07-21
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

## Giới thiệu

Trước khi triển khai hạ tầng, chúng ta cần chuẩn bị đầy đủ công cụ dòng
lệnh và tài khoản AWS với quyền phù hợp. Chương này hướng dẫn cài đặt
**AWS CLI**, **AWS SAM CLI**, **Node.js**, và tạo một **IAM User** riêng
cho việc phát triển thay vì dùng tài khoản `root`.

---

## Yêu cầu

- Tài khoản AWS đang hoạt động.
- Node.js phiên bản **>= 22.x**.
- AWS CLI đã cài đặt.
- AWS SAM CLI đã cài đặt.
- Một trình soạn thảo code (VS Code khuyến nghị).

---

## Tạo IAM User cho việc phát triển

Không sử dụng Access Key của user `root` cho bất kỳ thao tác nào. Tạo
một IAM User riêng, gắn quyền vừa đủ để triển khai project này.

1. Đăng nhập **AWS Console** → vào dịch vụ **IAM**.
2. Chọn **Users** → **Create user**.
3. Đặt tên, ví dụ `smart-notes-dev`.
4. Gắn các policy cần thiết (least-privilege cho việc phát triển, không phải cho Lambda runtime):
   - `AWSCloudFormationFullAccess`
   - Quyền tạo/xóa Lambda, API Gateway, DynamoDB, S3, IAM Role (có thể gắn tạm `PowerUserAccess` cho môi trường học tập, và siết lại khi lên production).
5. Tạo **Access Key** cho user này (chọn use case "Command Line Interface").
6. Lưu lại `Access Key ID` và `Secret Access Key` — đây là lần duy nhất Secret Access Key hiển thị.

![Tạo IAM User](/images/5-Workshop/5.3/create-iam-user.png)

---

## Cài đặt và cấu hình AWS CLI

```bash
aws --version
aws configure
```

Nhập lần lượt:

```
AWS Access Key ID: <access-key-vừa-tạo>
AWS Secret Access Key: <secret-key-vừa-tạo>
Default region name: ap-southeast-1
Default output format: json
```

Kiểm tra cấu hình đã đúng:

```bash
aws sts get-caller-identity
```

Kết quả trả về đúng `Account` và `Arn` của user `smart-notes-dev` vừa tạo
nghĩa là cấu hình thành công.

---

## Cài đặt AWS SAM CLI

```bash
# macOS
brew tap aws/tap
brew install aws-sam-cli

# Windows
# Tải installer .msi từ trang chính thức AWS SAM CLI và chạy theo hướng dẫn
```

Kiểm tra:

```bash
sam --version
```

---

## Kiểm tra Node.js

```bash
node --version   # phải >= 22.0.0
npm --version
```

Nếu chưa có Node.js 22, khuyến nghị cài qua **nvm** để dễ quản lý nhiều phiên bản:

```bash
nvm install 22
nvm use 22
```

---

## Lấy mã nguồn project

```bash
git clone <repo-url> smart-notes-api
cd smart-notes-api
npm install
```

Cấu trúc thư mục sẽ làm việc xuyên suốt workshop:

```
smart-notes-api/
├── template.yaml            # AWS SAM - định nghĩa toàn bộ hạ tầng
├── package.json
├── src/
│   ├── app.js                # Express app (routes, middleware)
│   ├── lambda.js             # Entry point cho Lambda (serverless-http)
│   ├── config/                # Cấu hình DynamoDB, S3 client
│   ├── routes/                # Định nghĩa route Express
│   ├── controllers/           # Nhận request, gọi service, trả response
│   ├── services/               # Business logic (validate, retry, presign...)
│   ├── repositories/           # Duy nhất tầng này gọi DynamoDB
│   ├── utils/                  # Response format, error class, validation
│   └── public/                 # Giao diện web tĩnh (index.html)
└── tests/                      # Unit test (Jest)
```

---

## Xác nhận môi trường sẵn sàng

```bash
sam validate --lint
```

Nếu không có lỗi nào được báo, môi trường đã sẵn sàng để chuyển sang
chương tiếp theo: triển khai hạ tầng bằng AWS SAM.