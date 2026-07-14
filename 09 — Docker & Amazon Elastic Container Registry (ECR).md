# Docker & Amazon Elastic Container Registry (ECR)

> **Objective**
>
> Build Docker images, push container images to Amazon Elastic Container Registry (ECR), and manage container image lifecycle policies.

---

# Learning Objectives

After completing this lab, you will be able to:

* Install and configure Docker on Ubuntu.
* Create and build Docker images.
* Run and test containers locally.
* Create an Amazon ECR repository.
* Authenticate Docker with Amazon ECR.
* Push container images to ECR.
* Configure ECR lifecycle policies.
* Apply Docker and container registry best practices.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* Administrative access to AWS.
* An Ubuntu Amazon EC2 instance.
* AWS CLI installed and configured.
* IAM permissions for:

  * Amazon ECR
  * Amazon EC2
  * Docker operations

> **Important**
>
> Replace placeholders such as `ACCOUNT-ID` and `REGION` with your AWS Account ID and AWS Region.

---

# Task 1 — Install Docker & Build an Image

## Step 1 — Install Docker on Ubuntu EC2

Update the package repository.

```bash id="1sfwpk"
sudo apt-get update
```

Install Docker.

```bash id="m8t6kk"
sudo apt-get install -y docker.io
```

Start and enable the Docker service.

```bash id="j8i7xn"
sudo systemctl start docker

sudo systemctl enable docker
```

Add the current user to the Docker group.

```bash id="8iqx9j"
sudo usermod -aG docker ubuntu
```

Apply the group change.

```bash id="g7cgqv"
newgrp docker
```

---

## Step 2 — Create a Docker Application

Create a project directory.

```bash id="t1v5mz"
mkdir ~/dockerapp
cd ~/dockerapp
```

Create the Dockerfile.

```dockerfile id="1lqv6k"
FROM nginx:alpine

LABEL maintainer='devops-vinod'

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=3s \
CMD wget -q --spider http://localhost/ || exit 1
```

Create the application HTML file.

```bash id="n3r4ml"
echo 'DevOps Lab - Containerized App' > index.html
```

---

## Step 3 — Build the Docker Image

Build the Docker image.

```bash id="wq4d1r"
docker build -t devops-nginx:v1 .
```

---

## Step 4 — Test the Container Locally

Run the container.

```bash id="l3kqv7"
docker run -d -p 8080:80 --name test-nginx devops-nginx:v1
```

Test the application.

```bash id="4j74eg"
curl http://localhost:8080
```

Expected result:

```text
DevOps Lab - Containerized App
```

Check running containers.

```bash id="l1iqxp"
docker ps
```

Stop and remove the test container.

```bash id="6n5h6n"
docker stop test-nginx

docker rm test-nginx
```

---

# Task 2 — Create an Amazon ECR Repository & Push Image

## Step 1 — Create an ECR Repository

1. Search for **ECR** in the AWS Console.
2. Click **Create repository**.

Configure the repository.

| Setting          | Value          |
| ---------------- | -------------- |
| Visibility       | Private        |
| Repository Name  | `devops-nginx` |
| Tag Immutability | Enabled        |
| Scan on Push     | Enabled        |

Click **Create repository**.

> **Note**
>
> Enabling tag immutability prevents overwriting existing image tags.

---

## Step 2 — Authenticate Docker with Amazon ECR

Copy the repository URI from the ECR console.

Authenticate Docker with ECR.

```bash id="4c9xgq"
aws ecr get-login-password --region ap-south-1 | \
docker login --username AWS --password-stdin \
ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com
```

---

## Step 3 — Tag & Push the Docker Image

Tag the local Docker image.

```bash id="8u3n4a"
docker tag devops-nginx:v1 \
ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-nginx:v1
```

Push the image to Amazon ECR.

```bash id="9x2zqv"
docker push ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-nginx:v1
```

---

## Step 4 — Verify the Image in ECR

1. Open the:

```text
devops-nginx
```

repository in Amazon ECR.

2. Select the uploaded image.
3. Review:

   * Image details
   * Security scan findings
   * Image URI

The image URI can later be used with:

* Amazon ECS
* Amazon EKS

Verify using AWS CLI:

```bash id="kzq6b7"
aws ecr describe-images \
--repository-name devops-nginx \
--region ap-south-1
```

---

# Task 3 — Configure an ECR Lifecycle Policy

## Step 1 — Create Lifecycle Policy

Navigate to:

```text id="7a6s8k"
ECR → devops-nginx repository → Lifecycle policy → Create rule
```

Configure:

| Setting       | Value                 |
| ------------- | --------------------- |
| Rule Priority | `1`                   |
| Image Status  | Untagged              |
| Count Type    | Image count more than |
| Count Number  | `5`                   |
| Action        | Expire                |

Click **Save**.

---

## Equivalent Lifecycle Policy JSON

```json id="3b4x7n"
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Remove untagged images older than 5",
      "selection": {
        "tagStatus": "untagged",
        "countType": "imageCountMoreThan",
        "countNumber": 5
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```

---

# Best Practice Tips

> **Tip**
>
> Follow these Docker and Amazon ECR best practices for secure and efficient container management.

* Use **multi-stage Docker builds** to significantly reduce final image size.
* Run containers as a **non-root user**.

Example:

```dockerfile
USER 1001
```

* Use a `.dockerignore` file to exclude:

  * `node_modules`
  * `.git`
  * Sensitive files
* Enable **ECR image scanning on push** to detect vulnerabilities using Amazon Inspector.
* Use semantic versioning for image tags:

Example:

```text
v1.0.0
```

Avoid using:

```text
latest
```

in production environments.

* Use ECR pull-through cache to reduce costs when repeatedly downloading public container images.

---

# Alternative Tool — Docker Hub / GitHub Container Registry

> **Note**
>
> Public container registries provide alternatives to Amazon ECR for storing and distributing Docker images.

## Docker Hub

Push images using:

```bash id="wq6z7t"
docker push yourusername/devops-nginx:v1
```

---

## GitHub Container Registry (GHCR)

* Use GitHub Actions to automatically:

  * Build Docker images
  * Push images to:

```text
ghcr.io
```

---

## Production Recommendation

For AWS production environments:

* Prefer **Amazon ECR** because it provides:

  * AWS IAM authentication
  * Native AWS service integration
  * Security scanning support
  * Integration with ECS and EKS

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Installed Docker on Ubuntu EC2.
* ✅ Created a Dockerfile.
* ✅ Built a Docker image.
* ✅ Tested a container locally.
* ✅ Created an Amazon ECR private repository.
* ✅ Authenticated Docker with Amazon ECR.
* ✅ Tagged and pushed an image to ECR.
* ✅ Verified images using the ECR console and AWS CLI.
* ✅ Created an ECR lifecycle policy.
* ✅ Reviewed Docker and container security best practices.
* ✅ Explored Docker Hub and GitHub Container Registry alternatives.
