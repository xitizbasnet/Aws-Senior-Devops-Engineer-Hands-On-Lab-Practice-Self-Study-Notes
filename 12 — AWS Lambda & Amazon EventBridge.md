# AWS Lambda & Amazon EventBridge

> **Objective**
>
> Create AWS Lambda functions, configure event triggers, and set up Amazon EventBridge rules for scheduled and event-driven automation.

---

# Learning Objectives

After completing this lab, you will be able to:

* Create AWS Lambda functions.
* Configure Lambda execution roles.
* Write and test Lambda code.
* Trigger Lambda functions using Amazon S3 events.
* Configure Amazon EventBridge scheduled rules.
* Monitor Lambda executions using Amazon CloudWatch Logs.
* Apply serverless automation best practices.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* Administrative access to AWS.
* An Amazon S3 bucket:

```text
devops-lab-bucket
```

* Permissions for:

  * AWS Lambda
  * Amazon S3
  * Amazon EventBridge
  * Amazon CloudWatch

> **Important**
>
> Ensure your Lambda execution role has permission to write logs to CloudWatch.

---

# Task 1 — Create an AWS Lambda Function

## Step 1 — Create the Lambda Function

1. Search for:

```text
Lambda
```

2. Navigate to:

```text
Functions → Create function
```

Select:

```text
Author from scratch
```

Configure the function.

| Setting       | Value                                         |
| ------------- | --------------------------------------------- |
| Function Name | `devops-s3-alert`                             |
| Runtime       | Python 3.12                                   |
| Architecture  | x86_64                                        |
| Permissions   | Create new role with basic Lambda permissions |

Click:

```text
Create function
```

---

# Step 2 — Write Lambda Function Code

Navigate to:

```text
Code source → Edit inline
```

Replace the existing code with the following Python code.

```python
import json
import boto3
import os

from datetime import datetime


def lambda_handler(event, context):

    print(f'Event received: {json.dumps(event)}')

    # Example: Notify on S3 object creation
    if 'Records' in event:

        for record in event['Records']:

            bucket = record['s3']['bucket']['name']
            key = record['s3']['object']['key']
            size = record['s3']['object']['size']

            print(
                f'New object: s3://{bucket}/{key}, size: {size} bytes'
            )

    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': 'Lambda executed successfully',
            'timestamp': str(datetime.now())
        })
    }
```

---

# Step 3 — Test the Lambda Function

Create a test event.

1. Click:

```text
Test → Configure test event
```

2. Configure:

| Setting    | Value         |
| ---------- | ------------- |
| Event Name | `TestEvent`   |
| Template   | `hello-world` |

3. Click:

```text
Test
```

Verify:

```text
Execution result: Succeeded
```

Review output:

```text
CloudWatch Logs
```

to confirm Lambda execution details.

---

# Task 2 — Add an Amazon S3 Trigger to Lambda

## Step 1 — Configure S3 Trigger

Navigate to:

```text
Lambda → devops-s3-alert → Configuration → Triggers
```

Click:

```text
Add trigger → S3
```

Configure the trigger.

| Setting    | Value                                |
| ---------- | ------------------------------------ |
| Bucket     | `devops-lab-bucket`                  |
| Event Type | PUT (All object create events)       |
| Prefix     | `uploads/` (Optional filter by path) |

Click:

```text
Add
```

---

# Step 2 — Test the S3 Trigger

Upload a file to:

```text
S3 → devops-lab-bucket → uploads/
```

Monitor Lambda execution.

Navigate to:

```text
Lambda → Monitor → Logs → View logs in CloudWatch
```

Verify that the log output displays:

* Uploaded object name.
* Object size.

Example:

```text
New object: s3://devops-lab-bucket/uploads/file.txt, size: XXXX bytes
```

---

# Task 3 — Create an Amazon EventBridge Scheduled Rule

## Step 1 — Create EventBridge Rule

Search:

```text
EventBridge
```

Navigate to:

```text
Rules → Create rule
```

Configure the rule.

| Setting   | Value              |
| --------- | ------------------ |
| Rule Name | `daily-ec2-report` |
| Event Bus | `default`          |
| Rule Type | Schedule           |

Configure the schedule:

```text
cron(0 9 * * ? *)
```

This runs:

```text
Every day at 9:00 AM UTC
```

---

# Step 2 — Configure Target

Set the target.

| Setting     | Value             |
| ----------- | ----------------- |
| Target Type | Lambda function   |
| Function    | `devops-s3-alert` |

Click:

```text
Create rule
```

---

# Step 3 — Verify EventBridge Schedule

Navigate to:

```text
EventBridge → Rules → daily-ec2-report
```

Click:

```text
Target
```

Verify:

* Target Lambda function.
* Next invocation time.

---

# Best Practice Tips

> **Tip**
>
> Follow these AWS Lambda best practices for secure, reliable, and cost-efficient serverless applications.

* Set an appropriate Lambda timeout.

Default:

```text
3 seconds
```

Increase the timeout for complex workloads.

* Use **Lambda Layers** for shared dependencies such as:

  * `boto3`
  * `numpy`

across multiple Lambda functions.

* Monitor Lambda performance using CloudWatch metrics:

  * Duration
  * Errors
  * Throttles
  * ConcurrentExecutions

* Use **Lambda Power Tuning** (open source) to identify the optimal memory configuration for cost and performance.

* Avoid large deployment packages.

For packages larger than:

```text
50 MB
```

use Amazon S3 for deployment packages.

* Configure **Dead Letter Queues (DLQ)** using:

  * Amazon SQS
  * Amazon SNS

for failed Lambda invocations.

---

# Alternative Tool — AWS Step Functions

> **Note**
>
> AWS Step Functions is a serverless workflow orchestration service used for building complex multi-step processes.

## Key Features

* Orchestrate multiple Lambda functions using visual workflows.
* Provide built-in:

  * Error handling
  * Retry logic
  * State management

---

## Workflow Types

### Step Functions Standard

Recommended for:

* Long-running workflows.
* Business processes.
* Human approval workflows.

### Step Functions Express

Recommended for:

* High-frequency workflows.
* Short-duration tasks.
* Event-driven processing.

---

## Common Use Cases

AWS Step Functions are ideal for:

* ETL pipelines.
* Order processing systems.
* Multi-step approval workflows.
* Serverless application orchestration.

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Created an AWS Lambda function using Python 3.12.
* ✅ Configured Lambda permissions and execution roles.
* ✅ Tested Lambda execution using a test event.
* ✅ Created an Amazon S3 event trigger.
* ✅ Verified S3-triggered Lambda execution through CloudWatch Logs.
* ✅ Created an Amazon EventBridge scheduled rule.
* ✅ Connected EventBridge to a Lambda target.
* ✅ Reviewed Lambda operational best practices.
* ✅ Explored AWS Step Functions for complex serverless workflows.
