# Amazon ECS Fargate

> **Objective**
>
> Create an Amazon ECS cluster, define Task Definitions, run containerized services using AWS Fargate, and attach an Application Load Balancer (ALB) for traffic distribution.

---

# Learning Objectives

After completing this lab, you will be able to:

* Create an Amazon ECS cluster using AWS Fargate.
* Configure ECS Task Definitions.
* Deploy containerized applications as ECS Services.
* Integrate ECS Services with an Application Load Balancer.
* Scale ECS services by increasing task count.
* Apply ECS production best practices.
* Understand when to use ECS EC2 Launch Type instead of Fargate.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* Administrative access to AWS.
* A Docker image stored in Amazon ECR.

Example image:

```text
ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-nginx:v1
```

* A configured VPC:

```text
devops-vpc
```

* Public subnets available for the Application Load Balancer.

> **Important**
>
> Replace `ACCOUNT-ID` with your AWS Account ID.

---

# Task 1 — Create ECS Cluster & Task Definition

## Step 1 — Create an ECS Cluster

1. Search for **ECS** in the AWS Console.
2. Navigate to:

```text
Clusters → Create cluster
```

Configure the cluster.

| Setting        | Value                     |
| -------------- | ------------------------- |
| Cluster Name   | `devops-cluster`          |
| Infrastructure | AWS Fargate (serverless)  |
| Monitoring     | Enable Container Insights |

Click **Create**.

---

# Step 2 — Create an ECS Task Definition

Navigate to:

```text
ECS → Task definitions → Create new task definition
```

Configure the task definition.

| Setting                | Value               |
| ---------------------- | ------------------- |
| Task Definition Family | `devops-nginx-task` |
| Launch Type            | AWS Fargate         |
| Operating System       | Linux               |
| Architecture           | X86_64              |
| CPU                    | 0.25 vCPU           |
| Memory                 | 0.5 GB              |

---

# Step 3 — Add Container Definition

Add a container configuration.

| Setting        | Value                                                         |
| -------------- | ------------------------------------------------------------- |
| Container Name | `nginx-container`                                             |
| Image URI      | `ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-nginx:v1` |
| Port Mapping   | 80/TCP                                                        |

Configure the container health check.

```text
CMD-SHELL, curl -f http://localhost/ || exit 1
```

Click:

```text
Add container → Create
```

---

# Task 2 — Create ECS Service with Application Load Balancer

## Step 1 — Create Application Load Balancer

Before creating the ECS service, create an Application Load Balancer.

Navigate to:

```text
EC2 → Load balancers → Create load balancer
```

Select:

```text
Application Load Balancer
```

Configure the load balancer.

| Setting        | Value               |
| -------------- | ------------------- |
| Name           | `devops-alb`        |
| Scheme         | Internet-facing     |
| VPC            | `devops-vpc`        |
| Subnets        | Both public subnets |
| Security Group | `web-sg`            |
| Listener       | HTTP : 80           |

---

## Create Target Group

Create a target group with:

| Setting           | Value        |
| ----------------- | ------------ |
| Target Type       | IP           |
| Target Group Name | `devops-tg`  |
| Port              | 80           |
| VPC               | `devops-vpc` |

Click:

```text
Create load balancer
```

---

# Step 2 — Create ECS Service

Navigate to:

```text
ECS → devops-cluster → Services → Create
```

Configure the service.

| Setting         | Value                  |
| --------------- | ---------------------- |
| Launch Type     | AWS Fargate            |
| Task Definition | `devops-nginx-task`    |
| Service Name    | `devops-nginx-service` |
| Desired Tasks   | 2                      |
| VPC             | `devops-vpc`           |
| Subnets         | Public subnets         |
| Security Group  | `web-sg`               |

Configure the load balancer.

| Setting            | Value                     |
| ------------------ | ------------------------- |
| Load Balancer Type | Application Load Balancer |
| Load Balancer Name | `devops-alb`              |
| Container          | `nginx-container:80`      |
| Target Group       | `devops-tg`               |

Create the service.

---

# Step 3 — Verify the ECS Service

Wait until ECS tasks reach:

```text
RUNNING
```

Verify the deployment.

1. Copy the Application Load Balancer DNS name.
2. Open the DNS name in a browser.

Expected result:

```text
DevOps Lab HTML page
```

---

# Step 4 — Scale the ECS Service

Increase the number of running tasks.

Navigate to:

```text
ECS → Service → Update service
```

Change:

```text
Desired tasks: 4
```

Click:

```text
Update
```

Observe ECS launching additional tasks across Availability Zones.

---

# Best Practice Tips

> **Tip**
>
> Follow these ECS Fargate best practices for production container workloads.

* Use **Fargate Spot** for non-critical workloads.

Benefits:

* Lower cost.

* Suitable for fault-tolerant applications.

* Can provide savings of up to 70% compared with Fargate On-Demand.

* Always use **private subnets** for ECS tasks.

Recommended architecture:

```text
Internet
   |
Application Load Balancer (Public Subnet)
   |
ECS Tasks (Private Subnets)
```

* Use **ECS Exec** for debugging running containers.

Example:

```bash
aws ecs execute-command
```

* Enable **Service Auto Scaling** using:

  * CPU utilization
  * Memory utilization
  * Target tracking policies

* Configure the health check grace period based on application startup time to avoid premature task termination.

* Use:

  * ECS Service Connect
  * AWS App Mesh

for advanced service-to-service communication.

---

# Alternative Tool — ECS EC2 Launch Type

> **Note**
>
> Amazon ECS supports two main compute options: AWS Fargate and EC2 Launch Type.

## ECS EC2 Launch Type

Use EC2 Launch Type when you require:

* GPU workloads.
* Custom kernel settings.
* Windows containers.
* Full control over EC2 instances.

---

## Operational Considerations

EC2 Launch Type requires management of:

* EC2 instance provisioning.
* Operating system patching.
* Instance scaling.
* Capacity planning.

This introduces additional operational overhead.

---

## Recommendation

Use:

```text
AWS Fargate
```

unless there is a specific requirement to manage EC2 infrastructure directly.

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Created an Amazon ECS Fargate cluster.
* ✅ Enabled Container Insights monitoring.
* ✅ Created an ECS Task Definition.
* ✅ Configured an Nginx container from Amazon ECR.
* ✅ Added container health checks.
* ✅ Created an Application Load Balancer.
* ✅ Created an ECS Service.
* ✅ Connected ECS tasks with an ALB target group.
* ✅ Verified application availability.
* ✅ Scaled ECS tasks across Availability Zones.
* ✅ Reviewed ECS production best practices.
* ✅ Compared Fargate with ECS EC2 Launch Type.
