---
title : "Kiểm thử và đo lường"
date : 2026-07-21
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---
## Giới thiệu

Một hệ thống Serverless không có máy chủ để SSH vào xem log trực tiếp,
nên việc kiểm thử và quan sát (observability) phải được thiết kế ngay từ
đầu. Chương này hướng dẫn kiểm thử ở 3 mức: unit test trước khi deploy,
gọi API thật sau khi deploy, và giám sát bằng CloudWatch/X-Ray trong lúc
vận hành.

---

## Unit test (trước khi deploy)

Project dùng **Jest** kết hợp **aws-sdk-client-mock** để test cả 3 tầng
(Controller/Service/Repository) mà **không gọi AWS thật** — nhanh, miễn
phí, chạy được trong CI/CD.

```bash
npm test
```

Ví dụ một business rule được test: service phải retry tối đa 3 lần khi
upload S3 gặp lỗi tạm thời, và phải xóa/giữ ảnh đúng theo trạng thái
soft-delete — cả hai đều được assert bằng mock.

![Kết quả chạy unit test](/images/5-Workshop/5.6/jest-test-result.png)

---

## Kiểm thử API thật bằng curl / Postman

```bash
BASE=https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev

# Tạo note
curl -X POST $BASE/notes -H "Content-Type: application/json" \
  -d '{"title":"Test","content":"Hello workshop"}'

# Lấy danh sách note đang hoạt động
curl $BASE/notes

# Upload ảnh cho note
curl -X POST $BASE/notes/<id>/image -F "image=@photo.png"

# Xóa note (chuyển vào thùng rác)
curl -X DELETE $BASE/notes/<id>

# Xem thùng rác
curl $BASE/notes/trash

# Khôi phục note
curl -X POST $BASE/notes/<id>/restore
```

Mọi response đều theo format thống nhất: `{"success": true/false,
"data"/"message": ...}`. Bộ Postman Collection có sẵn trong
`postman/Smart Notes API.postman_collection.json` để kiểm thử nhanh
toàn bộ endpoint mà không cần gõ tay từng lệnh `curl`.

---

## Xem log bằng Amazon CloudWatch Logs

Mỗi lần Lambda invoke đều tự động ghi log — quyền ghi log được SAM cấp
mặc định kèm execution role, không cần cấu hình thêm.

**Qua Console:** CloudWatch → Log groups → `/aws/lambda/smart-notes-api-dev` → chọn log stream mới nhất.

**Qua CLI (tail log theo thời gian thực khi đang test):**

```bash
sam logs -n SmartNotesFunction --stack-name smart-notes-api --tail
```

**Lọc log lỗi bằng CloudWatch Logs Insights:**

```
fields @timestamp, @message
| filter @message like /ERROR|error/
| sort @timestamp desc
| limit 20
```

![CloudWatch Logs Insights](/images/5-Workshop/5.6/cloudwatch-logs-insights.png)

---

## Trace request bằng AWS X-Ray

Vì đã bật `Tracing: Active` trong `template.yaml` ở chương 5.4, mỗi
request được trace qua từng bước: API Gateway → Lambda → DynamoDB/S3,
kèm thời gian xử lý từng đoạn.

Vào **X-Ray console → Service Map** để xem trực quan toàn bộ kiến trúc,
node nào đang lỗi sẽ được tô đỏ. Vào **Traces** để xem chi tiết từng
request, phát hiện request nào chậm nhất (ví dụ do retry upload S3).

![AWS X-Ray Service Map](/images/5-Workshop/5.6/xray-service-map.png)

---

## Thiết lập Alarm cơ bản

Đo lường chủ động thay vì bị động chờ người dùng báo lỗi:

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name smart-notes-api-error-rate \
  --namespace AWS/Lambda \
  --metric-name Errors \
  --dimensions Name=FunctionName,Value=smart-notes-api-dev \
  --statistic Sum \
  --period 300 \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --region ap-southeast-1
```

Alarm này kích hoạt nếu có hơn 5 lỗi Lambda trong 5 phút. Có thể gắn
thêm Amazon SNS Topic để nhận email/SMS khi alarm chuyển trạng thái
`ALARM` — bước mở rộng tự nhiên tiếp theo ngoài phạm vi workshop này.

---

## Các chỉ số (metrics) nên theo dõi định kỳ

| Metric | Ý nghĩa |
|---|---|
| `Lambda Duration` | Phát hiện cold start hoặc truy vấn DynamoDB chậm |
| `Lambda Errors` / `Throttles` | Phát hiện lỗi code hoặc vượt concurrency limit |
| `DynamoDB ConsumedRead/WriteCapacityUnits` | Kiểm tra chi phí thực tế đang phát sinh |
| `API Gateway 4XX/5XX Count` | Phân biệt lỗi do client hay do backend |

---

## Kết quả

Sau chương này, bạn đã có đầy đủ công cụ để biết hệ thống đang chạy đúng
hay không: từ unit test cục bộ, kiểm thử API thật, tới log và trace phân
tán trên AWS. Chương tiếp theo sẽ tối ưu chi phí và bảo mật cho hệ thống.