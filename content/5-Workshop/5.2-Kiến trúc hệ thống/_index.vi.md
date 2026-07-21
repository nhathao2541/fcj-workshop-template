---
title : "Kiến trúc hệ thống"
date : 2026-07-21
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---
## Giới thiệu

Trong chương này, chúng ta sẽ tìm hiểu kiến trúc tổng thể của dự án **Smart Notes API – Serverless Notes Management Platform**.

Hệ thống được xây dựng theo mô hình **Cloud Native**, áp dụng **Serverless
Architecture** kết hợp **Clean Architecture** (Controller → Service →
Repository) trên nền tảng Amazon Web Services (AWS). Kiến trúc này giúp
hệ thống tự động scale theo traffic, không tốn chi phí khi không có
request (idle), và tách biệt rõ ràng giữa logic nghiệp vụ với chi tiết
hạ tầng AWS.

Toàn bộ quy trình từ khi client gửi request tạo/sửa/xóa ghi chú, tới khi
dữ liệu được lưu trên DynamoDB và ảnh được lưu an toàn trên S3, đều được
xử lý hoàn toàn thông qua các dịch vụ được quản lý (Managed Services) của
AWS, không có bất kỳ máy chủ nào cần vận hành hay vá lỗi thủ công.

---

# Kiến trúc tổng thể

Kiến trúc của hệ thống được chia thành các tầng chính sau:

- Frontend Layer
- API Layer
- Data Layer
- Lifecycle Layer (thùng rác & tự động dọn dữ liệu)
- Monitoring Layer
- Security Layer

Mỗi tầng đảm nhận một nhiệm vụ riêng biệt nhằm đảm bảo hệ thống dễ mở
rộng, dễ bảo trì và đáp ứng kiến trúc Well-Architected Framework của AWS
(đặc biệt là hai trụ cột **Cost Optimization** và **Security**).

---

## Kiến trúc triển khai

Hình dưới đây mô tả toàn bộ kiến trúc của hệ thống Smart Notes API trên AWS.

![System Architecture](/images/5-Workshop/5.1/smart-notes-architecture.png)

### Mô tả kiến trúc

#### Frontend Layer

Giao diện người dùng là một trang **HTML/CSS/JavaScript thuần**, được
phục vụ trực tiếp bởi chính **AWS Lambda** (Express `static` middleware)
thay vì dựng riêng Amazon S3 Static Website + CloudFront.

Lựa chọn này giúp:

- Tối giản số lượng resource cần quản lý cho một API quy mô nhỏ.
- Giao diện và API nằm cùng domain, không phát sinh vấn đề CORS phức tạp.
- Vẫn có thể nâng cấp lên S3 + CloudFront sau này nếu traffic tăng, mà không cần đổi API phía sau.

---

#### API Layer

Các yêu cầu từ client được gửi đến **Amazon API Gateway** (REST API).

API Gateway sẽ:

- Định tuyến toàn bộ request (`/` và `/{proxy+}`) tới một **AWS Lambda** duy nhất.
- Xử lý CORS và hỗ trợ `BinaryMediaTypes` cho việc upload ảnh (`multipart/form-data`).
- Ghi log truy cập.

Bên trong Lambda, request được xử lý bởi **Express + serverless-http**,
tổ chức theo Clean Architecture 3 tầng:

- **Controller** – nhận request, gọi Service, trả response theo format thống nhất.
- **Service** – chứa toàn bộ business logic: validate input, retry S3, sinh presigned URL, tính hạn thùng rác.
- **Repository** – tầng duy nhất được phép gọi trực tiếp Amazon DynamoDB.

---

#### Data Layer

Hệ thống sử dụng **Amazon DynamoDB** để lưu trữ dữ liệu ghi chú.

Mỗi note gồm các thuộc tính:

- `id`, `title`, `content`, `createdAt`, `updatedAt`
- `imageUrl` (nếu có đính kèm ảnh)
- `deletedAt`, `expiresAt` (khi note đang ở trong thùng rác)

Trong khi đó **Amazon S3** lưu trữ:

- Ảnh gốc đính kèm từng note, trong một bucket **private hoàn toàn** (chặn public access ở cả 4 cờ).

---

#### Lifecycle Layer (Thùng rác & tự động dọn dữ liệu)

Khi người dùng xóa một note, hệ thống **không xóa cứng ngay** mà chuyển
note sang trạng thái **soft-delete**: gắn thêm `deletedAt` (hiển thị) và
`expiresAt` (epoch giây, dùng cho DynamoDB TTL).

Người dùng có thể khôi phục note trong vòng **3 ngày**. Sau thời hạn đó,
**DynamoDB Time To Live (TTL)** tự động xóa vĩnh viễn item — hoàn toàn
không cần Lambda lịch (cron) hay Step Functions điều phối, giảm cả độ
phức tạp lẫn chi phí vận hành.

---

#### Monitoring Layer

Toàn bộ hệ thống được giám sát bằng **Amazon CloudWatch** và **AWS X-Ray**.

CloudWatch thu thập:

- Logs của mọi lần Lambda invoke.
- Metrics: Duration, Errors, Throttles, DynamoDB Consumed Capacity.
- Alarm khi tỉ lệ lỗi vượt ngưỡng.

X-Ray (bật qua `Tracing: Active`) giúp trace một request đi qua từng
tầng: API Gateway → Lambda → DynamoDB/S3, xác định chính xác điểm chậm
hoặc điểm lỗi.

---

#### Security Layer

**AWS IAM** cấp quyền theo nguyên tắc **least-privilege** cho Lambda: chỉ
được `Get/Put/Delete` đúng bảng DynamoDB và bucket S3 của project (thông
qua `DynamoDBCrudPolicy`/`S3CrudPolicy` của SAM), không có quyền quản trị
toàn cục.

Ảnh trong S3 không bao giờ có URL public tĩnh — mọi lượt xem đều đi qua
**presigned URL** có hạn 15 phút, được Lambda ký bằng chính quyền IAM của
nó.

---

# Vòng đời của một Note (Note Lifecycle Workflow)

Ngoài kiến trúc tổng thể, hệ thống còn có một quy trình tự động đáng chú
ý: vòng đời của note từ lúc tạo, chỉnh sửa, đính kèm ảnh, cho tới khi bị
xóa (vào thùng rác) và cuối cùng được khôi phục hoặc tự động biến mất.

Quy trình này được điều phối hoàn toàn bởi Amazon DynamoDB (item
attributes + TTL) kết hợp AWS Lambda, không cần thêm dịch vụ điều phối
sự kiện riêng.

![Note Lifecycle Workflow](/images/5-Workshop/5.2/note-lifecycle-workflow.png)

---

## Luồng xử lý

Quy trình xử lý của hệ thống gồm các bước sau:

1. Client gửi `POST /notes` tạo note mới.
2. AWS Lambda validate input (title ≤ 100 ký tự, content ≤ 5000 ký tự) và ghi vào Amazon DynamoDB.
3. Client gửi `POST /notes/{id}/image` để đính kèm ảnh (multipart/form-data).
4. AWS Lambda nhận file qua buffer (không ghi disk), upload lên Amazon S3, có retry tối đa 3 lần nếu lỗi tạm thời.
5. Amazon DynamoDB được cập nhật với key ảnh vừa upload.
6. Khi client gọi `GET /notes`, AWS Lambda đọc DynamoDB và **ký lại (presign)** từng ảnh thành URL tạm thời trước khi trả về.
7. Khi client gọi `DELETE /notes/{id}`, note được **soft-delete**: gắn `deletedAt` và `expiresAt` (= hiện tại + 3 ngày).
8. Note biến mất khỏi `GET /notes` nhưng vẫn xem được qua `GET /notes/trash`.
9. Nếu người dùng gọi `POST /notes/{id}/restore` trước hạn, note được phục hồi về trạng thái active.
10. Nếu không khôi phục, **Amazon DynamoDB TTL** tự động xóa vĩnh viễn item khi `expiresAt` tới hạn.
11. Mọi bước trên đều được ghi log vào Amazon CloudWatch và trace bởi AWS X-Ray.
12. Nếu tỉ lệ lỗi Lambda vượt ngưỡng, CloudWatch Alarm được kích hoạt để cảnh báo.

---

## Kết quả

Sau khi hoàn thành chương này, bạn đã có cái nhìn tổng quan về kiến trúc
của dự án Smart Notes API và hiểu được cách các dịch vụ AWS phối hợp với
nhau trong toàn bộ hệ thống.

Ở các chương tiếp theo, chúng ta sẽ lần lượt triển khai từng thành phần
của kiến trúc, bắt đầu từ việc chuẩn bị môi trường, cấu hình IAM, và
triển khai hạ tầng bằng AWS SAM.