# Terraform (Infrastructure as Code)

> **Objective**
>
> Install Terraform, write HashiCorp Configuration Language (HCL) configuration files, manage remote state using Amazon S3, and use modules for reusable infrastructure.

---

# Learning Objectives

After completing this lab, you will be able to:

* Install Terraform.
* Configure the AWS Provider.
* Write Terraform configuration using HCL.
* Execute the Terraform workflow (`init`, `plan`, `apply`, `destroy`).
* Inspect Terraform state.
* Configure a remote backend using Amazon S3 and Amazon DynamoDB.
* Apply Infrastructure as Code (IaC) best practices.
* Explore Pulumi as an alternative IaC solution.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* Administrative access to AWS.
* AWS CLI configured with appropriate credentials or an EC2 instance with an IAM role.
* Ubuntu (local machine or Amazon EC2).
* Internet connectivity.

> **Important**
>
> Replace placeholders such as `YOUR-ACCT-ID` with your own AWS Account ID.

---

# Task 1 — Set Up Terraform & Configure the AWS Provider

## Step 1 — Install Terraform

Connect to your Ubuntu system (Amazon EC2 or local machine).

Run the following commands.

### Install Terraform

```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common

wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install terraform
```

### Verify the Installation

```bash
terraform version
```

---

## Step 2 — Configure the AWS Provider

Create a project directory.

```bash
mkdir ~/tf-lab
cd ~/tf-lab
```

Create a file named:

```text
main.tf
```

Add the following configuration.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}

resource "aws_s3_bucket" "devops_bucket" {
  bucket = "tf-devops-lab-bucket-${var.account_id}"

  tags = {
    Environment = "dev"
    ManagedBy   = "Terraform"
  }
}

variable "account_id" {
  description = "AWS Account ID"
  type        = string
}

output "bucket_name" {
  value = aws_s3_bucket.devops_bucket.bucket
}
```

---

# Task 2 — Execute the Terraform Workflow

## Step 1 — Initialize Terraform

Navigate to the project directory.

```bash
cd ~/tf-lab
```

Initialize Terraform.

```bash
terraform init
```

> **Note**
>
> This command downloads the required AWS provider plugin and initializes the working directory.

---

## Step 2 — Validate & Plan

Validate the Terraform configuration.

```bash
terraform validate
```

Generate an execution plan.

```bash
terraform plan -var='account_id=YOUR-ACCT-ID'
```

> **Tip**
>
> The execution plan previews infrastructure changes before they are applied.

---

## Step 3 — Apply the Infrastructure

Deploy the resources.

```bash
terraform apply -var='account_id=YOUR-ACCT-ID'
```

When prompted, type:

```text
yes
```

After deployment, verify the output.

```text
bucket_name
```

---

## Step 4 — Inspect Terraform State

Display all managed resources.

```bash
terraform state list
```

Display detailed information about the Amazon S3 bucket.

```bash
terraform state show aws_s3_bucket.devops_bucket
```

---

## Step 5 — Destroy the Infrastructure

Remove all managed resources.

```bash
terraform destroy -var='account_id=YOUR-ACCT-ID'
```

When prompted, type:

```text
yes
```

All Terraform-managed resources will be deleted.

---

# Task 3 — Configure Remote State Using Amazon S3

## Step 1 — Configure the Backend

Add the following backend configuration to `main.tf`.

```hcl
terraform {
  backend "s3" {
    bucket         = "tf-state-bucket-YOUR-ACCT-ID"
    key            = "devops-lab/terraform.tfstate"
    region         = "ap-south-1"
    encrypt        = true
    dynamodb_table = "tf-state-lock"
  }
}
```

> **Important**
>
> The Amazon DynamoDB table is used to prevent concurrent updates to the Terraform state file.

---

## Step 2 — Create the DynamoDB Lock Table

Run the following command once.

```bash
aws dynamodb create-table \
--table-name tf-state-lock \
--attribute-definitions AttributeName=LockID,AttributeType=S \
--key-schema AttributeName=LockID,KeyType=HASH \
--billing-mode PAY_PER_REQUEST \
--region ap-south-1
```

---

## Step 3 — Reinitialize Terraform

After configuring the backend, reinitialize Terraform.

```bash
terraform init
```

Terraform migrates the local state to the configured Amazon S3 backend.

---

# Best Practice Tips

> **Tip**
>
> Follow these Terraform best practices when managing Infrastructure as Code.

* Always use a **remote backend** (Amazon S3 + DynamoDB locking) for team environments instead of storing `terraform.tfstate` locally.
* Use the following workflow to reduce deployment risk.

```bash
terraform plan -out=plan.tfplan
terraform apply plan.tfplan
```

* Create reusable **Terraform modules** for common infrastructure components such as:

  * Amazon VPC
  * Amazon EC2
  * Amazon RDS
* Automatically format code before committing changes.

```bash
terraform fmt
```

* Validate Terraform configurations before every commit.

```bash
terraform validate
```

* Store environment-specific variables in dedicated `.tfvars` files such as:

  * `dev.tfvars`
  * `prod.tfvars`
* Never hardcode AWS credentials inside Terraform configuration files. Use IAM roles, environment variables, or AWS credential profiles.

---

# Alternative Tool — Pulumi

> **Note**
>
> Pulumi is an Infrastructure as Code platform that allows infrastructure to be defined using general-purpose programming languages.

Key capabilities include:

* Write infrastructure using:

  * Python
  * Go
  * TypeScript
* Supports multiple cloud providers:

  * AWS
  * Microsoft Azure
  * Google Cloud Platform (GCP)
* Uses native cloud SDKs instead of HCL.
* Well suited for teams that prefer application programming languages over declarative configuration files.

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Installed Terraform.
* ✅ Configured the AWS Provider.
* ✅ Created Terraform infrastructure using HCL.
* ✅ Initialized a Terraform project.
* ✅ Validated and planned infrastructure changes.
* ✅ Deployed AWS resources using Terraform.
* ✅ Inspected Terraform state.
* ✅ Destroyed managed infrastructure.
* ✅ Configured a remote backend using Amazon S3 and Amazon DynamoDB.
* ✅ Reviewed Terraform Infrastructure as Code best practices.
* ✅ Explored Pulumi as an alternative Infrastructure as Code platform.
