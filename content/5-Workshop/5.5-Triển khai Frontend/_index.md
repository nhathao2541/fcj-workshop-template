---
title : "Frontend Deployment"
date : 2026-07-21
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

## Introduction

In this chapter, we will deploy the static web interface for the Smart Notes API. Unlike many common architectures that use **Amazon S3 Static Website Hosting** together with **Amazon CloudFront**, this workshop serves the frontend **directly from the same AWS Lambda function** that runs the backend.

This is an intentional architectural decision that fits the scope of a lightweight personal API.

---

## Comparing Two Frontend Deployment Approaches

| Deployment Option | Advantages | Recommended Use Case |
|---|---|---|
| **Static files served by Lambda (Selected)** | No additional AWS resources, no CloudFront cost, same domain as the API so no CORS issues | Low traffic, personal projects, MVPs, or simple applications |
| Amazon S3 Static Website + CloudFront | Global CDN acceleration, caching, separation of frontend and backend | High traffic, custom frontend domain, users distributed across multiple regions |

The architecture can easily be upgraded in the future by moving the static website to Amazon S3 and CloudFront without modifying the backend APIs.

![Frontend Deployment Options](/images/5-Workshop/5.5/frontend-options.png)

---

## Add a Static File Route to Express

Since the project uses ES Modules (`"type": "module"` in `package.json`), the `__dirname` variable is not available as it is in CommonJS. It must be recreated using `import.meta.url`:

```js
import path from "path";
import { fileURLToPath } from "url";
import express from "express";

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

const app = express();

// Register BEFORE the /notes router so "/" serves the frontend
// instead of falling through to the 404 handler.
app.use(express.static(path.join(__dirname, "public")));

app.use("/notes", notesRouter);
```

Place the frontend file in `src/public/index.html`.

Because `CodeUri: ./` is specified in `template.yaml`, the entire project directory is packaged during deployment. Therefore, the frontend files are automatically deployed together with the Lambda function **without requiring any additional AWS resources**.

---

## Why the Root Route `/` Must Be Explicitly Declared

By default, Amazon API Gateway REST APIs only route requests that match the `{proxy+}` path.

Requests sent to the root path `/` **are not automatically forwarded** to the Lambda proxy if only `{proxy+}` is configured.

For this reason, the `template.yaml` file created in Chapter 5.4 defines two API events:

```yaml
Events:
  RootApi:
    Type: Api
    Properties:
      Path: /
      Method: ANY

  ProxyApi:
    Type: Api
    Properties:
      Path: /{proxy+}
      Method: ANY
```

Omitting the `RootApi` route is one of the most common reasons why the homepage returns **403** or **404** errors while all other API endpoints continue to function correctly.

---

## Build and Deploy Again

```bash
sam build && sam deploy
```

Verify the deployment:

```bash
curl -I https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev/
```

Expected result:

```text
HTTP/2 200
content-type: text/html
```

Open the URL in your web browser. The frontend should now be able to communicate directly with the backend APIs using the same domain.

![Smart Notes Frontend Running](/images/5-Workshop/5.5/frontend-running.png)

---

## Result

After completing this chapter, users can access the fully functional Smart Notes web application through a single API Gateway URL.

The application supports creating, editing, deleting, searching, and sorting notes, uploading images, and managing the recycle bin, all through the same endpoint.

In the next chapter, we will test the application and evaluate its overall functionality and performance.