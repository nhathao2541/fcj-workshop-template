---
title : "Environment Setup"
date : 2026-07-21
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

## Introduction

Before deploying the infrastructure, we need to prepare the required command-line tools and an AWS account with the appropriate permissions. This chapter guides you through installing **AWS CLI**, **AWS SAM CLI**, **Node.js**, and creating a dedicated **IAM User** for development instead of using the `root` account.

---

## Prerequisites

- An active AWS account.
- Node.js version **>= 22.x**.
- AWS CLI installed.
- AWS SAM CLI installed.
- A code editor (Visual Studio Code is recommended).

---

## Create an IAM User for Development

Do not use the `root` user's Access Key for any operations. Instead, create a dedicated IAM User with the minimum permissions required to deploy this project.

1. Sign in to the **AWS Management Console** and open the **IAM** service.
2. Select **Users** → **Create user**.
3. Enter a username, for example, `smart-notes-dev`.
4. Attach the required policies (least-privilege for development, not for the Lambda runtime):
   - `AWSCloudFormationFullAccess`
   - Permissions to create/delete Lambda, API Gateway, DynamoDB, S3, and IAM Roles (for learning purposes, you may temporarily attach `PowerUserAccess`, then restrict permissions before deploying to production).
5. Create an **Access Key** for this user (choose the **Command Line Interface (CLI)** use case).
6. Save the **Access Key ID** and **Secret Access Key**. The Secret Access Key is displayed only once.

![Create IAM User](/images/5-Workshop/5.3/create-iam-user.png)

---

## Install and Configure AWS CLI

```bash
aws --version
aws configure
```

Enter the following information:

```
AWS Access Key ID: <your-access-key>
AWS Secret Access Key: <your-secret-key>
Default region name: ap-southeast-1
Default output format: json
```

Verify the configuration:

```bash
aws sts get-caller-identity
```

If the returned `Account` and `Arn` match the newly created `smart-notes-dev` IAM User, the configuration is successful.

---

## Install AWS SAM CLI

```bash
# macOS
brew tap aws/tap
brew install aws-sam-cli

# Windows
# Download the .msi installer from the official AWS SAM CLI website
# and follow the installation instructions.
```

Verify the installation:

```bash
sam --version
```

---

## Verify Node.js

```bash
node --version   # must be >= 22.0.0
npm --version
```

If Node.js 22 is not installed, it is recommended to install it using **nvm** for easier version management:

```bash
nvm install 22
nvm use 22
```

---

## Clone the Project Repository

```bash
git clone <repo-url> smart-notes-api
cd smart-notes-api
npm install
```

The project structure used throughout this workshop is as follows:

```text
smart-notes-api/
├── template.yaml            # AWS SAM template defining the entire infrastructure
├── package.json
├── src/
│   ├── app.js               # Express application (routes, middleware)
│   ├── lambda.js            # Lambda entry point (serverless-http)
│   ├── config/              # DynamoDB and S3 client configuration
│   ├── routes/              # Express route definitions
│   ├── controllers/         # Handle requests and responses
│   ├── services/            # Business logic (validation, retry, presigned URLs, etc.)
│   ├── repositories/        # The only layer that communicates with DynamoDB
│   ├── utils/               # Response formatting, custom errors, validation
│   └── public/              # Static web frontend (index.html)
└── tests/                   # Unit tests (Jest)
```

---

## Verify the Environment

Run the following command:

```bash
sam validate --lint
```

If no errors are reported, your development environment is ready, and you can proceed to the next chapter to deploy the infrastructure using AWS SAM.