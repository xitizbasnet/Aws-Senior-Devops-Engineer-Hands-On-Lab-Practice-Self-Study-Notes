# AWS CI CD Pipeline

> **Objective**
>
> Set up a complete Continuous Integration and Continuous Delivery (CI/CD) pipeline using AWS CodeCommit for source control, AWS CodeBuild for build and testing, and AWS CodePipeline for orchestration.

---

# Learning Objectives

After completing this lab, you will be able to:

* Create an AWS CodeCommit repository.
* Configure Git credentials for AWS CodeCommit.
* Push application code to a repository.
* Create an AWS CodeBuild project.
* Build and test applications using `buildspec.yml`.
* Create an AWS CodePipeline pipeline.
* Automate application deployment.
* Apply AWS CI/CD best practices.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* Administrative access to AWS.
* Git installed on your local machine.
* Node.js application source code.
* An Amazon S3 bucket (`devops-lab-bucket-ACCOUNT-ID`) for deployment artifacts.
* IAM permissions to manage CodeCommit, CodeBuild, CodePipeline, Amazon S3, and Amazon SNS.

> **Important**
>
> Replace `ACCOUNT-ID` with your AWS Account ID throughout this lab.

---

# Task 1 — Create an AWS CodeCommit Repository

## Step 1 — Create the Repository

1. Sign in to the AWS Management Console.
2. Search for **CodeCommit**.
3. Navigate to:

```text
Repositories → Create repository
```

Configure the repository.

| Setting         | Value             |
| --------------- | ----------------- |
| Repository Name | `devops-app-repo` |
| Description     | `DevOps Lab App`  |

Click **Create**.

---

## Step 2 — Configure Git Credentials

Navigate to:

```text
IAM → Users → devops-user01 → Security credentials
```

Under **HTTPS Git credentials for AWS CodeCommit**:

1. Click **Generate credentials**.
2. Download the generated credentials CSV file.

> **Note**
>
> Store the Git credentials securely. They are required when cloning and pushing code to AWS CodeCommit.

---

## Step 3 — Clone the Repository & Push Code

Clone the repository.

```bash
git clone https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/devops-app-repo
```

Navigate to the project directory.

```bash
cd devops-app-repo
```

Create:

* `index.html`
* `buildspec.yml`

Commit and push the source code.

```bash
git add .
git commit -m "Initial commit"
git push origin main
```

---

### Create the `buildspec.yml` File

Create a file named `buildspec.yml` in the repository root.

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - echo Installing dependencies
      - npm install

  pre_build:
    commands:
      - echo Running tests
      - npm test

  build:
    commands:
      - echo Building application
      - npm run build

  post_build:
    commands:
      - echo Build completed
      - aws s3 sync ./build s3://devops-lab-bucket-ACCOUNT-ID/

artifacts:
  files:
    - '**/*'
  base-directory: build

cache:
  paths:
    - node_modules/**/*
```

---

# Task 2 — Create an AWS CodeBuild Project

## Step 1 — Create the Build Project

Search for **CodeBuild** and click **Create build project**.

Configure the following settings.

| Setting         | Value                  |
| --------------- | ---------------------- |
| Project Name    | `devops-build-project` |
| Source Provider | AWS CodeCommit         |
| Repository      | `devops-app-repo`      |
| Branch          | `main`                 |

---

## Step 2 — Configure the Build Environment

Configure the environment.

| Setting           | Value                        |
| ----------------- | ---------------------------- |
| Environment Image | Managed image                |
| Operating System  | Ubuntu                       |
| Runtime           | Standard                     |
| Image             | `aws/codebuild/standard:7.0` |
| Service Role      | Create new service role      |

---

## Step 3 — Configure Buildspec & Artifacts

Configure the following settings.

| Setting             | Value                                   |
| ------------------- | --------------------------------------- |
| Build Specification | Use `buildspec.yml` from the repository |
| Artifacts           | Amazon S3                               |
| Destination Bucket  | `devops-lab-bucket`                     |

Click **Create build project**.

---

## Step 4 — Test the Build

1. Start a build manually.
2. Verify that the build completes successfully.
3. Review the **Build logs** for any errors or warnings.

---

# Task 3 — Create an AWS CodePipeline Pipeline

## Step 1 — Create the Pipeline

Search for **CodePipeline** and click **Create pipeline**.

Configure the pipeline.

| Setting        | Value                    |
| -------------- | ------------------------ |
| Pipeline Name  | `devops-app-pipeline`    |
| Service Role   | Create new service role  |
| Artifact Store | Default Amazon S3 bucket |

---

## Step 2 — Configure the Source Stage

Configure the source stage.

| Setting          | Value                                        |
| ---------------- | -------------------------------------------- |
| Source Provider  | AWS CodeCommit                               |
| Repository       | `devops-app-repo`                            |
| Branch           | `main`                                       |
| Change Detection | Amazon CloudWatch Events (automatic trigger) |

---

## Step 3 — Configure the Build Stage

Configure the build stage.

| Setting        | Value                  |
| -------------- | ---------------------- |
| Build Provider | AWS CodeBuild          |
| Project Name   | `devops-build-project` |

Click **Next**.

---

## Step 4 — Configure the Deploy Stage (Optional)

Select a deployment provider.

### Amazon S3

| Setting                    | Value               |
| -------------------------- | ------------------- |
| Deployment Provider        | Amazon S3           |
| Bucket                     | `devops-lab-bucket` |
| Extract File Before Deploy | Enabled             |

Alternatively, choose **AWS Elastic Beanstalk** as the deployment target.

---

## Step 5 — Review & Create the Pipeline

1. Review the pipeline configuration.
2. Click **Create pipeline**.

The pipeline automatically starts its first execution.

Monitor each stage.

```text
Source → Build → Deploy
```

---

# Best Practice Tips

> **Tip**
>
> Follow these AWS CI/CD best practices when designing production pipelines.

* Store the `buildspec.yml` file in the repository root to reduce coupling with the CodeBuild console.
* Enable build caching (Amazon S3 cache) to reduce dependency download times.
* Add a **Manual Approval** stage between **Staging** and **Production** deployments.
* Configure Amazon SNS notifications for pipeline failures.
* Store secrets as **CodeBuild environment variables** instead of hardcoding them in `buildspec.yml`.
* Configure concurrent build limits to control infrastructure costs during periods of high activity.

---

# Alternative Tool — GitHub Actions / Jenkins

> **Note**
>
> GitHub Actions and Jenkins are widely used alternatives for implementing CI/CD pipelines on AWS.

### GitHub Actions

* Define workflows using:

```text
.github/workflows/deploy.yml
```

* Store AWS credentials securely using **GitHub Secrets**.
* Trigger deployments automatically on `git push`.

### Jenkins

* Install the AWS CLI.
* Configure credentials using **Jenkins Credentials Manager**.
* Deploy applications using:

  * AWS CLI
  * Terraform

### Comparison

| Tool           | Typical Use Case                                                       |
| -------------- | ---------------------------------------------------------------------- |
| GitHub Actions | Best suited for GitHub-hosted repositories                             |
| Jenkins        | Commonly used for on-premises and highly customized CI/CD environments |

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Created an AWS CodeCommit repository.
* ✅ Configured Git credentials.
* ✅ Cloned and pushed application source code.
* ✅ Created an AWS CodeBuild project.
* ✅ Built and tested an application using `buildspec.yml`.
* ✅ Created an AWS CodePipeline pipeline.
* ✅ Configured source, build, and deployment stages.
* ✅ Executed an end-to-end CI/CD pipeline.
* ✅ Reviewed AWS CI/CD best practices.
* ✅ Explored GitHub Actions and Jenkins as alternative CI/CD solutions.
