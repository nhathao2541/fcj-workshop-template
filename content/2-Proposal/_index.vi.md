---
title: "Bản đề xuất"
date: 2026-07-21
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
# Smart Notes API

## Serverless Note-Taking REST API Platform

Smart Notes API là hệ thống quản lý ghi chú (notes) dạng REST API, được xây dựng theo kiến trúc Serverless hoàn toàn trên Amazon Web Services (AWS). Hệ thống cho phép người dùng tạo, xem, sửa, xóa ghi chú và đính kèm hình ảnh, đi kèm giao diện web tĩnh phong cách "thẻ mục lục thư viện" (card catalog), có xác thực đăng nhập và bảo mật API ở tầng gateway.

---

# 1. Tóm tắt tổng quan

Các công cụ ghi chú cá nhân đơn giản thường phải đánh đổi giữa hai lựa chọn: dùng dịch vụ SaaS có sẵn (mất kiểm soát dữ liệu, phụ thuộc nhà cung cấp) hoặc tự vận hành server truyền thống (tốn chi phí duy trì 24/7 dù ít người dùng).

Dự án Smart Notes API giải quyết bài toán này bằng kiến trúc **Serverless hoàn toàn** trên AWS: không có máy chủ nào chạy thường trực, chi phí chỉ phát sinh khi có request thực tế, tự động mở rộng khi lượng truy cập tăng, và toàn quyền kiểm soát dữ liệu (DynamoDB + S3 riêng).

Hệ thống áp dụng **Clean Architecture** (Controller → Service → Repository) ngay trong tầng Lambda, giúp code dễ kiểm thử (unit test) và dễ mở rộng nghiệp vụ sau này.

---

# 2. Vấn đề cần giải quyết

## Vấn đề là gì?

Một số hạn chế khi xây dựng API quản lý ghi chú theo cách truyền thống:

- Server chạy 24/7 dù phần lớn thời gian không có request, gây lãng phí chi phí.
- Khó tự động mở rộng khi lượng người dùng tăng đột biến.
- Quản lý hạ tầng (patching, scaling, load balancer) tốn nhiều công sức vận hành.
- API mở công khai không có lớp bảo vệ dễ bị lạm dụng, quét tự động hoặc tấn công.
- Không có cơ chế xác thực người dùng, ai cũng có thể gọi API nếu biết URL.

---

## Giải pháp

Smart Notes API giải quyết bài toán bằng cách:

- API Gateway tiếp nhận toàn bộ request REST (GET/POST/PUT/DELETE).
- AWS Lambda (Node.js, Express + serverless-http) xử lý nghiệp vụ theo mô hình Clean Architecture.
- Amazon DynamoDB lưu trữ dữ liệu ghi chú (PAY_PER_REQUEST, không cần quản lý capacity).
- Amazon S3 lưu trữ hình ảnh đính kèm ghi chú, có retry khi upload lỗi.
- Amazon API Gateway Usage Plan + API Key giới hạn tốc độ gọi API (throttle/quota), chống lạm dụng.
- Amazon Cognito (Hosted UI + liên kết Google) bắt buộc đăng nhập trước khi truy cập ghi chú.
- Giao diện web tĩnh (HTML/CSS/JS thuần) gọi trực tiếp API qua fetch/CORS, không cần framework frontend.

---

# Lợi ích và hiệu quả đầu tư (ROI)

Việc triển khai Smart Notes API theo kiến trúc Serverless mang lại:

- Chi phí gần như bằng 0 khi không có traffic (Free Tier bao phủ phần lớn dịch vụ).
- Không cần quản lý, vá lỗi hay nâng cấp máy chủ.
- Tự động mở rộng khi số lượng người dùng/request tăng mà không cần cấu hình thêm.
- IAM Least Privilege: mỗi Lambda chỉ có đúng quyền cần thiết lên DynamoDB/S3, giảm rủi ro bảo mật.
- Có thể tái sử dụng kiến trúc này cho các API CRUD khác trong tương lai (to-do list, bookmark, v.v.).

---

# 3. Kiến trúc giải pháp

Kiến trúc được xây dựng theo mô hình **Serverless** và **Clean Architecture**.

Luồng hoạt động:

```
Client (Web tĩnh: HTML/CSS/JS)
         │  fetch/CORS + x-api-key + Authorization (Cognito ID Token)
         ▼
   Amazon API Gateway (REST API)
         │  Cognito Authorizer (bắt buộc đăng nhập) + Usage Plan/API Key
         ▼
     AWS Lambda (Express + serverless-http)
         │  Controller → Service → Repository
   ┌─────┴─────┐
   ▼           ▼
Amazon      Amazon S3
DynamoDB    (ảnh ghi chú)
   │
   ▼
Amazon Cognito (User Pool + Hosted UI + Google Identity Provider)
   │
   ▼
Amazon CloudWatch (Logs)
```

---

# Các dịch vụ AWS sử dụng

- **Amazon API Gateway**: Cung cấp REST API, áp Usage Plan/API Key và Cognito Authorizer.
- **AWS Lambda**: Xử lý nghiệp vụ Serverless (Node.js 22, Express + serverless-http).
- **Amazon DynamoDB**: Lưu trữ dữ liệu ghi chú (PAY_PER_REQUEST).
- **Amazon S3**: Lưu trữ hình ảnh đính kèm ghi chú.
- **Amazon Cognito**: Xác thực người dùng (đăng ký/đăng nhập email + Google), Hosted UI.
- **Amazon CloudWatch**: Giám sát và ghi log hệ thống, tra cứu lỗi.
- **IAM**: Quản lý quyền truy cập theo nguyên tắc Least Privilege.
- **AWS SAM (CloudFormation)**: Infrastructure as Code, đóng gói và triển khai toàn bộ hạ tầng.
- **Amazon DynamoDB Point-in-Time Recovery (PITR)**: Tự động sao lưu liên tục, cho phép khôi phục bảng Notes về bất kỳ thời điểm nào trong 35 ngày gần nhất.
- **CI/CD Pipeline**: Tự động hóa build & deploy (`sam build && sam deploy`) mỗi khi có thay đổi code, giảm rủi ro deploy thủ công sai sót.

---

# Thiết kế các thành phần

### Frontend

- HTML/CSS/JS thuần (1 file tĩnh, không build step)
- Giao diện "thẻ mục lục thư viện" (Fraunces + Courier Prime, dấu "FILED")
- Gọi trực tiếp API qua fetch, xác thực bằng Cognito Hosted UI

### Backend

- Amazon API Gateway
- AWS Lambda (Clean Architecture: Controller → Service → Repository)

### Data Layer

- Amazon DynamoDB (bảng Notes)
- Amazon S3 (bucket ảnh ghi chú)

### Authentication

- Amazon Cognito User Pool + Hosted UI
- Google Identity Provider (đăng nhập bằng Google)

### Security & Throttling

- Amazon API Gateway API Key + Usage Plan

### Monitoring

- Amazon CloudWatch Logs

---

# 4. Triển khai kỹ thuật

## Các giai đoạn triển khai

### Giai đoạn 1

Thiết kế kiến trúc & định nghĩa API (endpoints, response format, validation).

### Giai đoạn 2

Triển khai IAM, Amazon DynamoDB và Amazon S3 (least-privilege).

### Giai đoạn 3

Xây dựng Lambda theo Clean Architecture (Controller/Service/Repository) + serverless-http.

### Giai đoạn 4

Triển khai Amazon API Gateway, kiểm thử CRUD + upload/xóa ảnh qua Postman.

### Giai đoạn 5

Xây dựng Frontend HTML tĩnh, kết nối API qua fetch/CORS.

### Giai đoạn 6

Bổ sung bảo mật API Key + Usage Plan (throttle/quota).

### Giai đoạn 7

Tích hợp Amazon Cognito (Hosted UI + Google Identity Provider), bắt buộc đăng nhập.

### Giai đoạn 8

Thiết lập giám sát CloudWatch Logs, hướng dẫn tra cứu lỗi.

### Giai đoạn 9

Bật Point-in-Time Recovery (PITR) cho Amazon DynamoDB, bảo vệ dữ liệu ghi chú khỏi thao tác xóa/sửa nhầm.

### Giai đoạn 10

Thiết lập CI/CD tự động triển khai (build & deploy tự động khi push code).

### Giai đoạn 11

Kiểm thử toàn bộ hệ thống (CRUD, upload ảnh, đăng nhập/đăng xuất, bảo mật, khôi phục PITR).

---

## Yêu cầu kỹ thuật

- AWS Account.
- AWS IAM User.
- AWS Region (ap-southeast-1).
- AWS CLI.
- AWS SAM CLI.
- Node.js 22.
- Google Cloud Console (tạo OAuth Client cho Cognito).
- Postman.
- Git.

---

# 5. Thời gian & Mốc quan trọng

| Giai đoạn | Công việc |
|------|-----------|
| Đã hoàn thành | Thiết kế kiến trúc, xây dựng CRUD + upload ảnh, deploy thành công |
| Đã hoàn thành | Xây dựng frontend HTML tĩnh (edit note, xem chi tiết note) |
| Đã hoàn thành | Bảo mật API Key + Usage Plan |
| Đã hoàn thành | Tích hợp Cognito Hosted UI + đăng nhập Google |
| Đã hoàn thành | Bật Point-in-Time Recovery (PITR) cho DynamoDB |
| Đã hoàn thành | Thiết lập CI/CD tự động deploy |
| Kế tiếp | Hướng dẫn xem log CloudWatch khi lỗi |
| Dự kiến sau | Phân trang/tìm kiếm phía backend, tags, ghim note |

---

# 6. Dự toán chi phí

Có thể sử dụng AWS Free Tier trong quá trình phát triển và thử nghiệm.

Tham khảo:

https://calculator.aws

## Chi phí hạ tầng (ước tính)

| Dịch vụ | Chi phí/tháng |
|----------|---------------|
| Amazon API Gateway | Free Tier |
| AWS Lambda | Free Tier |
| Amazon DynamoDB | Free Tier |
| Amazon S3 | ~1–2 USD |
| Amazon Cognito | Free Tier (dưới 50,000 MAU) |
| Amazon CloudWatch | ~1–2 USD |

**Tổng chi phí thử nghiệm:** khoảng **2–5 USD/tháng**.

---

# 7. Đánh giá rủi ro

## Ma trận rủi ro

| Rủi ro | Mức độ |
|----------|---------|
| API bị lạm dụng/quét tự động khi chưa có bảo mật | Cao (đã giảm nhờ API Key + Cognito) |
| Quên xóa tài nguyên AWS sau khi ngừng dùng, phát sinh phí | Trung bình |
| Sai cấu hình CORS/redirect URI khi tích hợp Cognito + Google | Trung bình |
| Mất dữ liệu do thao tác xóa nhầm | Trung bình (đã giảm nhờ bật PITR trên DynamoDB) |
| Quá tải API khi nhiều người dùng cùng lúc | Thấp |

---

## Chiến lược giảm thiểu

- Sử dụng Amazon CloudWatch để giám sát lỗi và log.
- Áp dụng IAM Least Privilege cho từng Lambda.
- Giới hạn truy cập API bằng API Key + Usage Plan + Cognito Authorizer.
- Thiết lập Billing Alarm để theo dõi chi phí phát sinh.
- Bật Point-in-Time Recovery (PITR) trên DynamoDB để khôi phục dữ liệu khi cần.
- Triển khai qua CI/CD tự động, hạn chế lỗi do deploy thủ công.
- Có kế hoạch xóa toàn bộ tài nguyên (sam delete) khi ngừng sử dụng.

---

## Phương án dự phòng

- Retry khi upload ảnh lên S3 thất bại (đã có sẵn trong service).
- Lưu log lỗi trên CloudWatch để tra cứu nhanh.
- Có thể khôi phục hạ tầng bất kỳ lúc nào từ template.yaml (Infrastructure as Code).

---

# 8. Kết quả kỳ vọng

## Cải tiến kỹ thuật

- Xây dựng hoàn chỉnh REST API Serverless theo Clean Architecture.
- Bảo mật nhiều lớp: API Key/Usage Plan + Cognito Authorizer.
- Đăng nhập linh hoạt: email/mật khẩu hoặc Google.
- Toàn bộ hạ tầng được quản lý bằng Infrastructure as Code (AWS SAM).
- Có khả năng giám sát và tra cứu lỗi qua CloudWatch.
- Dữ liệu được bảo vệ bằng Point-in-Time Recovery (PITR), khôi phục được trong 35 ngày gần nhất.
- Quy trình triển khai tự động hóa qua CI/CD, giảm sai sót do deploy thủ công.

---

## Giá trị lâu dài

Kiến trúc Smart Notes API có thể tái sử dụng để mở rộng cho:

- Ứng dụng to-do list / task manager cá nhân.
- Ứng dụng bookmark/lưu trữ liên kết.
- Nhật ký cá nhân (journal) có đính kèm ảnh.
- Nền tảng ghi chú nhóm (team notes) nếu mở rộng phân quyền theo user.
- Bất kỳ API CRUD nhỏ nào cần triển khai nhanh, chi phí thấp trên AWS.

Đây là một giải pháp Serverless gọn nhẹ, tối ưu chi phí, dễ bảo trì và đáp ứng các tiêu chuẩn kiến trúc cơ bản của AWS cho một ứng dụng quy mô cá nhân/nhỏ.