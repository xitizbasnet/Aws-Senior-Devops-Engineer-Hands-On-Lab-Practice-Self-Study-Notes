# Amazon S3 Storage & Lifecycle

> **Objective**
>
> Create Amazon S3 buckets, enable versioning, configure bucket policies, create lifecycle rules, and enable Cross-Region Replication (CRR).

---

# Learning Objectives

After completing this lab, you will be able to:

* Create an Amazon S3 bucket.
* Enable bucket versioning.
* Upload and manage object versions.
* Configure bucket policies.
* Create lifecycle management rules.
* Configure Cross-Region Replication (CRR).
* Enable Amazon S3 server access logging.
* Apply Amazon S3 security and storage best practices.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* Administrative access to the AWS Management Console.
* An IAM role named `EC2-DevOps-Role`.
* Basic knowledge of Amazon S3 storage classes and bucket permissions.

> **Important**
>
> Bucket names must be **globally unique** across all AWS accounts. Replace placeholders such as `ACCOUNT-ID` with your own AWS Account ID.

---

# Task 1 — Create & Configure an Amazon S3 Bucket

## Step 1 — Create the Bucket

1. Sign in to the AWS Management Console.
2. Search for **Amazon S3**.
3. Click **Create bucket**.

Configure the bucket using the following settings.

| Setting                | Value                                                      |
| ---------------------- | ---------------------------------------------------------- |
| Bucket Name            | `devops-lab-bucket-<ACCOUNT-ID>`                           |
| AWS Region             | `ap-south-1`                                               |
| Block Public Access    | Keep enabled (disable **only** for static website hosting) |
| Bucket Versioning      | Enable                                                     |
| Server-side Encryption | SSE-S3 (AES-256)                                           |

Click **Create bucket**.

> **Warning**
>
> Leave **Block all public access** enabled for private data. Disable it only when hosting a static website and after reviewing the associated security implications.

---

## Step 2 — Upload Objects

1. Open the newly created bucket.
2. Click **Upload**.
3. Click **Add files** and select any test files.
4. Expand **Properties**.
5. Configure:

| Setting       | Value    |
| ------------- | -------- |
| Storage Class | Standard |

6. Click **Upload**.

---

## Step 3 — Test Bucket Versioning

1. Modify one of the uploaded files.
2. Upload the same file again.
3. Select the uploaded object.
4. Open the **Versions** tab.
5. Enable **Show versions** to display all object versions.

> **Note**
>
> Amazon S3 stores every version of an object when versioning is enabled, allowing recovery from accidental overwrites or deletions.

---

## Step 4 — Configure a Bucket Policy

Navigate to:

```text
Bucket → Permissions → Bucket Policy → Edit
```

Paste the following JSON policy and replace **ACCOUNT-ID** with your AWS Account ID.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT-ID:role/EC2-DevOps-Role"
      },
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::devops-lab-bucket-ACCOUNT-ID",
        "arn:aws:s3:::devops-lab-bucket-ACCOUNT-ID/*"
      ]
    }
  ]
}
```

---

# Task 2 — Configure Lifecycle Policies & Replication

## Step 1 — Create a Lifecycle Rule

Navigate to:

```text
Bucket → Management → Lifecycle rules → Create lifecycle rule
```

Configure the following settings.

| Setting      | Value                                |
| ------------ | ------------------------------------ |
| Rule Name    | `archive-old-objects`                |
| Scope        | Apply to all objects in bucket       |
| Transition 1 | Move to S3 Standard-IA after 30 days |
| Transition 2 | Move to S3 Glacier after 90 days     |
| Expiration   | Delete objects after 365 days        |

Click **Create rule**.

---

## Step 2 — Configure Cross-Region Replication (CRR)

Navigate to:

```text
Bucket → Management → Replication rules → Create
```

Configure the following settings.

| Setting     | Value                              |
| ----------- | ---------------------------------- |
| Source      | Entire bucket                      |
| Destination | Create a new bucket in `us-east-1` |
| IAM Role    | Create new role (automatic)        |
| Rule Name   | `replicate-to-us-east`             |

Click **Save**.

> **Important**
>
> Versioning **must be enabled** on both the source and destination buckets before Cross-Region Replication can be configured.

---

## Step 3 — Enable Amazon S3 Server Access Logging

Navigate to:

```text
Bucket → Properties → Server access logging → Edit
```

Configure the following settings.

| Setting               | Value                                 |
| --------------------- | ------------------------------------- |
| Server Access Logging | Enable                                |
| Target Bucket         | Same bucket or a dedicated log bucket |
| Prefix                | `logs/`                               |

Click **Save**.

---

# Best Practice Tips

> **Tip**
>
> Follow these Amazon S3 best practices to improve security, durability, and cost optimization.

* Enable **Versioning** before configuring Cross-Region Replication.
* Use **S3 Intelligent-Tiering** for data with unpredictable access patterns.
* Keep **Block all public access** enabled by default.
* Use **pre-signed URLs** instead of making buckets publicly accessible.
* Enable **MFA Delete** on critical buckets to reduce the risk of accidental or malicious deletions.
* Use **Amazon S3 Object Lock (WORM)** for compliance workloads requiring immutable storage.
* Enable **Amazon S3 Inventory** for large buckets to audit object counts, storage classes, and metadata.

---

# Alternative Tool — AWS CLI for Amazon S3 Operations

> **Note**
>
> The AWS CLI provides an efficient way to automate common Amazon S3 operations.

### Upload a File

```bash
aws s3 cp file.txt s3://bucket-name/
```

---

### Synchronize a Directory

```bash
aws s3 sync ./local-dir s3://bucket-name/
```

---

### List Objects

```bash
aws s3 ls s3://bucket-name --recursive --human-readable
```

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Created an Amazon S3 bucket.
* ✅ Enabled bucket versioning.
* ✅ Uploaded and versioned objects.
* ✅ Configured a bucket policy.
* ✅ Created a lifecycle management rule.
* ✅ Configured Cross-Region Replication (CRR).
* ✅ Enabled Amazon S3 Server Access Logging.
* ✅ Reviewed Amazon S3 security and cost optimization best practices.
* ✅ Explored common Amazon S3 operations using the AWS CLI.
