---
title : "Dọn dẹp tài nguyên"
date : 2026-07-21
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

## Giới thiệu

Đây là bước **bắt buộc** sau khi hoàn thành workshop, để tránh phát sinh
chi phí ngoài ý muốn — đặc biệt là chi phí lưu trữ Amazon S3, vốn tính
theo dung lượng mỗi ngày chứ không phụ thuộc vào việc bạn có gọi API hay
không.

---

## Bước 1 — Xóa toàn bộ object trong bucket ảnh

AWS CloudFormation **từ chối xóa bucket S3 nếu bên trong còn object**.
Đây phải là bước làm trước tiên.

```bash
aws s3 rm s3://smart-notes-storage-<account-id>-dev --recursive
```

Nếu bucket có bật **Versioning**, cần kiểm tra và xóa cả các version cũ
qua tab "Versions" trong S3 Console, vì lệnh `rm --recursive` mặc định
không xóa version đã xóa (delete marker).

![Xóa object trong S3 bucket](/images/5-Workshop/5.8/s3-empty-bucket.png)

---

## Bước 2 — Xóa toàn bộ CloudFormation Stack

```bash
sam delete
```

Xác nhận `y` cho cả hai câu hỏi (xóa stack và xóa artifact bucket của SAM):

```
Are you sure you want to delete the stack smart-notes-api in the region ap-southeast-1 ? [y/N]: y
Are you sure you want to delete the folder smart-notes-api in S3 which contains the artifacts? [y/N]: y
```

Lệnh này tự động xóa: AWS Lambda, Amazon API Gateway, Amazon DynamoDB
table, IAM Role — toàn bộ resource được khai báo trong `template.yaml`.

Nếu dùng Console thay vì CLI: **CloudFormation → chọn stack
`smart-notes-api` → Delete**, theo dõi đến khi trạng thái chuyển thành
`DELETE_COMPLETE`.

![CloudFormation Stack Delete](/images/5-Workshop/5.8/cloudformation-delete.png)

---

## Bước 3 — Xóa bucket S3 dùng để deploy (SAM managed bucket)

Khi chạy `sam deploy --guided` lần đầu, SAM tự tạo **một bucket S3
khác** để chứa gói code — bucket này không nằm trong stack và không tự
xóa cùng Bước 2.

```bash
aws s3 rb s3://aws-sam-cli-managed-default-samclisourcebucket-xxxxxxxxxxxxx --force
```

---

## Bước 4 — Xóa CloudWatch Log Group

CloudFormation không tự xóa log group đã ghi trong lúc chạy:

```bash
aws logs delete-log-group --log-group-name /aws/lambda/smart-notes-api-dev --region ap-southeast-1
```

---

## Bước 5 — Checklist xác nhận đã dọn sạch

Kiểm tra tay trên Console, đúng region **ap-southeast-1**, để chắc chắn
không có gì bị bỏ sót:

| Dịch vụ | Kiểm tra |
|---|---|
| CloudFormation | Không còn stack `smart-notes-api` |
| AWS Lambda | Không còn function `smart-notes-api-dev` |
| Amazon API Gateway | Không còn API tương ứng |
| Amazon DynamoDB | Không còn bảng `Notes-dev` |
| Amazon S3 | Không còn cả 2 bucket (bucket ảnh và bucket SAM managed) |
| CloudWatch Logs | Không còn log group của function |
| IAM | Role thực thi Lambda tự bị xóa cùng stack — không cần thao tác tay |

![Checklist xác nhận đã dọn sạch](/images/5-Workshop/5.8/cleanup-checklist.png)

---

## Xử lý khi xóa thất bại

Nếu `sam delete` hoặc xóa stack trên Console báo `DELETE_FAILED`,
nguyên nhân gần như luôn là **Bước 1 chưa hoàn tất** (bucket còn object,
kể cả version cũ nếu bucket có bật Versioning). Quay lại Bước 1, xóa
sạch, sau đó thử xóa stack lại lần nữa — CloudFormation sẽ tiếp tục xóa
phần resource còn lại.

AWS X-Ray trace data tự động hết hạn sau 30 ngày và không tính phí lưu
trữ riêng, nên không cần dọn tay.

---

## Kết quả

Sau chương này, toàn bộ tài nguyên AWS của Smart Notes API đã được dọn
sạch, không còn phát sinh chi phí. Bạn đã hoàn thành toàn bộ workshop:
từ thiết kế kiến trúc, triển khai hạ tầng và frontend, kiểm thử/đo
lường, tối ưu chi phí/bảo mật, đến dọn dẹp đúng cách — một quy trình
đầy đủ vòng đời của một hệ thống Serverless trên AWS.