# Amazon Route 53 & AWS Systems Manager

> **Objective**
>
> Configure Amazon Route 53 hosted zones, DNS records, health checks, and use AWS Systems Manager (SSM) for parameter management, secure access, automation, and operational management.

---

# Learning Objectives

After completing this lab, you will be able to:

* Create and configure Route 53 hosted zones.
* Configure DNS A and CNAME records.
* Create Route 53 health checks.
* Configure DNS failover routing.
* Store application configuration and secrets using Systems Manager Parameter Store.
* Access EC2 instances securely using Session Manager.
* Execute remote commands using SSM Run Command.
* Configure patch management using Systems Manager Patch Manager.
* Apply DNS and systems management best practices.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* AWS Console access.
* Existing VPC:

```text id="x9v2lm"
devops-vpc
```

* Existing EC2 instance:

```text id="z5q8na"
devops-web-01
```

* IAM role attached to EC2:

```text id="a3w7kp"
EC2-DevOps-Role
```

with required SSM permissions.

> **Important**
>
> AWS Systems Manager requires the SSM Agent to be installed and running on the managed instance.

---

# Task 1 — Amazon Route 53 DNS Configuration

## Step 1 — Create a Private Hosted Zone

Search:

```text id="h6k1qn"
Route 53
```

Navigate to:

```text id="t9v3ma"
Route 53 → Hosted zones → Create hosted zone
```

Configure the hosted zone.

| Setting     | Value                 |
| ----------- | --------------------- |
| Domain Name | `devops-lab.internal` |
| Type        | Private hosted zone   |
| VPC         | `devops-vpc`          |
| Region      | `ap-south-1`          |

Click:

```text id="4q1c8x"
Create hosted zone
```

---

# Step 2 — Create an A Record

Inside the hosted zone:

```text id="w7f9bd"
Create record
```

Configure:

| Setting          | Value                     |
| ---------------- | ------------------------- |
| Record Name      | `web`                     |
| Full Domain Name | `web.devops-lab.internal` |
| Record Type      | A                         |
| Value            | EC2 Private IP            |
| Example IP       | `10.0.1.10`               |
| TTL              | 300                       |

Click:

```text id="2g8yqp"
Create records
```

---

# Step 3 — Create a CNAME Record

Create another DNS record.

| Setting     | Value                                           |
| ----------- | ----------------------------------------------- |
| Record Name | `app`                                           |
| Record Type | CNAME                                           |
| Value       | `devops-alb-XXXXX.ap-south-1.elb.amazonaws.com` |
| TTL         | 60                                              |

> **Note**
>
> A lower TTL is recommended for load balancers because DNS changes need to propagate faster.

Click:

```text id="r4x8sk"
Create records
```

---

# Step 4 — Create Route 53 Health Check

> **Note**
>
> Health checks are normally used with public hosted zones.

Navigate to:

```text id="p6y2nd"
Route 53 → Health checks → Create health check
```

Configure:

| Setting          | Value              |
| ---------------- | ------------------ |
| Name             | `web-health-check` |
| Monitor Type     | Endpoint           |
| Protocol         | HTTP               |
| Domain           | EC2 Public IP      |
| Path             | `/index.html`      |
| Request Interval | 30 seconds         |

Click:

```text id="w3m6bv"
Create
```

---

# Step 5 — Configure Failover Routing Policy (Optional)

Create two A records with the same name.

## Primary Record

Configuration:

```text id="f1x5bn"
Primary → EC2 IP
```

Associate with:

```text id="n2k7za"
Health Check
```

---

## Secondary Record

Configuration:

```text id="c9r4hm"
Secondary → Backup IP
```

Traffic behavior:

```text id="v7k3pw"
Primary available:
    Route traffic to primary

Primary failure:
    Route traffic to secondary
```

---

# Task 2 — AWS Systems Manager (SSM)

## Step 1 — Create Parameter Store Entry

Search:

```text id="z8t1jq"
Systems Manager
```

Navigate to:

```text id="m4y6pc"
Parameter Store → Create parameter
```

Configure:

| Setting    | Value                     |
| ---------- | ------------------------- |
| Name       | `/devops/app/db-password` |
| Tier       | Standard                  |
| Type       | SecureString              |
| Encryption | AWS KMS                   |
| Value      | `MyS3cretP@ssw0rd!`       |

Click:

```text id="g8n3vx"
Create parameter
```

---

# Step 2 — Access Parameter Store from EC2

Ensure EC2 has:

```text id="h5j8qa"
EC2-DevOps-Role
```

with SSM permissions.

Connect to EC2 and execute:

```bash id="m9q7lt"
aws ssm get-parameter \
--name '/devops/app/db-password' \
--region ap-south-1
```

---

## Retrieve SecureString Parameter

To decrypt the stored value:

```bash id="c8z2yn"
aws ssm get-parameter \
--name '/devops/app/db-password' \
--with-decryption \
--region ap-south-1
```

---

# Step 3 — Use Session Manager as Secure Shell

Navigate to:

```text id="r9w4pk"
Systems Manager → Session Manager → Start session
```

Select:

```text id="s2x7md"
devops-web-01
```

Click:

```text id="q5z8cw"
Start session
```

You will receive:

```text id="j3v6sa"
Browser-based terminal access
```

No SSH key is required.

Test access:

```bash id="b8p3hf"
sudo apt-get update
```

---

# Step 4 — Execute Commands Using Run Command

Navigate to:

```text id="k1m7xz"
Systems Manager → Run Command → Run a command
```

Configure:

| Setting  | Value                          |
| -------- | ------------------------------ |
| Document | `AWS-RunShellScript`           |
| Target   | `devops-web-01`                |
| Command  | `sudo systemctl restart nginx` |

Click:

```text id="d7q2vb"
Run
```

Review:

```text id="n4x9qa"
Output tab
```

for execution results.

---

# Step 5 — Configure Patch Manager

Navigate to:

```text id="s8w5km"
Systems Manager → Patch Manager
```

Options:

## Immediate Patching

Select:

```text id="p2r6vf"
Patch now
```

---

## Scheduled Patching

Create:

* Patch baseline.
* Maintenance Window.

Use scheduled patching for production environments.

---

# Example — Use Parameter Store in Shell Script

Retrieve a database password securely.

```bash id="e7v2kp"
DB_PASS=$(aws ssm get-parameter \
--name '/devops/app/db-password' \
--with-decryption \
--query 'Parameter.Value' \
--output text \
--region ap-south-1)

echo 'DB_PASSWORD='$DB_PASS >> /etc/app.env
```

---

# Best Practice Tips

> **Tip**
>
> Follow these AWS DNS and systems management recommendations for secure and reliable operations.

* Use **SecureString** type in Parameter Store for secrets.

Benefits:

* Encrypts values using AWS KMS.
* Protects sensitive configuration data.

---

* Use **SSM Session Manager** instead of bastion hosts.

Benefits:

* Removes the requirement to expose SSH port 22.
* Provides centralized access control.
* Logs administrative sessions.

---

* Use **SSM Run Command** for fleet-wide automation.

Common tasks:

* Operating system patching.
* Service restart.
* Script execution.
* Configuration updates.

---

* Configure Route 53:

  * Health checks.
  * Failover routing.

for highly available DNS architectures.

---

* Use Route 53 Latency-Based Routing to automatically direct users to the closest AWS Region.

---

* Store application configuration in Parameter Store instead of hardcoding values.

Supports:

```text id="m5q9zw"
12-factor application principles
```

---

# Alternative Tool — HashiCorp Vault (Secrets Management)

> **Note**
>
> HashiCorp Vault is an alternative secrets management platform used for storing and controlling access to sensitive information.

## Key Features

* Open-source secrets management engine.

* Stores:

  * Credentials.
  * API keys.
  * Certificates.

* Can be deployed:

  * Self-hosted on EC2.
  * Self-hosted on Kubernetes/EKS.
  * Using HCP Vault managed service.

---

## Kubernetes Integration

Vault supports:

```text id="x4w9mj"
Vault Agent Injector
```

for:

* Pod-level secret injection.
* Dynamic secrets.
* Kubernetes workload security.

---

## When to Use Vault

Recommended for:

* Multi-cloud environments.
* Organizations requiring centralized secrets management.
* Environments where AWS Systems Manager Parameter Store is insufficient.

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Created a Route 53 private hosted zone.
* ✅ Configured DNS A records.
* ✅ Configured DNS CNAME records.
* ✅ Created Route 53 health checks.
* ✅ Configured optional failover routing.
* ✅ Created secure parameters using Systems Manager Parameter Store.
* ✅ Retrieved encrypted parameters from EC2.
* ✅ Connected to EC2 using Session Manager.
* ✅ Executed remote commands using SSM Run Command.
* ✅ Reviewed Patch Manager capabilities.
* ✅ Compared AWS Systems Manager Parameter Store with HashiCorp Vault.
