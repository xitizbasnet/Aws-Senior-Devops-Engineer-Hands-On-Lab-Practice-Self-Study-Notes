# Amazon RDS & AWS Backup Services

> **Objective**
>
> Create an Amazon RDS MySQL database instance, configure Multi-AZ availability, create database snapshots, enable automated backups, and configure an AWS Backup vault and backup plan.

---

# Learning Objectives

After completing this lab, you will be able to:

* Create an Amazon RDS MySQL database instance.
* Configure DB subnet groups for private database deployment.
* Deploy RDS in a secure VPC architecture.
* Connect to RDS from an EC2 instance.
* Create and restore manual database snapshots.
* Configure AWS Backup vaults and backup plans.
* Apply RDS security and availability best practices.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* AWS Console access with required permissions.
* Existing VPC:

```text
devops-vpc
```

* Existing application security group:

```text
app-sg
```

* EC2 instance deployed in the same VPC for database connectivity testing.

> **Important**
>
> RDS should always be deployed inside private subnets for production environments.

---

# Task 1 — Create an Amazon RDS MySQL Instance

## Step 1 — Create DB Subnet Group

Search:

```text
RDS
```

Navigate to:

```text
RDS → Subnet groups → Create DB subnet group
```

Configure the subnet group.

| Setting | Value                                        |
| ------- | -------------------------------------------- |
| Name    | `devops-db-subnet-group`                     |
| VPC     | `devops-vpc`                                 |
| Subnets | Private subnets from both Availability Zones |

Click:

```text
Create
```

---

# Step 2 — Create RDS Database Instance

Navigate to:

```text
RDS → Databases → Create database
```

Configure the database engine.

| Setting         | Value               |
| --------------- | ------------------- |
| Engine          | MySQL               |
| Version         | 8.0.x               |
| Template        | Free tier (for lab) |
| DB Identifier   | `devops-mysql-db`   |
| Master Username | `admin`             |
| Master Password | Set secure password |
| Instance Class  | `db.t3.micro`       |

---

# Step 3 — Configure Storage & Networking

Configure storage settings.

| Setting             | Value                      |
| ------------------- | -------------------------- |
| Storage             | 20 GB gp2                  |
| Storage Autoscaling | Disabled (lab environment) |

Configure networking.

| Setting           | Value                    |
| ----------------- | ------------------------ |
| VPC               | `devops-vpc`             |
| Subnet Group      | `devops-db-subnet-group` |
| Public Access     | No (Private only)        |
| Security Group    | Create new: `rds-sg`     |
| Database Port     | 3306                     |
| Availability Zone | `ap-south-1a`            |

Security group rule:

```text
Allow MySQL TCP 3306 from app-sg
```

---

# Step 4 — Configure Additional Settings

Configure additional database options.

| Setting              | Value      |
| -------------------- | ---------- |
| Database Name        | `devopsdb` |
| Backup Retention     | 7 days     |
| Enhanced Monitoring  | Enabled    |
| Performance Insights | Enabled    |

Click:

```text
Create database
```

> **Note**
>
> RDS database creation may take approximately 5–10 minutes.

---

# Task 2 — Connect to RDS & Manage Snapshots

## Step 1 — Connect to RDS from EC2

Connect from an EC2 instance inside the same VPC.

Install MySQL client:

```bash
sudo apt-get install -y mysql-client
```

---

## Connect to RDS Database

Replace the endpoint with the endpoint shown in the RDS Console.

```bash
mysql -h devops-mysql-db.XXXX.ap-south-1.rds.amazonaws.com \
-u admin -p devopsdb
```

---

## Execute MySQL Commands

Inside MySQL:

```sql
SHOW DATABASES;
```

Create a users table:

```sql
CREATE TABLE users (
id INT AUTO_INCREMENT PRIMARY KEY,
name VARCHAR(100),
email VARCHAR(100) UNIQUE,
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Insert sample data:

```sql
INSERT INTO users (name, email)
VALUES ('Vinod', 'vinod@devops.com');
```

Retrieve records:

```sql
SELECT * FROM users;
```

Exit MySQL:

```sql
exit
```

---

# Step 2 — Create Manual Snapshot

Navigate to:

```text
RDS → Databases → devops-mysql-db
```

Select:

```text
Actions → Take snapshot
```

Configure:

| Setting       | Value                         |
| ------------- | ----------------------------- |
| Snapshot Name | `devops-mysql-manual-snap-01` |

Click:

```text
Take snapshot
```

---

# Step 3 — Restore Database from Snapshot

Navigate to:

```text
RDS → Snapshots
```

Select:

```text
devops-mysql-manual-snap-01
```

Choose:

```text
Actions → Restore snapshot
```

Configure:

| Setting        | Value                   |
| -------------- | ----------------------- |
| DB Identifier  | `devops-mysql-restored` |
| Instance Class | `db.t3.micro`           |

Click:

```text
Restore DB instance
```

---

# Task 3 — Configure AWS Backup Vault & Backup Plan

## Step 1 — Create Backup Vault

Search:

```text
AWS Backup
```

Navigate to:

```text
Backup vaults → Create backup vault
```

Configure:

| Setting    | Value                   |
| ---------- | ----------------------- |
| Vault Name | `devops-backup-vault`   |
| Encryption | Default AWS managed key |

Click:

```text
Create backup vault
```

---

# Step 2 — Create Backup Plan

Navigate to:

```text
AWS Backup → Backup plans → Create backup plan
```

Choose:

```text
Start with template
```

Select:

```text
Daily-35day-Retention
```

OR create a custom plan.

Custom configuration:

| Setting          | Value                 |
| ---------------- | --------------------- |
| Plan Name        | `devops-backup-plan`  |
| Backup Frequency | Daily                 |
| Retention        | 7 days                |
| Backup Vault     | `devops-backup-vault` |

---

# Step 3 — Assign Resources

Navigate to:

```text
Backup plan → Assign resources
```

Configure:

| Setting                  | Value            |
| ------------------------ | ---------------- |
| Resource Assignment Name | `rds-assignment` |
| Resource Selection       | By resource type |
| Resource Type            | RDS              |

Click:

```text
Assign resources
```

---

# Best Practice Tips

> **Tip**
>
> Follow these Amazon RDS security, availability, and backup recommendations for production environments.

* Always place RDS databases in **private subnets**.

Recommended architecture:

```text
Internet
   |
Application Load Balancer
   |
Application Servers
   |
Private RDS Database
```

* Enable **Multi-AZ deployment** for production RDS environments.

Benefits:

* Automatic failover.
* High availability.
* Improved resilience.

Expected failover time:

```text
Approximately 30–60 seconds
```

* Configure backup retention:

Recommended values:

| Workload             | Retention |
| -------------------- | --------- |
| Minimum              | 7 days    |
| Compliance workloads | 30 days   |

* Use **Amazon RDS Proxy** for:

  * AWS Lambda workloads.
  * Applications with high database connection requirements.
  * Connection pooling.

* Enable **IAM database authentication** instead of only username/password authentication for improved security.

* Use **Parameter Groups** to tune MySQL settings.

Examples:

```text
max_connections
innodb_buffer_pool_size
```

---

# Alternative Tool — Amazon Aurora

> **Note**
>
> Amazon Aurora is an AWS-managed relational database engine compatible with MySQL and PostgreSQL.

## Aurora Advantages

* Aurora MySQL can provide significantly higher performance than standard MySQL.
* Uses distributed cluster storage architecture.
* Supports automatic scaling with Aurora Serverless v2.

---

## Aurora Serverless v2

Features:

* Automatically adjusts database capacity based on demand.
* Charges based on Aurora Capacity Units (ACUs).

---

## When to Use Aurora

Recommended for:

* Production workloads.
* High-performance applications.
* Applications requiring automatic scaling.

Amazon RDS MySQL is suitable for:

* Development environments.
* Testing environments.
* Small to medium workloads.

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Created an RDS MySQL database instance.
* ✅ Configured a private DB subnet group.
* ✅ Deployed RDS securely inside a VPC.
* ✅ Connected to RDS from an EC2 instance.
* ✅ Created database tables and inserted test data.
* ✅ Created and restored manual snapshots.
* ✅ Created an AWS Backup vault.
* ✅ Configured an AWS Backup plan.
* ✅ Assigned RDS resources to automated backups.
* ✅ Reviewed RDS production best practices.
* ✅ Explored Amazon Aurora as an RDS alternative.
