---
title : "Giới thiệu"
date : 2026-07-21
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---
Smart Notes API là hệ thống quản lý ghi chú cá nhân (notes) được xây
dựng hoàn toàn trên nền tảng AWS Cloud theo kiến trúc **Serverless** và
**Clean Architecture**, không cần quản lý hay vận hành bất kỳ máy chủ
nào.

Hệ thống hỗ trợ đầy đủ các thao tác:

+ Tạo, xem, sửa, xóa ghi chú (CRUD).
+ Đính kèm và xóa ảnh minh họa cho từng ghi chú.
+ Xem lại ảnh một cách an toàn thông qua URL có chữ ký tạm thời (presigned URL), không public bucket.
+ Khôi phục ghi chú vừa xóa trong vòng 3 ngày (thùng rác), tự động xóa vĩnh viễn sau đó.
+ Khôi phục toàn bộ bảng dữ liệu về một thời điểm bất kỳ trong 35 ngày gần nhất nhờ Point-in-Time Recovery (PITR), phòng trường hợp xóa/sửa nhầm ngoài ý muốn.

Toàn bộ hạ tầng được khai báo bằng **AWS SAM** (Infrastructure as Code) —
triển khai lặp lại được, xóa sạch bằng một lệnh, không cấu hình tay trên
Console. Quy trình build và deploy này còn được tự động hóa thêm một
bước bằng **CI/CD**, giúp mọi thay đổi code được kiểm thử và triển khai
lại mà không cần chạy lệnh thủ công mỗi lần.

---

#### Mục tiêu của Workshop

Trong workshop này, bạn sẽ triển khai lại **end-to-end** một REST API
Serverless từ đầu đến cuối. Sau khi hoàn thành workshop, bạn sẽ có thể:

+ Xây dựng API Serverless bằng **Amazon API Gateway** và **AWS Lambda**.
+ Lưu trữ dữ liệu ghi chú trên **Amazon DynamoDB**, dùng TTL để tự động dọn dữ liệu hết hạn.
+ Bảo vệ dữ liệu ghi chú bằng **Point-in-Time Recovery (PITR)** trên DynamoDB, khôi phục được về bất kỳ thời điểm nào trong 35 ngày gần nhất.
+ Lưu trữ và bảo mật hình ảnh trên **Amazon S3** (private bucket + presigned URL).
+ Quản lý toàn bộ hạ tầng bằng **AWS SAM / AWS CloudFormation** (Infrastructure as Code).
+ Phân quyền least-privilege cho Lambda bằng **AWS IAM**.
+ Giám sát và trace request phân tán bằng **Amazon CloudWatch** và **AWS X-Ray**.
+ Kiểm thử API bằng unit test (Jest) và Postman/curl.
+ Tự động hóa build & deploy bằng **CI/CD** (GitHub Actions), loại bỏ thao tác deploy thủ công.
+ Tối ưu chi phí, áp dụng bảo mật cơ bản, và dọn dẹp tài nguyên đúng cách.

---

#### Tổng quan kiến trúc hệ thống

Kiến trúc Smart Notes API được chia thành các thành phần chính:

+ **Frontend Layer**
  Giao diện web tĩnh (HTML/CSS/JS thuần) được phục vụ trực tiếp qua
  **AWS Lambda** (Express serve static), gọi thẳng API cùng domain —
  không cần dựng thêm S3 Static Website hay CloudFront cho quy mô
  workshop này.

+ **Application Layer**
  Mọi request từ client được tiếp nhận bởi **Amazon API Gateway** (REST
  API), định tuyến toàn bộ qua một **AWS Lambda** duy nhất chạy Express
  (serverless-http), tổ chức theo Clean Architecture 3 tầng: Controller
  → Service → Repository.

+ **Data Layer**
  Dữ liệu ghi chú được lưu trên **Amazon DynamoDB** (chế độ
  PAY_PER_REQUEST, có bật TTL cho tính năng thùng rác và Point-in-Time
  Recovery để bảo vệ dữ liệu); ảnh đính kèm được lưu trên **Amazon S3**
  (private hoàn toàn, truy cập qua presigned URL).

+ **Monitoring Layer**
  Toàn bộ log invocation và trace request được quản lý bởi **Amazon
  CloudWatch** và **AWS X-Ray**, hỗ trợ debug hệ thống serverless không
  có máy chủ để truy cập trực tiếp.

+ **Security Layer**
  **AWS IAM** cấp quyền least-privilege cho Lambda (chỉ đúng bảng
  DynamoDB và bucket S3 của project), không dùng quyền quản trị toàn
  cục.

+ **CI/CD Layer**
  Mọi thay đổi code được **GitHub Actions** tự động build, chạy unit
  test và deploy lại lên AWS, loại bỏ thao tác `sam deploy` thủ công.

---

#### Kiến trúc triển khai

Trong workshop này, bạn sẽ triển khai các thành phần sau:

+ **Application Layer**
  + Amazon API Gateway
  + AWS Lambda
+ **Data Storage**
  + Amazon DynamoDB (kèm TTL và Point-in-Time Recovery)
  + Amazon S3
+ **Frontend**
  + Giao diện tĩnh phục vụ qua Lambda (Express static)
+ **Monitoring & Security**
  + Amazon CloudWatch
  + AWS X-Ray
  + IAM Roles (least-privilege)
+ **Infrastructure as Code**
  + AWS SAM / AWS CloudFormation
+ **CI/CD**
  + GitHub Actions (tự động test & deploy)

Workshop này được xây dựng nhằm giúp người học làm quen với kiến trúc
**Serverless**, nguyên tắc **Clean Architecture**, và các thực hành cơ
bản của **AWS Well-Architected Framework** — đặc biệt là bốn trụ cột
**Cost Optimization**, **Security**, **Reliability** (nhờ PITR) và
**Operational Excellence** (nhờ CI/CD).

---

![Smart Notes API Architecture](/images/5-Workshop/5.1/smart-notes-architecture.png)