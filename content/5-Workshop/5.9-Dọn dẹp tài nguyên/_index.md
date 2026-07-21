---
title : "Cleaning Up Resources"
date : 2026-07-21
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

## Introduction

This is a **required** step after completing the workshop to avoid unexpected AWS charges—especially Amazon S3 storage costs, which are charged based on the amount of stored data per day, regardless of whether the API is being used.

---

## Step 1 — Delete All Objects in the Image Bucket

AWS CloudFormation **cannot delete an Amazon S3 bucket if it still contains objects**. Therefore, this must be the first step.

```bash
aws s3 rm s3://smart-notes-storage-<account-id>-dev --recursive
```

If **Versioning** is enabled on the bucket, you must also delete all previous object versions from the **Versions** tab in the Amazon S3 Console, because the `rm --recursive` command does not remove deleted object versions (delete markers).

![Delete Objects from the S3 Bucket](/images/5-Workshop/5.8/s3-empty-bucket.png)

---

## Step 2 — Delete the Entire CloudFormation Stack

```bash
sam delete
```

Confirm **`y`** for both prompts:

```
Are you sure you want to delete the stack smart-notes-api in the region ap-southeast-1 ? [y/N]: y
Are you sure you want to delete the folder smart-notes-api in S3 which contains the artifacts? [y/N]: y
```

This command automatically deletes the AWS Lambda function, Amazon API Gateway, Amazon DynamoDB table, IAM role, and all other resources defined in `template.yaml`.

If you prefer using the AWS Console instead of the CLI:

**CloudFormation → Select the `smart-notes-api` stack → Delete**

Wait until the stack status changes to:

```
DELETE_COMPLETE
```

![CloudFormation Stack Delete](/images/5-Workshop/5.8/cloudformation-delete.png)

---

## Step 3 — Delete the AWS SAM Managed S3 Bucket

When `sam deploy --guided` is executed for the first time, AWS SAM automatically creates **another Amazon S3 bucket** to store deployment artifacts. This bucket is **not** part of the CloudFormation stack and is **not** deleted automatically in Step 2.

```bash
aws s3 rb s3://aws-sam-cli-managed-default-samclisourcebucket-xxxxxxxxxxxxx --force
```

---

## Step 4 — Delete the CloudWatch Log Group

CloudFormation does not automatically delete log groups that were created during execution.

```bash
aws logs delete-log-group --log-group-name /aws/lambda/smart-notes-api-dev --region ap-southeast-1
```

---

## Step 5 — Cleanup Checklist

Verify manually in the AWS Console (Region: **ap-southeast-1**) to ensure that no resources remain.

| Service | Verification |
|---|---|
| CloudFormation | The `smart-notes-api` stack no longer exists |
| AWS Lambda | The `smart-notes-api-dev` function no longer exists |
| Amazon API Gateway | The corresponding API has been removed |
| Amazon DynamoDB | The `Notes-dev` table no longer exists |
| Amazon S3 | Both buckets (the image bucket and the AWS SAM managed bucket) have been deleted |
| CloudWatch Logs | The Lambda log group no longer exists |
| IAM | The Lambda execution role has been deleted automatically with the stack—no manual action is required |

![Cleanup Checklist](/images/5-Workshop/5.8/cleanup-checklist.png)

---

## Troubleshooting Cleanup Failures

If `sam delete` or deleting the stack through the AWS Console results in a `DELETE_FAILED` status, the most common cause is that **Step 1 was not completed successfully** (the S3 bucket still contains objects, including previous versions if Versioning is enabled).

Return to Step 1, remove all remaining objects and versions, then try deleting the stack again. CloudFormation will continue deleting the remaining resources.

AWS X-Ray trace data expires automatically after 30 days and does not incur separate storage charges, so no manual cleanup is required.

---

## Result

After completing this chapter, all AWS resources created for the Smart Notes API have been removed, and no additional AWS charges will be incurred.

You have now completed the entire workshop—from designing the architecture, deploying the infrastructure and frontend, testing and monitoring, optimizing cost and security, to properly cleaning up resources—covering the complete lifecycle of a Serverless application on AWS.