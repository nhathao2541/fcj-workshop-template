---
title : "Workshop"
date : 2026-07-21
weight : 5
chapter : true
pre : "<b>5.</b>"
---

# Workshop

## Giới thiệu

Trong phần này, chúng ta sẽ triển khai **Smart Notes API – Nền tảng
Quản lý Ghi chú Serverless** trên **Amazon Web Services (AWS)**.

Dự án này là một REST API quản lý ghi chú cá nhân (tạo, cập nhật,
xoá, đính kèm hình ảnh, và khôi phục ghi chú đã xoá) được xây dựng
theo kiến trúc **Cloud Native** và **Serverless** hoàn toàn. Dự án
tuân theo mô hình **Clean Architecture** (Controller → Service →
Repository), tách biệt rõ ràng logic nghiệp vụ khỏi các chi tiết hạ
tầng AWS.

Workshop này được xây dựng dựa trên phương pháp **Infrastructure as
Code (IaC)** sử dụng **AWS SAM**, trong đó toàn bộ quy trình—từ việc
tiếp nhận request của client, lưu trữ dữ liệu trong Amazon DynamoDB,
lưu trữ hình ảnh an toàn trong Amazon S3, cho đến giám sát hệ thống
bằng Amazon CloudWatch và AWS X-Ray—đều được định nghĩa và quản lý
hoàn toàn bằng code mà không cần cấu hình thủ công trên AWS Console.

Quy trình triển khai còn được tự động hoá hơn nữa bằng **CI/CD**,
đảm bảo mỗi thay đổi code đều được kiểm thử và triển khai tự động mà
không cần can thiệp thủ công.

{{% notice tip %}}
🎬 **Xem video demo toàn bộ hệ thống trước khi bắt đầu:** [Video Demo Smart Notes API](https://drive.google.com/file/d/1tQAHcoE9rRgxeoYFCjHok-B_4vdAjKIs/view?usp=sharing)

💻 **Mã nguồn:** [Smart Notes API - GitHub Repository](https://github.com/danghoang11524/smart-notes-api.git)
{{% /notice %}}

---

## Mục tiêu

Sau khi hoàn thành workshop này, bạn sẽ có thể:

- Triển khai một kiến trúc Serverless hoàn chỉnh trên AWS.
- Xây dựng REST API bằng Amazon API Gateway và AWS Lambda theo mô hình Clean Architecture.
- Lưu trữ dữ liệu ứng dụng trong Amazon DynamoDB và sử dụng Time-to-Live (TTL) để tự động dọn dẹp dữ liệu hết hạn.
- Bảo vệ dữ liệu ghi chú bằng Point-in-Time Recovery (PITR), cho phép khôi phục trong vòng 35 ngày trước đó.
- Lưu trữ và bảo mật hình ảnh bằng Amazon S3 (bucket riêng tư với Presigned URLs).
- Quản lý toàn bộ hạ tầng bằng AWS SAM và AWS CloudFormation (Infrastructure as Code).
- Áp dụng nguyên tắc bảo mật least-privilege bằng AWS IAM.
- Giám sát log và distributed traces bằng Amazon CloudWatch và AWS X-Ray.
- Kiểm thử ứng dụng bằng Jest và các request API thực tế với Postman/curl.
- Tự động hoá quá trình build và deploy bằng CI/CD (GitHub Actions), loại bỏ việc triển khai thủ công.
- Tối ưu chi phí và áp dụng các best practice bảo mật cơ bản.
- Dọn dẹp tài nguyên AWS đúng cách để tránh phát sinh chi phí ngoài ý muốn.

---

## Kiến trúc hệ thống

Hệ thống bao gồm các thành phần chính sau:

- Amazon API Gateway tiếp nhận và định tuyến các request từ client.
- AWS Lambda (Node.js, Express + serverless-http) xử lý toàn bộ logic nghiệp vụ.
- Amazon DynamoDB lưu trữ dữ liệu ghi chú, với Time-to-Live (TTL) được kích hoạt cho tính năng thùng rác và Point-in-Time Recovery (PITR) được kích hoạt để bảo vệ khỏi việc cập nhật hoặc xoá nhầm.
- Amazon S3 lưu trữ hình ảnh đính kèm trong một bucket hoàn toàn riêng tư.
- AWS IAM cấp quyền least-privilege cho Lambda.
- Amazon CloudWatch và AWS X-Ray cung cấp khả năng ghi log và truy vết request phân tán.
- Giao diện web tĩnh được phục vụ trực tiếp thông qua AWS Lambda mà không cần Amazon CloudFront.
- AWS SAM và AWS CloudFormation quản lý toàn bộ hạ tầng dưới dạng code.
- Pipeline CI/CD được vận hành bởi GitHub Actions tự động kiểm thử và triển khai lại ứng dụng mỗi khi có thay đổi code được push lên.

---

## Nội dung Workshop

Workshop bao gồm các chương sau:

**5.1.** Giới thiệu

**5.2.** Kiến trúc hệ thống

**5.3.** Thiết lập môi trường

**5.4.** Triển khai hạ tầng

**5.5.** Triển khai Frontend

**5.6.** Kiểm thử và Giám sát

**5.7.** Tự động hoá triển khai CI/CD

**5.8.** Tối ưu Chi phí và Bảo mật

**5.9.** Dọn dẹp Tài nguyên

**5.10.** Kết luận

---

## Kết quả mong đợi

Sau khi hoàn thành workshop này, bạn sẽ triển khai thành công **Smart
Notes API** trên AWS với đầy đủ các thành phần, bao gồm REST API, cơ
sở dữ liệu, lưu trữ hình ảnh, giám sát, bảo mật cơ bản, và một pipeline
triển khai tự động.

Kiến trúc này tuân theo các nguyên tắc của **AWS Well-Architected
Framework**, đặc biệt tập trung vào bốn trụ cột chính:

- **Tối ưu Chi phí** (không tốn chi phí khi không hoạt động, tự động dọn dẹp bằng TTL)
- **Bảo mật** (IAM least-privilege, bucket Amazon S3 riêng tư với Presigned URLs)
- **Độ tin cậy** (Point-in-Time Recovery cho Amazon DynamoDB)
- **Vận hành xuất sắc** (kiểm thử và triển khai tự động thông qua CI/CD)

Đồng thời, hệ thống vẫn duy trì khả năng mở rộng và tính sẵn sàng cao
nhờ tận dụng kiến trúc Serverless hoàn toàn.