---
title : "Testing and Monitoring"
date : 2026-07-21
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

## Introduction

A Serverless system has no servers to SSH into for viewing logs directly, so testing and observability must be designed from the beginning. This chapter covers testing at three levels: unit testing before deployment, testing the live API after deployment, and monitoring with Amazon CloudWatch and AWS X-Ray during operation.

---

## Unit Testing (Before Deployment)

The project uses **Jest** together with **aws-sdk-client-mock** to test all three layers (Controller/Service/Repository) **without calling real AWS services**—making the tests fast, free, and suitable for CI/CD.

```bash
npm test
```

One example of a tested business rule is that the service must retry up to three times when an Amazon S3 upload encounters a temporary error, and correctly delete or retain images based on the soft-delete status. Both behaviors are verified using mocks.

![Unit Test Result](/images/5-Workshop/5.6/jest-test-result.png)

---

## Testing the Live API with curl / Postman

```bash
BASE=https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev

# Create a note
curl -X POST $BASE/notes -H "Content-Type: application/json" \
  -d '{"title":"Test","content":"Hello workshop"}'

# Get active notes
curl $BASE/notes

# Upload an image for a note
curl -X POST $BASE/notes/<id>/image -F "image=@photo.png"

# Delete a note (move it to the recycle bin)
curl -X DELETE $BASE/notes/<id>

# View the recycle bin
curl $BASE/notes/trash

# Restore a note
curl -X POST $BASE/notes/<id>/restore
```

All responses follow a consistent format: `{"success": true/false, "data"/"message": ...}`. A Postman Collection is available in `postman/Smart Notes API.postman_collection.json` for quickly testing all endpoints without manually typing each `curl` command.

---

## Viewing Logs with Amazon CloudWatch Logs

Every Lambda invocation automatically writes logs. Logging permissions are granted by AWS SAM through the default execution role, so no additional configuration is required.

**Using the Console:** CloudWatch → Log groups → `/aws/lambda/smart-notes-api-dev` → Select the latest log stream.

**Using the CLI (tail logs in real time while testing):**

```bash
sam logs -n SmartNotesFunction --stack-name smart-notes-api --tail
```

**Filter error logs with CloudWatch Logs Insights:**

```
fields @timestamp, @message
| filter @message like /ERROR|error/
| sort @timestamp desc
| limit 20
```

![CloudWatch Logs Insights](/images/5-Workshop/5.6/cloudwatch-logs-insights.png)

---

## Tracing Requests with AWS X-Ray

Since `Tracing: Active` was enabled in `template.yaml` in Chapter 5.4, every request is traced through each stage: API Gateway → Lambda → DynamoDB/S3, including the execution time of each segment.

Open the **X-Ray Console → Service Map** to visualize the entire architecture. Any service experiencing errors will be highlighted in red. Open **Traces** to inspect individual requests and identify the slowest ones (for example, due to Amazon S3 upload retries).

![AWS X-Ray Service Map](/images/5-Workshop/5.6/xray-service-map.png)

---

## Creating a Basic Alarm

Proactively monitor the system instead of waiting for users to report problems.

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name smart-notes-api-error-rate \
  --namespace AWS/Lambda \
  --metric-name Errors \
  --dimensions Name=FunctionName,Value=smart-notes-api-dev \
  --statistic Sum \
  --period 300 \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --region ap-southeast-1
```

This alarm is triggered if more than five Lambda errors occur within five minutes. You can also attach an Amazon SNS topic to receive email or SMS notifications when the alarm enters the `ALARM` state—a natural extension beyond the scope of this workshop.

---

## Metrics to Monitor Regularly

| Metric | Description |
|---|---|
| `Lambda Duration` | Detects cold starts or slow DynamoDB queries |
| `Lambda Errors` / `Throttles` | Detects application errors or concurrency limit issues |
| `DynamoDB ConsumedRead/WriteCapacityUnits` | Monitors actual operational costs |
| `API Gateway 4XX/5XX Count` | Distinguishes client-side errors from backend errors |

---

## Result

After completing this chapter, you will have all the tools needed to determine whether your system is operating correctly—from local unit testing and live API testing to centralized logging and distributed tracing on AWS. The next chapter focuses on optimizing cost and security.