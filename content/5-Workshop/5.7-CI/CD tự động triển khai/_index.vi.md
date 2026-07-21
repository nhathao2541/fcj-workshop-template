---
title : "CI/CD tự động triển khai"
date : 2026-07-21
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

## Giới thiệu

Từ chương 5.4 đến 5.6, mọi lần thay đổi hạ tầng hay code đều được deploy
bằng cách gõ tay `sam build && sam deploy` trên máy cá nhân. Cách này có
2 rủi ro: dễ quên chạy test trước khi deploy, và phụ thuộc vào đúng máy,
đúng người có quyền AWS CLI mới deploy được.

Chương này thiết lập **CI/CD (Continuous Integration / Continuous
Deployment)** bằng **GitHub Actions**: mỗi khi code được push lên nhánh
`main`, pipeline sẽ tự động chạy unit test, build, và deploy lại toàn bộ
hạ tầng + code mới — không cần thao tác tay.

AWS không có "1 dịch vụ CI/CD" duy nhất; CI/CD là một quy trình mà bạn có
thể ráp từ AWS CodePipeline/CodeBuild, hoặc — như trong dự án này — từ
GitHub Actions (đơn giản hơn, không tốn thêm chi phí AWS, phù hợp quy mô
một ứng dụng cá nhân).

---

## Chuẩn bị: cấp quyền AWS cho GitHub Actions

GitHub Actions cần một IAM User (hoặc IAM Role qua OIDC) có đủ quyền chạy
`sam deploy`. Cách đơn giản nhất cho dự án quy mô nhỏ: tạo 1 IAM User
riêng, chỉ dùng cho CI/CD, gắn policy đủ dùng (CloudFormation, Lambda,
API Gateway, DynamoDB, S3, Cognito, IAM PassRole) — không dùng
`AdministratorAccess`.

Sau khi có Access Key ID/Secret Access Key, vào GitHub repo → **Settings
→ Secrets and variables → Actions** → thêm các secret sau:

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
GOOGLE_CLIENT_ID
GOOGLE_CLIENT_SECRET
```

Hai secret cuối chính là `GoogleClientId`/`GoogleClientSecret` đã dùng ở
chương 5.4 phần Cognito — vì đây là tham số `NoEcho`, SAM CLI sẽ không
lưu chúng vào `samconfig.toml`, nên pipeline phải tự truyền lại mỗi lần
deploy.

![Thêm Secrets trên GitHub](/images/5-Workshop/5.7/github-secrets.png)

---

## Khai báo workflow GitHub Actions

Tạo file `.github/workflows/deploy.yml`:

```yaml
name: Deploy Smart Notes API

on:
  push:
    branches: [main]

jobs:
  test-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "22"

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm test

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-southeast-1

      - name: Setup AWS SAM CLI
        uses: aws-actions/setup-sam@v2

      - name: SAM build
        run: sam build

      - name: SAM deploy
        run: |
          sam deploy \
            --stack-name smart-notes-api \
            --no-confirm-changeset \
            --no-fail-on-empty-changeset \
            --capabilities CAPABILITY_IAM \
            --parameter-overrides \
              GoogleClientId=${{ secrets.GOOGLE_CLIENT_ID }} \
              GoogleClientSecret=${{ secrets.GOOGLE_CLIENT_SECRET }}
```

Điểm cần chú ý so với deploy thủ công:

- `--no-confirm-changeset` và `--no-fail-on-empty-changeset`: bỏ qua các
  câu hỏi tương tác, vì pipeline chạy không người trực tiếp trả lời.
- `npm test` chạy **trước** bước deploy — nếu bất kỳ unit test nào ở
  chương trước fail, pipeline dừng lại ngay, không deploy code lỗi lên
  production.
- `--parameter-overrides` truyền lại 2 tham số Cognito/Google mỗi lần,
  bù cho việc `NoEcho` không lưu được vào `samconfig.toml`.

---

## Kiểm tra pipeline hoạt động

Push một thay đổi nhỏ (ví dụ sửa message lỗi trong `errors.js`) lên nhánh
`main`, sau đó vào tab **Actions** trên GitHub repo để theo dõi tiến
trình từng bước: Checkout → Install → Test → Deploy.

![Pipeline chạy thành công trên GitHub Actions](/images/5-Workshop/5.7/github-actions-run.png)

Nếu bước `Run unit tests` báo đỏ, pipeline dừng tại đó — hạ tầng AWS
hiện tại **không bị đụng tới**, vì bước deploy chưa từng được chạy.

---

## Kết quả

Sau chương này, mọi thay đổi code chỉ cần push lên nhánh `main` là tự
động được kiểm thử và triển khai lại lên AWS, không cần ai phải nhớ chạy
`sam build && sam deploy` bằng tay. Đây là bước cuối cùng để Smart Notes
API vận hành theo đúng quy trình DevOps cơ bản, trước khi sang chương tối
ưu chi phí và bảo mật.