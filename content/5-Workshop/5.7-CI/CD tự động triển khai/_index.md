---
title : "Automatic CI/CD Deployment"
date : 2026-07-21
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

In Chapters 5.4 through 5.6, every infrastructure or code change was
deployed manually by running `sam build && sam deploy` on a local
machine. This approach has two main drawbacks: it is easy to forget to
run tests before deployment, and deployment depends on a specific
machine and a specific person with AWS CLI credentials.

In this chapter, we will configure **CI/CD (Continuous Integration /
Continuous Deployment)** using **GitHub Actions**. Whenever code is
pushed to the `main` branch, the pipeline will automatically run unit
tests, build the application, and deploy both the infrastructure and
the latest code—without any manual intervention.

AWS does not provide a single dedicated CI/CD service. CI/CD is a
workflow that can be implemented using AWS CodePipeline and CodeBuild,
or—as in this project—with GitHub Actions, which is simpler, incurs no
additional AWS cost, and is well suited for a personal project.

---

## Preparation: Grant AWS Access to GitHub Actions

GitHub Actions requires an IAM User (or an IAM Role through OIDC) with
sufficient permissions to execute `sam deploy`.

For a small project, the simplest approach is to create a dedicated IAM
User for CI/CD with permissions for CloudFormation, Lambda, API Gateway,
DynamoDB, S3, Cognito, and IAM PassRole. Avoid using
`AdministratorAccess`.

After obtaining the Access Key ID and Secret Access Key, open your
GitHub repository and navigate to:

**Settings → Secrets and variables → Actions**

Create the following repository secrets:

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
GOOGLE_CLIENT_ID
GOOGLE_CLIENT_SECRET
```

The last two secrets correspond to the `GoogleClientId` and
`GoogleClientSecret` parameters used during the Cognito configuration in
Chapter 5.4. Because these parameters are marked as `NoEcho`, AWS SAM
does not save them in `samconfig.toml`, so the pipeline must provide
them during every deployment.

![Add GitHub Secrets](/images/5-Workshop/5.7/github-secrets.png)

---

## Create the GitHub Actions Workflow

Create the file:

```text
.github/workflows/deploy.yml
```

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

Key differences compared to a manual deployment:

- `--no-confirm-changeset` and `--no-fail-on-empty-changeset` remove
  interactive prompts because the pipeline runs unattended.
- `npm test` executes **before** deployment. If any unit test fails, the
  pipeline stops immediately and the application is **not** deployed.
- `--parameter-overrides` passes the Cognito/Google parameters during
  every deployment because `NoEcho` parameters are not stored in
  `samconfig.toml`.

---

## Verify the Pipeline

Push a small change—for example, updating an error message in
`errors.js`—to the `main` branch.

Open the **Actions** tab in your GitHub repository to monitor the
workflow:

**Checkout → Install → Test → Deploy**

![Successful GitHub Actions pipeline](/images/5-Workshop/5.7/github-actions-run.png)

If the **Run unit tests** step fails, the workflow stops immediately,
and the existing AWS infrastructure remains unchanged because the
deployment step is never executed.

---

## Result

After completing this chapter, every change pushed to the `main` branch
is automatically tested and deployed to AWS. There is no longer any need
to manually run `sam build && sam deploy`.

This completes the basic DevOps workflow for Smart Notes API before
moving on to the next chapter on cost optimization and security.