# Amazon CloudWatch Monitoring

> **Objective**
>
> Create Amazon CloudWatch dashboards, configure alarms, create log groups, and use Amazon EventBridge (formerly CloudWatch Events) for monitoring and automation.

---

# Learning Objectives

After completing this lab, you will be able to:

* Create Amazon CloudWatch dashboards.
* Monitor Amazon EC2 performance metrics.
* Configure CloudWatch alarms and Amazon SNS notifications.
* Create and manage CloudWatch Log Groups.
* Query logs using CloudWatch Logs Insights.
* Install and configure the Amazon CloudWatch Agent.
* Apply CloudWatch monitoring best practices.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* Administrative access to the AWS Management Console.
* A running Amazon EC2 instance (`devops-web-01`).
* An IAM role with CloudWatch permissions (for example, `EC2-DevOps-Role`).
* SSH access to the EC2 instance.

> **Important**
>
> Memory and disk utilization metrics are **not collected by default**. Install the Amazon CloudWatch Agent to collect operating system–level metrics.

---

# Task 1 — Create a Dashboard & Alarms

## Step 1 — Open Amazon CloudWatch

1. Sign in to the AWS Management Console.
2. Search for **CloudWatch**.
3. Navigate to:

```text
Dashboards → Create dashboard
```

Configure:

| Setting        | Value              |
| -------------- | ------------------ |
| Dashboard Name | `DevOps-Dashboard` |

Click **Create**.

---

## Step 2 — Add an Amazon EC2 Widget

1. Click **Add widget**.
2. Select **Line graph**.
3. Configure the metric.

| Setting         | Value                |
| --------------- | -------------------- |
| Service         | EC2                  |
| Metric Category | Per-Instance Metrics |
| Metric          | CPUUtilization       |
| Instance        | `devops-web-01`      |
| Period          | 1 minute             |

Click **Create widget**.

---

## Step 3 — Add a Network Widget

1. Click **Add another widget**.
2. Select **Number**.

Configure the following metrics:

* `NetworkIn`
* `NetworkOut`

Stack the widgets on the dashboard and click **Save**.

---

## Step 4 — Create a CPU Alarm

Navigate to:

```text
CloudWatch → Alarms → Create alarm
```

Configure the alarm.

| Setting   | Value                |
| --------- | -------------------- |
| Metric    | EC2 → CPUUtilization |
| Resource  | Your EC2 instance    |
| Period    | 5 minutes            |
| Statistic | Average              |
| Condition | Greater than 80      |

### Notification

Create an Amazon SNS topic.

| Setting   | Value            |
| --------- | ---------------- |
| SNS Topic | `devops-alerts`  |
| Email     | `your@email.com` |

Configure:

| Setting    | Value            |
| ---------- | ---------------- |
| Alarm Name | `High-CPU-Alarm` |

Click **Create alarm**.

---

## Step 5 — Confirm the Amazon SNS Subscription

1. Check your email inbox.
2. Open the AWS SNS confirmation email.
3. Click **Confirm subscription**.
4. (Optional) Generate high CPU utilization (for example, by using a stress-testing tool) to verify that the alarm is triggered.

---

# Task 2 — CloudWatch Logs & Logs Insights

## Step 1 — Install the Amazon CloudWatch Agent

1. Connect to the EC2 instance using SSH.
2. Run the following commands to install and configure the CloudWatch Agent.

### Install the CloudWatch Agent

```bash
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb

sudo dpkg -i amazon-cloudwatch-agent.deb
```

### Run the Configuration Wizard

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

### Start and Enable the Agent

```bash
sudo systemctl start amazon-cloudwatch-agent

sudo systemctl enable amazon-cloudwatch-agent
```

---

## Step 2 — Create a CloudWatch Log Group

Navigate to:

```text
CloudWatch → Logs → Log groups → Create log group
```

Configure:

| Setting          | Value               |
| ---------------- | ------------------- |
| Log Group Name   | `/devops/ec2-nginx` |
| Retention Period | 30 days             |

Click **Create**.

---

## Step 3 — Query Logs Using CloudWatch Logs Insights

Navigate to:

```text
CloudWatch → Logs → Insights
```

Select the log group:

```text
/devops/ec2-nginx
```

Run the following query.

```sql
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
```

Visualize the query results.

> **Tip**
>
> CloudWatch Logs Insights enables interactive querying of log data without exporting logs to another analytics platform.

---

# Best Practice Tips

> **Tip**
>
> Follow these CloudWatch monitoring recommendations for production environments.

* Configure alarms for all critical metrics, including:

  * CPU utilization
  * Memory utilization (via CloudWatch Agent)
  * Disk utilization
  * NetworkIn
  * NetworkOut
* Use **Composite Alarms** to reduce alert fatigue by combining multiple alarm conditions.
* Configure appropriate retention periods for CloudWatch Log Groups to control storage costs.
* Use **CloudWatch Contributor Insights** to identify the top contributors to HTTP errors and other high-volume events.
* Use **Metric Math** to calculate derived metrics, such as:

```text
Error Rate = (Errors / Requests) × 100
```

* Enable **Container Insights** for Amazon ECS and Amazon EKS to automatically collect cluster-level metrics.

---

# Alternative Tool — Prometheus + Grafana (Open Source)

> **Note**
>
> Prometheus and Grafana provide a powerful open-source monitoring stack and can be integrated with AWS services.

Common deployment options include:

* Install **Prometheus** on an Amazon EC2 instance or within a Kubernetes cluster.
* Use **node_exporter** to collect operating system metrics.
* Connect **Grafana** to Prometheus as a data source to build advanced dashboards.
* Use **Amazon Managed Grafana** for a fully managed, serverless Grafana experience.
* Configure **Amazon CloudWatch** as a native Grafana data source.

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Created an Amazon CloudWatch dashboard.
* ✅ Added Amazon EC2 CPU and network monitoring widgets.
* ✅ Configured a CPU utilization alarm.
* ✅ Created an Amazon SNS notification topic.
* ✅ Confirmed an SNS email subscription.
* ✅ Installed and configured the Amazon CloudWatch Agent.
* ✅ Created a CloudWatch Log Group.
* ✅ Queried logs using CloudWatch Logs Insights.
* ✅ Reviewed CloudWatch monitoring best practices.
* ✅ Explored Prometheus and Grafana as alternative monitoring solutions.
