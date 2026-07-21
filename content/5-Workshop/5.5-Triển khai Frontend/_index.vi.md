---
title : "Triển khai Frontend"
date : 2026-07-21
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

## Giới thiệu

Chương này triển khai giao diện web tĩnh cho Smart Notes API. Khác với
nhiều kiến trúc phổ biến dùng Amazon S3 Static Website Hosting kết hợp
Amazon CloudFront, workshop này phục vụ frontend **ngay tại chính AWS
Lambda** đang chạy backend — một quyết định thiết kế có chủ đích, phù hợp
với quy mô của một API cá nhân.

---

## So sánh hai phương án triển khai frontend

| Phương án | Ưu điểm | Khi nào nên dùng |
|---|---|---|
| **Serve tĩnh qua Lambda (đã chọn)** | Không cần thêm resource, không phát sinh phí CloudFront, cùng domain với API nên không lo CORS | Traffic thấp, muốn tối giản hạ tầng cho project cá nhân/MVP |
| Amazon S3 Static Website + CloudFront | Tốc độ tải toàn cầu, cache CDN, tách biệt frontend/backend | Traffic cao, cần custom domain riêng cho frontend, nhiều người dùng ở nhiều khu vực địa lý |

Đây là điểm có thể nâng cấp kiến trúc trong tương lai mà không cần đổi
API phía sau — chỉ cần chuyển phần serve tĩnh sang S3 + CloudFront.

![So sánh hai phương án frontend](/images/5-Workshop/5.5/frontend-options.png)

---

## Thêm route serve static vào Express

Vì project dùng ES Modules (`"type": "module"` trong `package.json`),
không có sẵn `__dirname` như CommonJS, cần tự dựng từ `import.meta.url`:

```js
import path from "path";
import { fileURLToPath } from "url";
import express from "express";

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

const app = express();

// Đặt TRƯỚC router /notes để "/" trả về giao diện thay vì rơi vào 404 handler
app.use(express.static(path.join(__dirname, "public")));

app.use("/notes", notesRouter);
```

Đặt file giao diện tại `src/public/index.html`. Vì `CodeUri: ./` trong
`template.yaml` đóng gói toàn bộ thư mục project, file này tự động được
deploy cùng Lambda mà **không cần khai báo thêm resource nào**.

---

## Vì sao route gốc `/` phải khai báo riêng trong API Gateway

API Gateway REST API mặc định chỉ định tuyến những path khớp `{proxy+}`
— request tới path gốc `/` **không tự động** đi qua Lambda proxy nếu chỉ
khai báo `{proxy+}`. Đây là lý do `template.yaml` ở chương 5.4 phải khai
báo cả hai event:

```yaml
Events:
  RootApi:
    Type: Api
    Properties: { Path: /, Method: ANY }
  ProxyApi:
    Type: Api
    Properties: { Path: /{proxy+}, Method: ANY }
```

Thiếu route `RootApi` là nguyên nhân phổ biến nhất khiến trang chủ trả
về lỗi 403/404 dù mọi API con vẫn hoạt động bình thường.

---

## Build và deploy lại

```bash
sam build && sam deploy
```

Kiểm tra kết quả:

```bash
curl -I https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev/
```

Kỳ vọng: `HTTP/2 200` và header `content-type: text/html`.

Mở URL đó trên trình duyệt — giao diện đã có thể gọi thẳng API cùng domain.

![Giao diện Smart Notes hoạt động](/images/5-Workshop/5.5/frontend-running.png)

---

## Kết quả

Sau chương này, người dùng đã có thể truy cập giao diện web đầy đủ chức
năng: tạo/sửa/xóa note, đính kèm ảnh, tìm kiếm, sắp xếp, và xem thùng
rác — tất cả thông qua một URL duy nhất do API Gateway cung cấp. Chương
tiếp theo sẽ hướng dẫn kiểm thử và đo lường hệ thống.