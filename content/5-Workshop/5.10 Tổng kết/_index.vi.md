---
title : "Tổng kết"
date : 2026-07-21
weight : 10
chapter : false
pre : " <b> 5.10. </b> "
---

## Giới thiệu

Xin chúc mừng!

Bạn đã hoàn thành Workshop **Smart Notes API – Serverless Notes
Management Platform** trên nền tảng **Amazon Web Services (AWS)**.

Thông qua Workshop này, bạn đã triển khai thành công một REST API quản
lý ghi chú cá nhân theo kiến trúc **Cloud Native**, **Serverless** và
**Clean Architecture**, có khả năng tự động scale theo traffic, không
tốn chi phí khi không có request, và tự động dọn dữ liệu hết hạn mà
không cần bất kỳ tiến trình lịch (cron) nào.

---

# Các dịch vụ AWS đã triển khai

Trong Workshop, chúng ta đã sử dụng các dịch vụ sau:

+ AWS Identity and Access Management (IAM)

+ Amazon API Gateway

+ AWS Lambda

+ Amazon DynamoDB

+ Amazon S3

+ Amazon CloudWatch

+ AWS X-Ray

+ AWS CloudFormation (thông qua AWS SAM)

---

# Kiến trúc hoàn chỉnh

Toàn bộ hệ thống sau khi triển khai sẽ hoạt động theo kiến trúc dưới đây.

![Final Architecture](/images/5-Workshop/5.1/smart-notes-architecture.png)

---

## Luồng hoạt động của hệ thống

Quy trình xử lý hoàn chỉnh bao gồm:

1. Người dùng truy cập giao diện web, được phục vụ trực tiếp qua AWS Lambda.

2. Client gửi request tạo/sửa/xóa note đến Amazon API Gateway.

3. Amazon API Gateway định tuyến request đến AWS Lambda.

4. AWS Lambda validate input và xử lý nghiệp vụ theo Clean Architecture (Controller → Service → Repository).

5. AWS Lambda ghi/đọc dữ liệu ghi chú trên Amazon DynamoDB.

6. Client upload ảnh đính kèm qua `POST /notes/{id}/image`.

7. AWS Lambda upload ảnh lên Amazon S3 (bucket private hoàn toàn), có retry khi lỗi tạm thời.

8. Khi trả danh sách note, AWS Lambda ký lại (presign) URL ảnh trước khi trả về client.

9. Khi note bị xóa, AWS Lambda đánh dấu soft-delete; Amazon DynamoDB TTL tự động xóa vĩnh viễn sau 3 ngày nếu không được khôi phục.

10. Amazon CloudWatch ghi Logs và Metrics của mọi lần Lambda invoke.

11. AWS X-Ray trace từng request qua API Gateway → Lambda → DynamoDB/S3.

12. AWS IAM đảm bảo Lambda chỉ có quyền least-privilege trên đúng bảng DynamoDB và bucket S3 của hệ thống.

---

# Những kiến thức đạt được

Sau khi hoàn thành Workshop, bạn đã có thể:

✔ Thiết kế kiến trúc Serverless hoàn chỉnh trên AWS.

✔ Áp dụng Clean Architecture (Controller → Service → Repository) cho một Lambda function.

✔ Xây dựng REST API bằng Amazon API Gateway và AWS Lambda.

✔ Lưu trữ dữ liệu bằng Amazon DynamoDB, sử dụng TTL để tự động dọn dữ liệu hết hạn.

✔ Lưu trữ và bảo mật hình ảnh bằng Amazon S3 (private bucket + presigned URL).

✔ Quản lý toàn bộ hạ tầng bằng AWS SAM / AWS CloudFormation (Infrastructure as Code).

✔ Phân quyền least-privilege cho Lambda bằng AWS IAM.

✔ Giám sát và trace request phân tán bằng Amazon CloudWatch và AWS X-Ray.

✔ Kiểm thử hệ thống bằng unit test (Jest) và gọi API thật (Postman/curl).

✔ Tối ưu chi phí và áp dụng các biện pháp bảo mật cơ bản.

✔ Dọn dẹp toàn bộ tài nguyên đúng cách để tránh phát sinh chi phí.

---

# Hướng phát triển

Trong tương lai, hệ thống có thể được mở rộng theo nhiều hướng:

+ **Amazon Cognito** hoặc API Key (`UsagePlan`) để xác thực và phân quyền người dùng thật.

+ **Amazon S3 Static Website + Amazon CloudFront** để tách frontend khỏi Lambda, phục vụ traffic cao và tốc độ tải toàn cầu.

+ **Amazon OpenSearch Service** để hỗ trợ tìm kiếm full-text mạnh hơn thay vì lọc phía client.

+ **Amazon SNS** gắn với CloudWatch Alarm để gửi cảnh báo tức thời khi hệ thống có sự cố.

+ **AWS WAF** bảo vệ API Gateway khỏi các cuộc tấn công phổ biến (SQL injection, rate-based attack).

+ **GitHub Actions / AWS CodePipeline** để tự động hóa CI/CD, build và deploy mỗi khi có commit mới.

+ **Amazon DynamoDB Streams + AWS Lambda** để dọn ảnh trên S3 mồ côi khi note bị TTL tự xóa vĩnh viễn.

+ **Custom Domain (Route 53 + ACM)** thay cho URL mặc định của API Gateway.

---

# Kết quả cuối cùng

Sau khi hoàn thành Workshop, bạn đã xây dựng thành công một hệ thống
**Smart Notes API** hoạt động hoàn toàn trên AWS.

Hệ thống đáp ứng đầy đủ các yêu cầu:

+ Quản lý ghi chú (tạo, sửa, xóa, xem).

+ Đính kèm và hiển thị ảnh một cách an toàn.

+ Khôi phục dữ liệu đã xóa trong thời gian giới hạn.

+ Tự động dọn dữ liệu hết hạn mà không cần can thiệp thủ công.

+ Giám sát và trace toàn bộ hệ thống theo thời gian thực.

![Workshop Result](/images/5-Workshop/5.9/workshop-result.png)

---

# Kết luận

Workshop **Smart Notes API – Serverless Notes Management Platform** là
một ví dụ hoàn chỉnh về việc xây dựng REST API trên nền tảng AWS theo mô
hình **Cloud Native**, **Serverless** và **Clean Architecture**.

Thông qua Workshop này, bạn đã thực hành triển khai một kiến trúc hiện
đại sử dụng các dịch vụ AWS từ xử lý API, lưu trữ dữ liệu, quản lý vòng
đời dữ liệu, cho đến giám sát hệ thống — tất cả được quản lý hoàn toàn
bằng Infrastructure as Code.

Kiến trúc được xây dựng theo các nguyên tắc của **AWS Well-Architected
Framework**, đảm bảo tính bảo mật, khả năng mở rộng, hiệu năng, độ tin
cậy và đặc biệt là tối ưu chi phí — phù hợp cho một project cá nhân/MVP
nhưng vẫn có lộ trình rõ ràng để nâng cấp lên quy mô production.

Xin cảm ơn bạn đã tham gia Workshop!