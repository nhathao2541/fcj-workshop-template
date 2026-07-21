---
title : "Introduction"
date : 2026-07-21
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---
Smart Notes API is a personal note management system built entirely on
AWS Cloud using a **Serverless** architecture and **Clean
Architecture**, eliminating the need to manage or operate any servers.

The system provides the following features:

+ Create, view, update, and delete notes (CRUD).
+ Upload and remove images attached to notes.
+ Securely access images through temporary signed URLs (Presigned URLs) without exposing the S3 bucket publicly.
+ Restore deleted notes within three days (Recycle Bin), after which they are permanently removed automatically.
+ Recover the entire database table to any point within the previous 35 days using **Point-in-Time Recovery (PITR)**, protecting against accidental updates or deletions.

The entire infrastructure is defined using **AWS SAM**
(**Infrastructure as Code**), making deployments repeatable and allowing
all resources to be removed with a single command without manual
configuration in the AWS Console. The build and deployment process is
further automated through **CI/CD**, ensuring every code change is tested
and deployed automatically without manually running deployment commands.

---

#### Workshop Objectives

In this workshop, you will build and deploy a complete **end-to-end**
Serverless REST API from scratch.

After completing this workshop, you will be able to:

+ Build a Serverless API using **Amazon API Gateway** and **AWS Lambda**.
+ Store note data in **Amazon DynamoDB**, using **Time-to-Live (TTL)** to automatically remove expired data.
+ Protect note data with **Amazon DynamoDB Point-in-Time Recovery (PITR)**, allowing recovery to any point within the previous 35 days.
+ Store and secure images in **Amazon S3** using a private bucket and Presigned URLs.
+ Manage the entire infrastructure using **AWS SAM / AWS CloudFormation** (Infrastructure as Code).
+ Apply the least-privilege principle using **AWS IAM**.
+ Monitor logs and distributed traces with **Amazon CloudWatch** and **AWS X-Ray**.
+ Test the API using **Jest**, **Postman**, and **curl**.
+ Automate build and deployment using **CI/CD** with **GitHub Actions**, eliminating manual deployment.
+ Optimize costs, apply basic security best practices, and properly clean up AWS resources.

---

#### System Architecture Overview

The Smart Notes API architecture consists of the following main layers:

+ **Frontend Layer**  
  A static web application (HTML/CSS/JavaScript) is served directly by
  **AWS Lambda** using Express static middleware. The frontend calls the
  API through the same domain, eliminating the need for Amazon S3 Static
  Website Hosting or Amazon CloudFront in this workshop.

+ **Application Layer**  
  All client requests are received by **Amazon API Gateway** (REST API)
  and routed to a single **AWS Lambda** function running Express
  (`serverless-http`), following a three-layer Clean Architecture:
  Controller → Service → Repository.

+ **Data Layer**  
  Note data is stored in **Amazon DynamoDB** using
  **PAY_PER_REQUEST** billing mode, with **Time-to-Live (TTL)** enabled
  for the recycle bin feature and **Point-in-Time Recovery (PITR)** for
  data protection. Attached images are stored in **Amazon S3** using a
  completely private bucket and accessed through Presigned URLs.

+ **Monitoring Layer**  
  Invocation logs and distributed request traces are collected by
  **Amazon CloudWatch** and **AWS X-Ray**, making it easier to monitor
  and debug a serverless application without direct server access.

+ **Security Layer**  
  **AWS IAM** grants Lambda only the minimum permissions required to
  access the project's DynamoDB table and S3 bucket, following the
  **least-privilege** principle rather than using administrator
  privileges.

+ **CI/CD Layer**  
  Every code change is automatically built, tested, and deployed to AWS
  through **GitHub Actions**, eliminating the need to run `sam deploy`
  manually.

---

#### Deployment Architecture

During this workshop, you will deploy the following components:

+ **Application Layer**
  + Amazon API Gateway
  + AWS Lambda

+ **Data Storage**
  + Amazon DynamoDB (with TTL and Point-in-Time Recovery)
  + Amazon S3

+ **Frontend**
  + Static web application served through Lambda (Express static)

+ **Monitoring & Security**
  + Amazon CloudWatch
  + AWS X-Ray
  + IAM Roles (Least Privilege)

+ **Infrastructure as Code**
  + AWS SAM
  + AWS CloudFormation

+ **CI/CD**
  + GitHub Actions (automatic testing and deployment)

This workshop is designed to help learners become familiar with
**Serverless Architecture**, **Clean Architecture**, and the best
practices recommended by the **AWS Well-Architected Framework**,
especially the four pillars of **Cost Optimization**, **Security**,
**Reliability** (through PITR), and **Operational Excellence** (through
CI/CD).

---

![Smart Notes API Architecture](/images/5-Workshop/5.1/smart-notes-architecture.png)