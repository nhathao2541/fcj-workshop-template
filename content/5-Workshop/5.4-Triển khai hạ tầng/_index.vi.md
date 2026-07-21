---
title : "Triển khai hạ tầng"
date : 2026-07-21
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

## Giới thiệu

Chương này triển khai toàn bộ hạ tầng backend của Smart Notes API bằng
**AWS SAM** — Amazon DynamoDB, Amazon S3, AWS Lambda, Amazon API Gateway,
và IAM Role least-privilege — tất cả được định nghĩa trong một file duy
nhất: `template.yaml`. Đây là cách tiếp cận **Infrastructure as Code**:
không thao tác tay trên Console, mọi thay đổi hạ tầng đều đi qua code, có
thể review qua Git và triển khai lại y hệt ở bất kỳ tài khoản AWS nào.

---

## Khai báo Amazon DynamoDB

```yaml
NotesTable:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: !Sub "Notes-${StageName}"
    BillingMode: PAY_PER_REQUEST      # không cần dự đoán capacity trước
    AttributeDefinitions:
      - AttributeName: id
        AttributeType: S
    KeySchema:
      - AttributeName: id
        KeyType: HASH
    TimeToLiveSpecification:           # phục vụ tính năng "thùng rác 3 ngày"
      AttributeName: expiresAt
      Enabled: true
```

`PAY_PER_REQUEST` (On-Demand) được chọn vì traffic của một ứng dụng ghi
chú cá nhân rất thất thường — trả tiền đúng theo số request thực tế, không
phải trả cho capacity đặt trước mà không dùng hết.

---

## Bật Point-in-Time Recovery (PITR) cho DynamoDB

`TimeToLiveSpecification` ở trên chỉ xử lý việc **tự động xóa** các note
đã hết hạn trong thùng rác — nó không giúp gì nếu người dùng (hoặc chính
code có bug) **xóa/sửa nhầm** một note đang hoạt động. Đây là lúc cần đến
**Point-in-Time Recovery (PITR)**, một tính năng có sẵn của DynamoDB: tự
động sao lưu liên tục toàn bộ thay đổi trên bảng, cho phép khôi phục bảng
về **bất kỳ thời điểm nào trong 35 ngày gần nhất** mà không cần tự dựng
cơ chế backup riêng.

Thêm đúng 3 dòng vào ngay dưới `TimeToLiveSpecification` của `NotesTable`:

```yaml
NotesTable:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: !Sub "Notes-${StageName}"
    BillingMode: PAY_PER_REQUEST
    AttributeDefinitions:
      - AttributeName: id
        AttributeType: S
    KeySchema:
      - AttributeName: id
        KeyType: HASH
    TimeToLiveSpecification:
      AttributeName: expiresAt
      Enabled: true
    PointInTimeRecoverySpecification:  # <-- thêm mới
      PointInTimeRecoveryEnabled: true
```

PITR không tính thêm chi phí cố định — chỉ tính theo dung lượng dữ liệu
đã lưu, và với quy mô một ứng dụng ghi chú cá nhân, chi phí phát sinh gần
như không đáng kể. Đổi lại, đây là lớp bảo vệ cuối cùng cho dữ liệu note
của người dùng trước các thao tác xóa/sửa ngoài ý muốn.

Khi cần khôi phục, thao tác qua AWS CLI (khôi phục ra một bảng mới, không
ghi đè bảng đang chạy):

```bash
aws dynamodb restore-table-to-point-in-time \
  --source-table-name Notes-dev \
  --target-table-name Notes-dev-restored \
  --restore-date-time "2026-07-20T10:00:00Z"
```

![PITR bật trên bảng Notes](/images/5-Workshop/5.4/dynamodb-pitr-enabled.png)

---

## Khai báo Amazon S3 (bucket ảnh, private hoàn toàn)

```yaml
NotesBucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: !Sub "${BucketNamePrefix}-${AWS::AccountId}-${StageName}"
    PublicAccessBlockConfiguration:
      BlockPublicAcls: true
      BlockPublicPolicy: true
      IgnorePublicAcls: true
      RestrictPublicBuckets: true
    CorsConfiguration:
      CorsRules:
        - AllowedHeaders: ["*"]
          AllowedMethods: [GET, PUT, POST, DELETE]
          AllowedOrigins: ["*"]
          MaxAge: 3000
```

Bucket chặn public access ở cả 4 cờ — ảnh chỉ có thể truy cập thông qua
presigned URL được Lambda ký tạm thời (xem chi tiết ở chương 5.6).

---

## Khai báo AWS Lambda và IAM least-privilege

```yaml
SmartNotesFunction:
  Type: AWS::Serverless::Function
  Properties:
    FunctionName: !Sub "smart-notes-api-${StageName}"
    CodeUri: ./
    Handler: src/lambda.handler
    Runtime: nodejs22.x
    Tracing: Active                 # bật AWS X-Ray
    Environment:
      Variables:
        NOTES_TABLE_NAME: !Ref NotesTable
        NOTES_BUCKET_NAME: !Ref NotesBucket
    Policies:                        # least-privilege - KHÔNG dùng AdministratorAccess
      - DynamoDBCrudPolicy:
          TableName: !Ref NotesTable
      - S3CrudPolicy:
          BucketName: !Ref NotesBucket
    Events:
      RootApi:
        Type: Api
        Properties: { Path: /, Method: ANY }
      ProxyApi:
        Type: Api
        Properties: { Path: /{proxy+}, Method: ANY }
```

`DynamoDBCrudPolicy` và `S3CrudPolicy` là các policy template có sẵn của
SAM, tự sinh ra IAM policy JSON chỉ cho phép thao tác trên đúng
table/bucket được tham chiếu. Lambda không có quyền động vào bất kỳ tài
nguyên AWS nào khác trong tài khoản, kể cả khi code có lỗ hổng bảo mật.

![IAM Role least-privilege](/images/5-Workshop/5.4/iam-role-policy.png)

---

## Khai báo Amazon API Gateway

```yaml
SmartNotesApi:
  Type: AWS::Serverless::Api
  Properties:
    StageName: !Ref StageName
    BinaryMediaTypes:
      - "multipart/form-data"   # bắt buộc để nhận file ảnh qua API Gateway
      - "image/*"
    Cors:
      AllowMethods: "'GET,POST,PUT,DELETE,OPTIONS'"
      AllowHeaders: "'Content-Type,Authorization'"
      AllowOrigin: "'*'"
```

---

## Build và Deploy

```bash
sam build
sam deploy --guided
```

Trả lời các câu hỏi hướng dẫn:

```
Stack Name: smart-notes-api
AWS Region: ap-southeast-1
Parameter StageName: dev
Confirm changes before deploy: Y
Allow SAM CLI IAM role creation: Y
Save arguments to configuration file: Y
```

Sau khi deploy xong, SAM in ra phần **Outputs**:

```
ApiUrl: https://xxxxxxxxxx.execute-api.ap-southeast-1.amazonaws.com/dev/
NotesTableName: Notes-dev
NotesBucketName: smart-notes-storage-xxxxxxxxxxxx-dev
```

Đây chính là endpoint gốc cho toàn bộ route REST: `GET/POST/PUT/DELETE
/notes`, upload/xóa ảnh, xem/khôi phục thùng rác.

Từ lần deploy thứ hai trở đi (khi sửa code), chỉ cần:

```bash
sam build && sam deploy
```

![Kết quả deploy thành công](/images/5-Workshop/5.4/sam-deploy-output.png)

---

## Kết quả

Sau chương này, toàn bộ hạ tầng backend đã sẵn sàng trên AWS: DynamoDB
(có bật PITR để bảo vệ dữ liệu), S3, Lambda, API Gateway đều đã được tạo
tự động thông qua CloudFormation (bên dưới AWS SAM). Chương tiếp theo sẽ
triển khai giao diện web cho người dùng cuối.