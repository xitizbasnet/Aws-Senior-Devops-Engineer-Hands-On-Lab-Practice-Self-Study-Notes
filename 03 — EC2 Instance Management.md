# EC2 Instance Management

> **Objective**
>
> Launch Amazon EC2 instances, configure key pairs, attach Amazon EBS volumes, create Amazon Machine Images (AMIs), and set up an Auto Scaling Group (ASG).

---

# Learning Objectives

After completing this lab, you will be able to:

* Launch an Amazon EC2 instance.
* Create and manage EC2 key pairs.
* Configure networking and storage.
* Attach and mount an Amazon EBS volume.
* Create an Amazon Machine Image (AMI).
* Create a Launch Template.
* Configure an Auto Scaling Group (ASG).
* Apply Amazon EC2 operational best practices.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* Administrative access to the AWS Management Console.
* The resources created in previous labs:

  * `devops-vpc`
  * `public-subnet-1a`
  * `web-sg`
  * `EC2-DevOps-Role`
* SSH client:

  * Linux/macOS Terminal
  * PuTTY (Windows)

> **Important**
>
> This lab builds on the networking and IAM resources created in **LAB 1** and **LAB 2**.

---

# Task 1 — Launch an Amazon EC2 Instance

## Step 1 — Open the Amazon EC2 Console

1. Sign in to the AWS Management Console.
2. Search for **EC2**.
3. Click **Launch instance**.

---

## Step 2 — Configure the Instance

Configure the instance using the following settings.

| Setting       | Value                                        |
| ------------- | -------------------------------------------- |
| Instance Name | `devops-web-01`                              |
| AMI           | Ubuntu Server 22.04 LTS (Free Tier eligible) |
| Instance Type | `t2.micro` (Free Tier)                       |

### Create a Key Pair

Click **Create new key pair** and configure:

| Setting       | Value        |
| ------------- | ------------ |
| Key Pair Name | `devops-key` |
| Key Type      | RSA          |
| File Format   | `.pem`       |

Download the key pair and store it securely.

> **Warning**
>
> The private key (`.pem`) can only be downloaded once. Store it in a secure location.

---

## Step 3 — Configure Network Settings

Configure the networking options.

| Setting               | Value              |
| --------------------- | ------------------ |
| VPC                   | `devops-vpc`       |
| Subnet                | `public-subnet-1a` |
| Auto-assign Public IP | Enable             |
| Security Group        | `web-sg`           |

---

## Step 4 — Configure Storage

Configure the instance storage.

### Root Volume

| Volume      | Size | Type          |
| ----------- | ---- | ------------- |
| Root Volume | 8 GB | gp3 (Default) |

### Additional EBS Volume

Click **Add new volume**.

| Volume            | Size      | Device     |
| ----------------- | --------- | ---------- |
| Additional Volume | 10 GB gp3 | `/dev/sdb` |

Enable **Delete on termination** for both volumes.

---

## Step 5 — Configure Advanced Details

Select the IAM instance profile.

| Setting              | Value             |
| -------------------- | ----------------- |
| IAM Instance Profile | `EC2-DevOps-Role` |

### User Data (Bootstrap Script)

Paste the following script.

```bash
#!/bin/bash
apt-get update -y
apt-get install -y nginx
systemctl start nginx
```

---

## Step 6 — Launch the Instance

1. Review the configuration.
2. Click **Launch instance**.
3. Wait **2–3 minutes** until the instance status shows:

```text
2/2 checks passed
```

4. Copy the **Public IP address**.

---

# Task 2 — Connect via SSH & Manage Amazon EBS

## Step 1 — Connect to the EC2 Instance

Open:

* Linux/macOS Terminal
* PuTTY (Windows)

Update the permissions of the private key.

```bash
chmod 400 devops-key.pem
```

Connect to the instance.

```bash
ssh -i devops-key.pem ubuntu@<Public-IP>
```

Accept the SSH fingerprint prompt when prompted.

---

## Step 2 — Verify Attached Storage

List the available block devices.

```bash
lsblk
```

---

## Step 3 — Format the New Amazon EBS Volume

Format the attached volume.

```bash
sudo mkfs -t ext4 /dev/xvdb
```

---

## Step 4 — Create a Mount Point

Create a directory for the new volume.

```bash
sudo mkdir /data
```

Mount the volume.

```bash
sudo mount /dev/xvdb /data
```

---

## Step 5 — Configure Persistent Mount

Add the following entry to `/etc/fstab`.

```bash
echo '/dev/xvdb /data ext4 defaults,nofail 0 2' | sudo tee -a /etc/fstab
```

This ensures that the volume is automatically mounted after every reboot.

---

# Task 3 — Create an AMI & Auto Scaling Group

## Step 1 — Create an AMI from the Running Instance

Navigate to:

```text
EC2 → Instances
```

Select:

```text
devops-web-01
```

Navigate to:

```text
Actions → Image and templates → Create image
```

Configure:

| Setting    | Value                            |
| ---------- | -------------------------------- |
| Image Name | `devops-web-ami`                 |
| No Reboot  | Unchecked (for data consistency) |

Click **Create image**.

Wait until the image status changes to:

```text
Available
```

---

## Step 2 — Create a Launch Template

Navigate to:

```text
EC2 → Launch templates → Create launch template
```

Configure the following settings.

| Setting              | Value                      |
| -------------------- | -------------------------- |
| Launch Template Name | `devops-lt`                |
| Template Version     | `1`                        |
| AMI                  | `devops-web-ami` (My AMIs) |
| Instance Type        | `t2.micro`                 |
| Key Pair             | `devops-key`               |
| Security Group       | `web-sg`                   |
| IAM Instance Profile | `EC2-DevOps-Role`          |

---

## Step 3 — Create an Auto Scaling Group (ASG)

Navigate to:

```text
EC2 → Auto Scaling Groups → Create
```

Configure the following.

| Setting                 | Value                                  |
| ----------------------- | -------------------------------------- |
| Auto Scaling Group Name | `devops-asg`                           |
| Launch Template         | `devops-lt`                            |
| VPC                     | `devops-vpc`                           |
| Subnets                 | `public-subnet-1a`, `public-subnet-1b` |

### Capacity Settings

| Property         | Value |
| ---------------- | ----- |
| Desired Capacity | `2`   |
| Minimum Capacity | `1`   |
| Maximum Capacity | `4`   |

### Scaling Policy

| Policy                 | Value           |
| ---------------------- | --------------- |
| Type                   | Target Tracking |
| Target CPU Utilization | 70%             |

Review the configuration and click **Create Auto Scaling Group**.

---

# Best Practice Tips

> **Tip**
>
> Follow these Amazon EC2 operational best practices to improve security, reliability, and scalability.

* Always use **IAM Roles** for EC2 instead of storing AWS access keys on the instance.
* Enable **Detailed Monitoring** to collect Amazon CloudWatch metrics at **1-minute intervals**.
* Use **gp3** Amazon EBS volumes instead of **gp2** because they provide similar performance at a lower cost.
* Use **EC2 Instance Connect** or **AWS Systems Manager Session Manager** instead of opening SSH (port 22) whenever possible.
* Configure Auto Scaling Group termination policies such as **OldestInstance** to simplify AMI rollouts.
* Create regular Amazon EBS snapshots or use **AWS Backup** to automate backup policies.

---

# Alternative Tool — AWS Systems Manager Session Manager

> **Note**
>
> AWS Systems Manager Session Manager provides secure shell access without exposing SSH (port 22).

Benefits include:

* No requirement to open port **22**.
* Uses the **AWS Systems Manager (SSM) Agent**, which is pre-installed on:

  * Amazon Linux 2
  * Ubuntu
* Requires an IAM role with the appropriate SSM permissions.

To connect:

```text
AWS Console → Systems Manager → Session Manager → Start session
```

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Launched an Amazon EC2 instance.
* ✅ Created and downloaded an EC2 key pair.
* ✅ Configured networking and security settings.
* ✅ Attached and mounted an Amazon EBS volume.
* ✅ Configured persistent storage using `/etc/fstab`.
* ✅ Created an Amazon Machine Image (AMI).
* ✅ Created a Launch Template.
* ✅ Configured an Auto Scaling Group (ASG).
* ✅ Reviewed Amazon EC2 operational best practices.
* ✅ Explored AWS Systems Manager Session Manager as a secure alternative to SSH.
