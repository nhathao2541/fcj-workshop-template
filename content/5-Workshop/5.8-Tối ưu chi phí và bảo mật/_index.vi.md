---
title : "Tối ưu chi phí và bảo mật"
date : 2026-07-21
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

## Giới thiệu

Chương này tổng hợp các quyết định thiết kế của Smart Notes API dưới góc
nhìn hai trụ cột trong AWS Well-Architected Framework: **Cost
Optimization** và **Security**. Đây không phải các bước làm thêm cuối
cùng, mà là những lựa chọn đã được cân nhắc xuyên suốt từ chương 5.2.

---

## Tối ưu chi phí

| Quyết định thiết kế | Tác động chi phí |
|---|---|
| AWS Lambda thay vì EC2/ECS | Không tính phí khi idle — chỉ trả tiền theo số lần invoke và thời gian chạy (ms) |
| Amazon DynamoDB `PAY_PER_REQUEST` | Không trả phí cho capacity đặt trước mà không dùng hết |
| DynamoDB TTL cho thùng rác | Note hết hạn tự xóa, không tích lũy dữ liệu rác vô thời hạn |
| Amazon S3 thay vì lưu ảnh trong DynamoDB | S3 rẻ hơn nhiều lần cho dữ liệu nhị phân so với lưu base64 trong DynamoDB item |
| Serve frontend qua Lambda (không CloudFront) | Tránh phí CloudFront không cần thiết cho traffic thấp, có thể nâng cấp sau nếu cần |

**Ước tính ở mức dùng thử/demo:** với vài trăm request/ngày, toàn bộ hệ
thống nằm trong **AWS Free Tier** (1 triệu lượt Lambda invoke/tháng,
25GB DynamoDB storage, 5GB S3 — miễn phí 12 tháng đầu cho tài khoản mới).

![Chi phí ước tính trên AWS Cost Explorer](/images/5-Workshop/5.7/cost-explorer.png)

### Thiết lập cảnh báo chi phí chủ động

```bash
aws budgets create-budget --account-id <account-id> --budget file://budget.json
```

Cấu hình AWS Budgets để nhận email ngay khi chi phí thực tế hoặc dự
đoán vượt ngưỡng đặt trước — tránh phát hiện chi phí phát sinh quá muộn.

---

## Bảo mật cơ bản

### IAM least-privilege

Lambda execution role **không** dùng `AdministratorAccess`. Thay vào đó,
`DynamoDBCrudPolicy`/`S3CrudPolicy` của SAM sinh ra policy chỉ cho phép
thao tác đúng bảng/bucket được `!Ref` trong template — Lambda không thể
động vào tài nguyên khác trong tài khoản dù code có bị khai thác lỗ hổng.

### Amazon S3 Block Public Access + Presigned URL

Bucket ảnh chặn public access ở cả 4 cờ. Ảnh không bao giờ có URL public
tĩnh — mọi lượt xem đều qua **presigned URL** (chữ ký tạm thời, hết hạn
sau 15 phút), được Lambda sinh ra bằng chính quyền IAM của nó. Kẻ tấn
công không thể liệt kê hay truy cập trực tiếp object trong bucket, kể cả
khi biết chính xác tên bucket.

![Bucket S3 chặn Public Access](/images/5-Workshop/5.7/s3-block-public-access.png)

### Validate input ở tầng Service

`title` tối đa 100 ký tự, `content` tối đa 5000 ký tự, đều bắt buộc —
chặn payload bất thường trước khi chạm tới DynamoDB, giảm rủi ro tấn
công kiểu injection hoặc payload quá khổ gây tốn chi phí ghi.

### Bảo mật API Gateway (mở rộng, khuyến nghị cho production)

API hiện tại **public hoàn toàn** — phù hợp demo/workshop, nhưng cho môi
trường thật nên bổ sung một trong hai:

- **API Key** (`UsagePlan` + `ApiKeyRequired: true`) — đơn giản, chặn request không có key hợp lệ.
- **Amazon Cognito User Pool Authorizer** — xác thực người dùng thật, phù hợp nếu có khái niệm "chủ sở hữu note".

### Không log dữ liệu nhạy cảm

Log hiện tại (`console.log`) chỉ ghi `title`, `id`, tên file — không ghi
toàn bộ `content` của note vào CloudWatch, tránh rò rỉ dữ liệu người
dùng qua log nếu log bị truy cập trái phép.

---

## Kết quả

Sau chương này, hệ thống đã đạt được sự cân bằng giữa chi phí thấp và
bảo mật cơ bản phù hợp cho một project cá nhân/MVP, đồng thời có lộ
trình rõ ràng để nâng cấp bảo mật khi đưa vào production thật. Chương
cuối cùng sẽ hướng dẫn dọn dẹp toàn bộ tài nguyên sau khi hoàn thành
workshop.