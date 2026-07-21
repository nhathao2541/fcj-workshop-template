---
title: "Project Proposal"
date: 2026-07-21
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
# Smart Notes API

## Serverless Note-Taking REST API Platform

Smart Notes API is a RESTful note management system built entirely on Amazon Web Services (AWS) using a fully serverless architecture. The system allows users to create, view, update, and delete notes, attach images, and interact with them through a static web interface inspired by a library card catalog. It also supports user authentication and API protection at the gateway layer.

---

# 1. Executive Summary

Many personal note-taking solutions require users to choose between using SaaS platforms (which reduce data ownership and increase vendor dependency) or maintaining traditional servers that incur continuous infrastructure costs even when idle.

Smart Notes API addresses this problem by adopting a **fully serverless architecture** on AWS. There are no continuously running servers, costs are incurred only when requests are processed, the system automatically scales with traffic, and all data remains under the owner's control through Amazon DynamoDB and Amazon S3.

The application follows **Clean Architecture** (Controller → Service → Repository) within AWS Lambda, making the codebase easier to maintain, test, and extend.

---

# 2. Problem Statement

## Challenges

Traditional note management APIs commonly face several issues:

- Servers remain running 24/7 even when there is little or no traffic.
- Scaling infrastructure during traffic spikes is complex.
- Infrastructure management (patching, scaling, load balancing) requires continuous maintenance.
- Public APIs without protection are vulnerable to abuse and automated attacks.
- Without authentication, anyone who knows the endpoint can access the API.

---

## Solution

Smart Notes API solves these challenges by using:

- Amazon API Gateway to receive all REST requests (GET/POST/PUT/DELETE).
- AWS Lambda (Node.js, Express, serverless-http) implementing Clean Architecture.
- Amazon DynamoDB for note storage using **PAY_PER_REQUEST** billing.
- Amazon S3 for image storage with automatic retry during temporary upload failures.
- Amazon API Gateway Usage Plans and API Keys to throttle requests and prevent abuse.
- Amazon Cognito (Hosted UI with Google Sign-In) to authenticate users before accessing notes.
- A lightweight HTML/CSS/JavaScript frontend communicating directly with the API.

---

# Benefits and Return on Investment (ROI)

Deploying Smart Notes API using a serverless architecture provides several advantages:

- Nearly zero infrastructure cost when idle (covered by the AWS Free Tier for most development scenarios).
- No server maintenance, operating system patching, or infrastructure management.
- Automatic scaling according to traffic.
- IAM Least Privilege permissions reduce security risks.
- The architecture can be reused for many CRUD-based applications such as task managers, bookmark systems, and journals.

---

# 3. Solution Architecture

The system follows a **Serverless** and **Clean Architecture** design.

System workflow:

```text
Client (Static HTML/CSS/JS)
         │  fetch/CORS + x-api-key + Authorization (Cognito ID Token)
         ▼
   Amazon API Gateway (REST API)
         │  Cognito Authorizer + Usage Plan/API Key
         ▼
     AWS Lambda (Express + serverless-http)
         │  Controller → Service → Repository
   ┌─────┴─────┐
   ▼           ▼
Amazon      Amazon S3
DynamoDB    (Note Images)
   │
   ▼
Amazon Cognito (User Pool + Hosted UI + Google Identity Provider)
   │
   ▼
Amazon CloudWatch (Logs)
```

---

# AWS Services Used

- **Amazon API Gateway** – Provides REST endpoints with API Keys and Cognito Authorizer.
- **AWS Lambda** – Executes serverless business logic using Node.js 22.
- **Amazon DynamoDB** – Stores note data using on-demand billing.
- **Amazon S3** – Stores note images.
- **Amazon Cognito** – Provides user authentication through email/password and Google Sign-In.
- **Amazon CloudWatch** – Monitors application logs and system metrics.
- **AWS IAM** – Implements Least Privilege access control.
- **AWS SAM (CloudFormation)** – Deploys and manages infrastructure as code.

---

# Solution Components

### Frontend

- Pure HTML/CSS/JavaScript
- Library card catalog-inspired interface
- Direct API communication using fetch
- Authentication through Amazon Cognito Hosted UI

### Backend

- Amazon API Gateway
- AWS Lambda implementing Clean Architecture

### Data Layer

- Amazon DynamoDB (Notes table)
- Amazon S3 (Image storage)

### Authentication

- Amazon Cognito User Pool
- Hosted UI
- Google Identity Provider

### Security & Rate Limiting

- API Gateway API Key
- Usage Plan

### Monitoring

- Amazon CloudWatch Logs

---

# 4. Technical Implementation

## Implementation Phases

### Phase 1

Design the system architecture, API endpoints, response formats, and validation rules.

### Phase 2

Configure IAM permissions, Amazon DynamoDB, and Amazon S3.

### Phase 3

Develop AWS Lambda using Clean Architecture and serverless-http.

### Phase 4

Deploy Amazon API Gateway and test CRUD operations together with image uploads.

### Phase 5

Develop the static frontend and integrate it with the API.

### Phase 6

Configure API Gateway Usage Plans and API Keys.

### Phase 7

Integrate Amazon Cognito Hosted UI and Google Sign-In.

### Phase 8

Enable CloudWatch logging and monitoring.

### Phase 9

Implement an automated CI/CD deployment pipeline.

### Phase 10

Perform end-to-end testing, including CRUD operations, authentication, image uploads, and security validation.

---

## Technical Requirements

- AWS Account
- AWS IAM User
- AWS Region (ap-southeast-1)
- AWS CLI
- AWS SAM CLI
- Node.js 22
- Google Cloud Console (OAuth Client for Cognito)
- Postman
- Git

---

# 5. Timeline & Milestones

| Phase | Status |
|------|--------|
| Completed | System architecture design, CRUD API, image upload, deployment |
| Completed | Static HTML frontend development |
| Completed | API Key and Usage Plan implementation |
| In Progress | Amazon Cognito Hosted UI and Google Sign-In |
| Next | CloudWatch logging guide |
| Next | CI/CD deployment pipeline |
| Future | Backend pagination, search, tags, and pinned notes |

---

# 6. Estimated Cost

The project can be developed almost entirely within the AWS Free Tier.

Reference:

https://calculator.aws

## Estimated Monthly Cost

| Service | Estimated Cost |
|----------|----------------|
| Amazon API Gateway | Free Tier |
| AWS Lambda | Free Tier |
| Amazon DynamoDB | Free Tier |
| Amazon S3 | ~1–2 USD |
| Amazon Cognito | Free Tier (up to 50,000 MAUs) |
| Amazon CloudWatch | ~1–2 USD |

**Estimated monthly cost:** **2–5 USD**.

---

# 7. Risk Assessment

## Risk Matrix

| Risk | Level |
|------|-------|
| API abuse before implementing security | High (mitigated by API Key and Cognito) |
| Forgotten AWS resources generating unexpected costs | Medium |
| Incorrect CORS or Cognito redirect configuration | Medium |
| Accidental data deletion without soft-delete | Medium |
| Heavy concurrent traffic | Low |

---

## Mitigation Strategy

- Monitor application logs with Amazon CloudWatch.
- Apply IAM Least Privilege principles.
- Protect APIs using API Keys, Usage Plans, and Cognito Authorizer.
- Configure AWS Billing Alarms.
- Remove infrastructure using `sam delete` when no longer needed.

---

## Backup Strategy

- Retry failed S3 uploads automatically.
- Store application logs in CloudWatch.
- Restore the entire infrastructure at any time using `template.yaml` (Infrastructure as Code).

---

# 8. Expected Outcomes

## Technical Improvements

- A complete Serverless REST API following Clean Architecture.
- Multi-layer security using API Keys, Usage Plans, and Cognito Authorizer.
- Flexible authentication with email/password and Google Sign-In.
- Infrastructure fully managed through AWS SAM.
- Comprehensive monitoring using Amazon CloudWatch.

---

## Long-Term Value

The Smart Notes API architecture can easily be extended to support:

- Personal to-do list applications.
- Bookmark management systems.
- Personal journals with image attachments.
- Team-based note sharing applications.
- Any lightweight CRUD application requiring rapid deployment and low operating costs.

This architecture provides a lightweight, cost-effective, maintainable, and scalable serverless solution that follows AWS architectural best practices for small- to medium-sized applications.