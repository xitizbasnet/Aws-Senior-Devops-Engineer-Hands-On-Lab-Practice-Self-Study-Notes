# AWS CloudFormation (Infrastructure as Code)

> **Objective**
>
> Write AWS CloudFormation templates, deploy stacks, use parameters, create Change Sets, and explore the AWS Serverless Application Model (SAM).

---

# Learning Objectives

After completing this lab, you will be able to:

* Create a basic AWS CloudFormation template.
* Deploy infrastructure using CloudFormation stacks.
* Use CloudFormation parameters.
* Monitor stack creation events.
* Create and review Change Sets.
* Update existing CloudFormation stacks.
* Apply CloudFormation best practices.
* Explore the AWS Cloud Development Kit (CDK) as an Infrastructure as Code (IaC) alternative.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* Administrative access to the AWS Management Console.
* An existing EC2 key pair named `devops-key`.
* Basic knowledge of YAML syntax.

> **Important**
>
> Always validate CloudFormation templates before deployment to reduce deployment failures.

---

# Task 1 — Write & Deploy Your First CloudFormation Stack

## Step 1 — Open AWS CloudFormation

1. Sign in to the AWS Management Console.
2. Search for **CloudFormation**.
3. Navigate to:

```text
Stacks → Create stack
```

Configure the following options.

| Setting              | Value                  |
| -------------------- | ---------------------- |
| Template Preparation | Template is ready      |
| Template Source      | Upload a template file |

---

## Step 2 — Create a Basic EC2 Template

Create a file named:

```text
ec2-stack.yaml
```

Add the following template.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Basic EC2 with Security Group

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t3.small

  KeyName:
    Type: AWS::EC2::KeyPair::KeyName

Resources:
  WebSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP SSH
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0f5ee92e2d63afc18
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      SecurityGroupIds:
        - !Ref WebSG
      Tags:
        - Key: Name
          Value: cfn-web-server

Outputs:
  InstancePublicIP:
    Value: !GetAtt WebServer.PublicIp
```

Upload the template to AWS CloudFormation.

---

## Step 3 — Deploy the Stack

Configure the stack.

| Setting      | Value              |
| ------------ | ------------------ |
| Stack Name   | `devops-ec2-stack` |
| InstanceType | `t2.micro`         |
| KeyName      | `devops-key`       |

Click:

```text
Next → Next
```

Select:

* **I acknowledge that AWS CloudFormation might create IAM resources**

Click **Submit**.

---

## Step 4 — Monitor Stack Creation

Open the **Events** tab to monitor resource creation.

Wait until the stack status changes to:

```text
CREATE_COMPLETE
```

Open the **Outputs** tab to view the EC2 instance public IP address.

---

# Task 2 — Create a Change Set & Update the Stack

## Step 1 — Modify the CloudFormation Template

Edit the `ec2-stack.yaml` file.

Make one of the following changes:

* Change the default value of `InstanceType` to:

```text
t3.small
```

**OR**

* Add a new tag to the EC2 resource.

Save the updated template.

---

## Step 2 — Create a Change Set

Navigate to:

```text
CloudFormation → devops-ec2-stack → Stack actions → Create change set
```

Upload the modified template.

Configure:

| Setting         | Value                  |
| --------------- | ---------------------- |
| Change Set Name | `update-instance-type` |

Click **Create change set**.

---

## Step 3 — Review & Execute the Change Set

Review the proposed infrastructure changes.

CloudFormation identifies resource actions such as:

* Modify
* Replace
* Add

If the changes are correct:

1. Click **Execute change set**.
2. Monitor the **Events** tab.

Wait until the stack status becomes:

```text
UPDATE_COMPLETE
```

---

# Best Practice Tips

> **Tip**
>
> Follow these CloudFormation best practices when managing production infrastructure.

* Always create a **Change Set** before updating production stacks to review the exact infrastructure changes.
* Enable **Termination Protection** on production stacks to prevent accidental deletion.
* Use **Stack Policies** to restrict updates or replacement of critical resources.
* Organize large environments using **Nested Stacks** to separate components such as:

  * VPC
  * EC2
  * Amazon RDS
* Validate templates locally using **cfn-lint** before deployment.
* Store CloudFormation templates in **AWS CodeCommit**, **Amazon S3**, or another version-controlled repository.

---

# Alternative Tool — AWS Cloud Development Kit (AWS CDK)

> **Note**
>
> AWS CDK allows you to define cloud infrastructure using familiar programming languages instead of writing raw YAML or JSON templates.

Key capabilities include:

* Define infrastructure using:

  * Python
  * TypeScript
  * Java
* AWS CDK synthesizes infrastructure into AWS CloudFormation templates.
* Deploy infrastructure using:

```bash
cdk deploy
```

* Preview infrastructure changes before deployment using:

```bash
cdk diff
```

Additional benefits include:

* Improved IDE support
* Autocomplete
* Type checking
* Better code reuse compared to raw YAML templates

---

# Additional Note — AWS Serverless Application Model (SAM)

> **Note**
>
> AWS Serverless Application Model (SAM) extends AWS CloudFormation with simplified syntax for deploying serverless applications such as AWS Lambda, Amazon API Gateway, and Amazon DynamoDB.

SAM templates are transformed into standard CloudFormation templates during deployment.

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Created an AWS CloudFormation template.
* ✅ Defined an Amazon EC2 instance and Security Group.
* ✅ Used CloudFormation parameters.
* ✅ Deployed a CloudFormation stack.
* ✅ Monitored stack creation events.
* ✅ Retrieved stack outputs.
* ✅ Modified a CloudFormation template.
* ✅ Created and reviewed a Change Set.
* ✅ Updated an existing CloudFormation stack.
* ✅ Reviewed Infrastructure as Code (IaC) best practices.
* ✅ Explored AWS CDK and AWS SAM as modern CloudFormation-based alternatives.
