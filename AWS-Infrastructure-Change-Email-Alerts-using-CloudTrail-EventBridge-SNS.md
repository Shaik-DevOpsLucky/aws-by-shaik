# 📧 AWS Infrastructure Change Email Alerts using CloudTrail, EventBridge & SNS

## 🏗️ Architecture Diagram

```text
+---------------------------+
|   AWS Users / Pipelines   |
| (Console, CLI, CI/CD)     |
+-------------+-------------+
              |
              v
+---------------------------+
|     AWS API Actions       |
| (Create / Update / Delete|
|  EC2, IAM, S3, RDS, etc.) |
+-------------+-------------+
              |
              v
+---------------------------+
|        AWS CloudTrail     |
|  (Records all API calls)  |
+-------------+-------------+
              |
              v
+---------------------------+
|     Amazon EventBridge    |
| (Filters infra & IAM      |
|  management events)       |
+-------------+-------------+
              |
              v
+---------------------------+
|        Amazon SNS         |
|   (Notification service)  |
+-------------+-------------+
              |
              v
+---------------------------+
|        Email Inbox        |
| (Infra Change Alert 📧)   |
+---------------------------+
```

This guide explains **how to get an email alert whenever any user creates, updates, or deletes AWS resources** (EC2, IAM, S3, RDS, EKS, etc.).

It is written for **beginners to advanced users**, with **clear practical steps** that anyone can follow.

---

## 🎯 What Problem Are We Solving?

### Real-Life Situation (Easy to Understand)

In a real AWS account:

* Many **people** (admins, developers)
* Many **systems** (CI/CD pipelines, Terraform, CloudFormation)
* Many **AWS services** (Auto Scaling, EventBridge, Backup)

👉 **All of them can change infrastructure**.

Without alerts, you may NOT know:

* Who deleted an EC2 instance ❓
* Who created a new IAM user ❓
* Who changed a security group ❓
* Whether the change was manual or automated ❓

This setup solves that problem by sending you an **email for every important change**.

In AWS, many users, pipelines, or tools can change infrastructure. Without monitoring:

* You don’t know **who changed what**
* Mistakes go unnoticed
* Security incidents can be missed

### ✅ Goal

Whenever **any AWS resource is changed**, you receive an **email notification**.

---

## 🏗️ Final Architecture

```
User / Pipeline / AWS Console
        |
        v
AWS API Call (Create / Update / Delete)
        |
        v
AWS CloudTrail (records the event)
        |
        v
Amazon EventBridge (filters important events)
        |
        v
Amazon SNS (email notification)
        |
        v
📧 Your Email Inbox
```

---

## 🧩 Services Used

| Service            | Purpose                      |
| ------------------ | ---------------------------- |
| AWS CloudTrail     | Records all AWS API activity |
| Amazon EventBridge | Detects specific changes     |
| Amazon SNS         | Sends email notifications    |

---

## ✅ Prerequisites

* AWS Account
* IAM permissions for:

  * CloudTrail
  * EventBridge
  * SNS
* Valid email address

---

# STEP 1️⃣: Create CloudTrail (Audit Logging)

CloudTrail records **every AWS API call**.

### 1. Open AWS Console → **CloudTrail**

### 2. Click **Create trail**

### General Details

* Trail name: `org-infra-audit-trail`
* Apply to all regions: ✅ Yes

### Storage

* S3 bucket: Create new **or** use existing
* Prefix (optional): `cloudtrail/`

### Encryption

* Log file SSE-KMS encryption: ✅ Enabled
* Select: **AWS managed key (aws/cloudtrail)** (recommended)

### Additional Settings

* Log file validation: ✅ Enabled
* SNS notification: ❌ Disable (not needed here)
* CloudWatch Logs: Optional

### Log Events (IMPORTANT)

* Management events: ✅ Enabled

  * Read events: ✅
  * Write events: ✅ (CRITICAL)
* Data events: ❌ Disabled

### Click **Create trail**

✅ CloudTrail is now logging all infrastructure and IAM activity.

---

# STEP 2️⃣: Create SNS Topic (Email Notifications)

SNS sends emails when an event occurs.

### 1. Open AWS Console → **SNS**

### 2. Go to **Topics → Create topic**

* Type: `Standard`
* Name: `infra-change-alerts`

### Create Email Subscription

1. Open the topic
2. Click **Create subscription**
3. Protocol: `Email`
4. Endpoint: `your-email@example.com`
5. Create subscription

📩 **Confirm the email subscription** (mandatory)

---

# STEP 3️⃣: Create EventBridge Rule (Detect Changes)

EventBridge listens to CloudTrail and sends matching events to SNS.

### 1. Open AWS Console → **EventBridge**

### 2. Go to **Rules → Create rule**

### Rule Details

* Name: `infra-change-events`
* Event bus: `default`
* Rule type: **Rule with an event pattern**

---

## Event Pattern (Recommended – Low Noise)

Paste this JSON:

```json
{
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventCategory": ["Management"],
    "userIdentity": {
      "type": ["IAMUser", "AssumedRole", "Root"]
    }
  }
}
```

### What This Captures

| Event                 | Alert |
| --------------------- | ----- |
| IAM user actions      | ✅     |
| Human assumed roles   | ✅     |
| Root user actions     | 🚨    |
| AWS internal services | ❌     |

---

## Target Configuration

* Target type: **AWS service**
* Service: **SNS topic**
* Topic: `infra-change-alerts`

### Permissions

* Select: **Create a new role for this rule**

### Click **Create rule**

---

# STEP 4️⃣: Scenarios & How Alerts Work (MOST IMPORTANT)

Below are **clear real-world scenarios** so you understand **what email you will get and why**.

---

## ✅ Scenario 1: IAM User Creation

### Action

* Admin creates a new IAM user from AWS Console

### What Happens Internally

```
IAM User → AWS API → CloudTrail → EventBridge → SNS → Email
```

### Email Meaning

* Someone created a **new identity** in your AWS account
* This is a **security-sensitive event**

---

## ✅ Scenario 2: EC2 Instance Terminated

### Action

* Developer terminates an EC2 instance

### Email Meaning

* Infrastructure was **deleted**
* Possible impact: downtime or data loss

---

## ✅ Scenario 3: Security Group Modified

### Action

* Port 22 (SSH) opened to `0.0.0.0/0`

### Email Meaning

* Network security was changed
* This could be a **security risk**

---

## ✅ Scenario 4: CI/CD Pipeline Changes Infra

### Action

* Terraform or Jenkins creates resources

### Email Meaning

* Change was automated
* User identity will show **AssumedRole**

---

## 🚨 Scenario 5: Root User Activity (Critical)

### Action

* Root account logs in or performs action

### Email Meaning

* **Very high risk event**
* Should be investigated immediately

---

## ❌ Scenario 6: AWS Internal Noise (Ignored)

### Example

* EventBridge assumes role to send email

### Result

* ❌ Filtered out
* ❌ No email sent

---

Try any of these actions:

* Create an IAM user
* Launch or terminate EC2
* Create or delete S3 bucket
* Modify security group

📧 You should receive an email immediately.

---

## Sample Email (Raw SNS Message)

```json
{
  "eventName": "RunInstances",
  "eventSource": "ec2.amazonaws.com",
  "userIdentity": { "type": "IAMUser" },
  "region": "us-east-1",
  "time": "2026-01-02T04:12:01Z"
}
```

---

# 📉 Too Many Emails? (Noise Reduction)

### Option 1: Ignore AssumeRole Events

```json
{
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventCategory": ["Management"],
    "eventName": [{ "anything-but": ["AssumeRole"] }]
  }
}
```

---

# 🔐 Advanced Scenarios

## Scenario 1: IAM-Only Alerts

```json
{
  "source": ["aws.iam"],
  "detail-type": ["AWS API Call via CloudTrail"]
}
```

## Scenario 2: Root User Alerts (Critical)

```json
{
  "detail": {
    "userIdentity": { "type": ["Root"] }
  }
}
```

## Scenario 3: Production Account Only

Filter using account ID in EventBridge rule.

---

# ⭐ Optional Enhancement (Professional Setup)

Add **AWS Lambda** between EventBridge and SNS to:

* Format emails
* Add emojis 🚨
* Send Slack / Teams alerts

```
EventBridge → Lambda → SNS → Email
```

---

# 🧠 Best Practices

* Enable CloudTrail in **all regions**
* Use **AWS-managed KMS key** unless required otherwise
* Separate rules for IAM, Infra, Root
* Centralize logs in security account

---

## ✅ Final Summary

✔️ CloudTrail records all changes
✔️ EventBridge filters important actions
✔️ SNS sends email alerts

🎉 You now have **real-time AWS infrastructure monitoring**.

---

## 📌 Author / Maintainer

Created for **learning, audit, and security monitoring**.

Feel free to fork, improve, and use in production 🚀
