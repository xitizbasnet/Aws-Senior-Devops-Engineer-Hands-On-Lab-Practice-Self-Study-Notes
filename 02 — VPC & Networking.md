# VPC & Networking

> **Objective**
>
> Build a custom Amazon Virtual Private Cloud (VPC) with public and private subnets, an Internet Gateway, a NAT Gateway, Security Groups, and Network ACLs (NACLs).

---

# Learning Objectives

After completing this lab, you will be able to:

* Create a custom Amazon VPC.
* Configure public and private subnets.
* Enable DNS hostnames and DNS resolution.
* Create and attach an Internet Gateway (IGW).
* Configure public and private route tables.
* Deploy a NAT Gateway for outbound internet access.
* Configure Security Groups for web and application tiers.
* Create and associate Network ACLs (NACLs).
* Apply AWS networking best practices.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* Administrative access to the AWS Management Console.
* IAM permissions to manage Amazon VPC resources.

> **Important**
>
> This lab uses the **ap-south-1** Region for demonstration purposes. Adjust the Availability Zones if you are working in another AWS Region.

---

# Task 1 — Create a Custom VPC

## Step 1 — Open the Amazon VPC Console

1. Sign in to the AWS Management Console.
2. Search for **VPC** using the AWS search bar.
3. Navigate to:

   ```text
   Your VPCs → Create VPC
   ```

---

## Step 2 — Configure the VPC

Select **VPC only** and configure the following settings.

| Setting         | Value         |
| --------------- | ------------- |
| Resource Type   | VPC only      |
| Name            | `devops-vpc`  |
| IPv4 CIDR Block | `10.0.0.0/16` |
| Tenancy         | Default       |

Click **Create VPC**.

---

## Step 3 — Enable DNS Hostnames

1. Select **devops-vpc**.
2. Navigate to:

```text
Actions → Edit VPC settings
```

3. Enable the following options:

* Enable DNS hostnames
* Enable DNS resolution

4. Click **Save**.

---

# Task 2 — Create Subnets

## Step 1 — Create the Public Subnet

Navigate to:

```text
VPC → Subnets → Create subnet
```

Configure the following:

| Setting           | Value              |
| ----------------- | ------------------ |
| VPC ID            | `devops-vpc`       |
| Subnet Name       | `public-subnet-1a` |
| Availability Zone | `ap-south-1a`      |
| IPv4 CIDR         | `10.0.1.0/24`      |

Click **Add new subnet**.

---

## Step 2 — Create the Private Subnet

Configure the following:

| Setting           | Value               |
| ----------------- | ------------------- |
| Subnet Name       | `private-subnet-1b` |
| Availability Zone | `ap-south-1b`       |
| IPv4 CIDR         | `10.0.2.0/24`       |

Click **Create subnet**.

---

## Step 3 — Enable Auto-Assign Public IP

1. Select:

```text
public-subnet-1a
```

2. Navigate to:

```text
Actions → Edit subnet settings
```

3. Enable:

* **Auto-assign public IPv4 address**

4. Click **Save**.

---

# Task 3 — Configure the Internet Gateway & Route Tables

## Step 1 — Create and Attach an Internet Gateway

Navigate to:

```text
VPC → Internet gateways → Create internet gateway
```

Configure:

| Setting | Value        |
| ------- | ------------ |
| Name    | `devops-igw` |

Click **Create**.

Attach the Internet Gateway to the VPC:

```text
Actions → Attach to VPC → devops-vpc → Attach
```

---

## Step 2 — Create the Public Route Table

Navigate to:

```text
VPC → Route tables → Create route table
```

Configure:

| Setting | Value        |
| ------- | ------------ |
| Name    | `public-rt`  |
| VPC     | `devops-vpc` |

Click **Create**.

Add the default internet route.

Navigate to:

```text
Routes → Edit routes → Add route
```

| Destination | Target       |
| ----------- | ------------ |
| `0.0.0.0/0` | `devops-igw` |

Click **Save**.

---

## Step 3 — Associate the Public Subnet

Navigate to:

```text
public-rt → Subnet associations → Edit
```

Select:

```text
public-subnet-1a
```

Click **Save**.

---

## Step 4 — Create the NAT Gateway

Navigate to:

```text
VPC → NAT gateways → Create NAT gateway
```

Configure the following:

| Setting           | Value              |
| ----------------- | ------------------ |
| Subnet            | `public-subnet-1a` |
| Connectivity Type | Public             |

Click:

```text
Allocate Elastic IP
```

Then click:

```text
Create NAT gateway
```

> **Important**
>
> A NAT Gateway **must always be deployed in a public subnet**.

---

## Step 5 — Create the Private Route Table

Navigate to:

```text
Route tables → Create route table
```

Configure:

| Setting | Value        |
| ------- | ------------ |
| Name    | `private-rt` |
| VPC     | `devops-vpc` |

Click **Create**.

Edit the routes and add:

| Destination | Target      |
| ----------- | ----------- |
| `0.0.0.0/0` | NAT Gateway |

Associate the route table with:

```text
private-subnet-1b
```

Click **Save**.

---

# Task 4 — Configure Security Groups & Network ACLs

## Step 1 — Create the Web Security Group

Navigate to:

```text
VPC → Security groups → Create security group
```

Configure:

| Setting | Value        |
| ------- | ------------ |
| Name    | `web-sg`     |
| VPC     | `devops-vpc` |

### Inbound Rules

| Protocol | Port | Source              |
| -------- | ---- | ------------------- |
| HTTP     | 80   | `0.0.0.0/0`         |
| HTTPS    | 443  | `0.0.0.0/0`         |
| SSH      | 22   | Your public IP only |

### Outbound Rules

* Allow **All traffic**

Click **Create**.

---

## Step 2 — Create the Application Security Group

Configure:

| Setting | Value    |
| ------- | -------- |
| Name    | `app-sg` |

### Inbound Rules

| Protocol   | Port | Source                              |
| ---------- | ---- | ----------------------------------- |
| Custom TCP | 8080 | `web-sg` (Security Group reference) |

### Outbound Rules

* Allow **All traffic**

Click **Create**.

---

## Step 3 — Create a Network ACL (NACL)

Navigate to:

```text
VPC → Network ACLs → Create network ACL
```

Configure:

| Setting | Value         |
| ------- | ------------- |
| Name    | `public-nacl` |
| VPC     | `devops-vpc`  |

### Inbound Rules

| Rule Number | Protocol    |
| ----------- | ----------- |
| 100         | Allow HTTP  |
| 110         | Allow HTTPS |
| 120         | Allow SSH   |
| *           | Deny All    |

### Outbound Rules

| Rule Number | Action            |
| ----------- | ----------------- |
| 100         | Allow All Traffic |

Associate the Network ACL with:

```text
public-subnet-1a
```

---

# Best Practice Tips

> **Tip**
>
> Follow these networking best practices when designing AWS VPC environments.

* Use a **/24 CIDR block** for subnets. This provides **256 IP addresses**, with **251 usable** after AWS reserves five addresses.
* A **NAT Gateway must always reside in a public subnet**, never in a private subnet.
* Remember the key difference:

  * **Security Groups are stateful**
  * **Network ACLs are stateless**
* Tag all VPC resources with metadata such as:

  * Project
  * Environment
  * Owner
* Enable **VPC Flow Logs** to capture network traffic for troubleshooting, monitoring, and security auditing.
* Delete unused **NAT Gateways** to avoid unnecessary hourly charges.

---

# Alternative Tool — Terraform / AWS CloudFormation

> **Note**
>
> Infrastructure as Code (IaC) enables repeatable and automated network deployments.

### Terraform

Use the following Terraform resources to automate VPC creation:

* `aws_vpc`
* `aws_subnet`
* `aws_internet_gateway`

### AWS CloudFormation

Use the following CloudFormation resource types:

* `AWS::EC2::VPC`
* `AWS::EC2::Subnet`

You can also use the **VPC Wizard** in the AWS Management Console for quick deployments and then export the configuration to **CloudFormation** for repeatable infrastructure.

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Created a custom Amazon VPC.
* ✅ Configured public and private subnets.
* ✅ Enabled DNS hostnames and DNS resolution.
* ✅ Created and attached an Internet Gateway.
* ✅ Configured public and private route tables.
* ✅ Created a NAT Gateway for private subnet internet access.
* ✅ Configured Security Groups for web and application tiers.
* ✅ Created and associated a Network ACL.
* ✅ Reviewed AWS networking best practices.
* ✅ Explored Infrastructure as Code (Terraform and AWS CloudFormation) alternatives.
