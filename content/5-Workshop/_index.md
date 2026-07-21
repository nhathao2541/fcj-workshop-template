---
title : "Workshop"
date : 2026-07-21
weight : 5
chapter : true
pre : "<b>5.</b>"
---

# Workshop

## Introduction

In this section, we will deploy the **Smart Notes API – Serverless Notes
Management Platform** on **Amazon Web Services (AWS)**.

This project is a personal note management REST API (create, update,
delete, attach images, and restore deleted notes) built using a fully
**Cloud Native** and **Serverless** architecture. It follows the
**Clean Architecture** pattern (Controller → Service → Repository),
clearly separating business logic from AWS infrastructure details.

The workshop is based on the **Infrastructure as Code (IaC)** approach
using **AWS SAM**, where the entire workflow—from receiving client
requests, storing data in Amazon DynamoDB, securely storing images in
Amazon S3, to monitoring the system with Amazon CloudWatch and AWS
X-Ray—is defined and managed entirely through code without manual
configuration in the AWS Console.

The deployment workflow is further automated using **CI/CD**, ensuring
that every code change is automatically tested and deployed without
manual intervention.

{{% notice tip %}}
🎬 **Watch the complete system demo before getting started:** [Smart Notes API Demo Video](https://drive.google.com/file/d/1tQAHcoE9rRgxeoYFCjHok-B_4vdAjKIs/view?usp=sharing)

💻 **Source code:** [Smart Notes API - GitHub Repository](https://github.com/danghoang11524/smart-notes-api.git)
{{% /notice %}}

---

## Objectives

After completing this workshop, you will be able to:

- Deploy a complete Serverless architecture on AWS.
- Build a REST API using Amazon API Gateway and AWS Lambda following the Clean Architecture pattern.
- Store application data in Amazon DynamoDB and use Time-to-Live (TTL) for automatic cleanup of expired data.
- Protect note data using Point-in-Time Recovery (PITR), allowing recovery within the previous 35 days.
- Store and secure images using Amazon S3 (private bucket with Presigned URLs).
- Manage the entire infrastructure using AWS SAM and AWS CloudFormation (Infrastructure as Code).
- Apply the least-privilege security principle using AWS IAM.
- Monitor logs and distributed traces with Amazon CloudWatch and AWS X-Ray.
- Test the application using Jest and real API requests with Postman/curl.
- Automate build and deployment using CI/CD (GitHub Actions), eliminating manual deployment.
- Optimize costs and implement fundamental security best practices.
- Properly clean up AWS resources to avoid unexpected charges.

---

## System Architecture

The system consists of the following major components:

- Amazon API Gateway receives and routes client requests.
- AWS Lambda (Node.js, Express + serverless-http) handles all business logic.
- Amazon DynamoDB stores note data, with Time-to-Live (TTL) enabled for the recycle bin feature and Point-in-Time Recovery (PITR) enabled for protection against accidental updates or deletions.
- Amazon S3 stores attached images in a completely private bucket.
- AWS IAM grants least-privilege permissions to Lambda.
- Amazon CloudWatch and AWS X-Ray provide logging and distributed request tracing.
- A static web interface is served directly through AWS Lambda without requiring Amazon CloudFront.
- AWS SAM and AWS CloudFormation manage the entire infrastructure as code.
- A CI/CD pipeline powered by GitHub Actions automatically tests and redeploys the application whenever code changes are pushed.

---

## Workshop Contents

The workshop consists of the following chapters:

**5.1.** Introduction

**5.2.** System Architecture

**5.3.** Environment Setup

**5.4.** Infrastructure Deployment

**5.5.** Frontend Deployment

**5.6.** Testing and Monitoring

**5.7.** Automated CI/CD Deployment

**5.8.** Cost Optimization and Security

**5.9.** Resource Cleanup

**5.10.** Conclusion

---

## Expected Outcome

After completing this workshop, you will have successfully deployed the
**Smart Notes API** on AWS with a complete set of components, including
the REST API, database, image storage, monitoring, fundamental security,
and an automated deployment pipeline.

The architecture follows the principles of the **AWS Well-Architected
Framework**, with a particular focus on four key pillars:

- **Cost Optimization** (no cost while idle, automatic cleanup using TTL)
- **Security** (least-privilege IAM, private Amazon S3 bucket with Presigned URLs)
- **Reliability** (Point-in-Time Recovery for Amazon DynamoDB)
- **Operational Excellence** (automated testing and deployment through CI/CD)

At the same time, the system maintains high scalability and availability
by leveraging a fully Serverless architecture.