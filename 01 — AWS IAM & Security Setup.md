# AWS IAM & Security Setup

> **Objective**
>
> Configure IAM users, groups, roles, and policies. Enable Multi-Factor Authentication (MFA) and enforce the principle of least privilege for AWS services.

---

# Learning Objectives

After completing this lab, you will be able to:

* Create IAM users and user groups.
* Assign AWS managed policies.
* Configure Multi-Factor Authentication (MFA).
* Create IAM roles for Amazon EC2.
* Create and attach an inline IAM policy.
* Apply AWS IAM security best practices.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* Root user or administrative access to the AWS Management Console.
* Internet connectivity.
* A mobile authenticator application (Google Authenticator or Authy).

> **Important**
>
> The **AdministratorAccess** policy is used **for lab purposes only**. In production environments, always follow the **Principle of Least Privilege**.

---

# Task 1 — Create IAM User & Group

## Step 1 — Sign in to AWS Console

1. Go to:

   ```text
   https://console.aws.amazon.com
   ```

2. Log in as the **root user** or an **administrator user**.

3. Search for **IAM** using the AWS search bar.

---

## Step 2 — Navigate to the IAM Dashboard

1. Open **IAM** under **Security, Identity & Compliance**.
2. Select **Users** from the left navigation panel.

---

## Step 3 — Create an IAM Group

1. Select **User groups**.
2. Click **Create group**.
3. Configure the following:

| Setting         | Value                              |
| --------------- | ---------------------------------- |
| Group Name      | `devops-team`                      |
| Attached Policy | `AdministratorAccess` *(Lab only)* |

4. Click **Create group**.

> **Warning**
>
> Restrict permissions appropriately in production environments instead of assigning **AdministratorAccess**.

---

## Step 4 — Create an IAM User

1. Navigate to **Users**.
2. Click **Add users**.
3. Configure the following settings.

| Setting        | Value                                              |
| -------------- | -------------------------------------------------- |
| Username       | `devops-user01`                                    |
| Console Access | Provide user access to the AWS Management Console  |
| Password       | Set a console password                             |
| Password Reset | Uncheck **User must change password** *(Lab only)* |

4. Click **Next**.
5. Add the user to the following group:

```text
devops-team
```

6. Click:

```text
Next → Create user
```

7. Download the generated **.csv** credentials file.

> **Note**
>
> Store the downloaded credentials securely.

---

## Step 5 — Verify User Login

1. Copy the **Console sign-in URL** from the IAM dashboard.
2. Open a private/incognito browser window.
3. Sign in as:

```text
devops-user01
```

4. Confirm that the user can successfully access the AWS Management Console.

---

# Task 2 — Enable MFA for IAM User

## Step 1 — Open User Security Credentials

1. Navigate to:

```text
IAM → Users → devops-user01
```

2. Open the **Security credentials** tab.

---

## Step 2 — Assign an MFA Device

1. Click **Assign MFA device**.
2. Configure the following:

| Setting     | Value               |
| ----------- | ------------------- |
| Device Name | `devops-user01-mfa` |
| Device Type | Authenticator app   |

3. Click **Next**.

---

## Step 3 — Scan the QR Code

1. Open **Google Authenticator** or **Authy**.
2. Tap the **+** icon.
3. Select **Scan QR code**.
4. Scan the QR code displayed by AWS.
5. Enter the two consecutive One-Time Passwords (OTP).
6. Click **Add MFA**.

---

## Step 4 — Test MFA

1. Sign out of AWS.
2. Sign back in as:

```text
devops-user01
```

3. Enter:

* Password
* MFA verification code

4. Confirm successful authentication.

---

# Task 3 — Create IAM Role for Amazon EC2

## Step 1 — Create the IAM Role

Navigate to:

```text
IAM → Roles → Create role
```

Configure the following:

| Setting             | Value       |
| ------------------- | ----------- |
| Trusted Entity Type | AWS service |
| Use Case            | EC2         |

Click **Next**.

---

## Step 2 — Attach Permissions

Attach the following AWS managed policies:

* `AmazonS3ReadOnlyAccess`
* `CloudWatchAgentServerPolicy`

Click:

```text
Next
```

Configure:

| Setting   | Value             |
| --------- | ----------------- |
| Role Name | `EC2-DevOps-Role` |

Click **Create role**.

---

## Step 3 — Create an Inline Policy (Custom)

Navigate to:

```text
IAM → Roles → EC2-DevOps-Role
```

Select:

```text
Add permissions → Create inline policy
```

Open the **JSON** tab and paste the following policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:GetParameter"
      ],
      "Resource": "*"
    }
  ]
}
```

Configure:

| Setting     | Value                  |
| ----------- | ---------------------- |
| Policy Name | `SSM-ReadParam-Policy` |

Click **Create policy**.

---

# Best Practice Tips

> **Tip**
>
> Follow these AWS IAM security recommendations when designing access controls.

* Always follow the **Principle of Least Privilege** by granting only the permissions that are required.
* Never use the **root account** for day-to-day operations. Create named IAM users instead.
* Rotate **access keys every 90 days** and use **AWS Secrets Manager** or **AWS Systems Manager Parameter Store** for managing secrets.
* Enable **AWS CloudTrail** to record all IAM API calls for auditing and compliance.
* Use **IAM Access Analyzer** to identify overly permissive IAM policies.
* Enforce a strong password policy:

  * Minimum of **14 characters**
  * Uppercase letters
  * Numbers
  * Special symbols

---

# Alternative Tool — Microsoft Entra ID (Azure Active Directory)

> **Note**
>
> Organizations operating hybrid or multi-cloud environments can integrate AWS identity management with Microsoft Entra ID.

* In Microsoft Azure, use **Azure Active Directory (Microsoft Entra ID)** with:

  * App Registrations
  * Managed Identities
* For hybrid identity management, configure **SAML 2.0 federation** between **AWS IAM Identity Center** and **Azure Active Directory**.
* Use **SCIM provisioning** to automatically synchronize users from **Azure Active Directory** to AWS.

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Created an IAM user group.
* ✅ Created an IAM user.
* ✅ Assigned AWS managed policies.
* ✅ Enabled Multi-Factor Authentication (MFA).
* ✅ Created an IAM role for Amazon EC2.
* ✅ Created a custom inline IAM policy.
* ✅ Reviewed AWS IAM security best practices.
* ✅ Explored Microsoft Entra ID integration for hybrid identity management.
