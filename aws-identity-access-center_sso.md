<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/cf99415e-d126-458e-940a-644b1991221a" />
# AWS IAM Identity Center (SSO) – From Basics to Expert (Hands-on Labs)
# APPENDIX – AWS ORGANIZATIONS & ACCOUNT ONBOARDING (FOUNDATION)

> This section is intentionally placed at the **end of the README** for reference, but in real-world projects this setup is usually completed **before** IAM Identity Center configuration.

---

## A1. Why AWS Organizations Matters for IAM Identity Center

IAM Identity Center (SSO) is designed to work with **AWS Organizations** to provide:

* Centralized access across multiple AWS accounts
* Automatic IAM role creation per account
* Scalable permission management using permission sets

Without AWS Organizations:

* Access must be managed per account
* Enterprise patterns are difficult to implement

---

## A2. Management Account vs Member Accounts

| Account Type       | Responsibility                              |
| ------------------ | ------------------------------------------- |
| Management Account | Billing, Organizations, IAM Identity Center |
| Security Account   | CloudTrail, Security Hub, audit logs        |
| Shared Services    | CI/CD, DNS, tooling                         |
| Development        | Dev workloads                               |
| Production         | Prod workloads                              |

> IAM Identity Center is **enabled only in the Management Account**.

---

## A3. Create or Enable AWS Organization

### Steps

1. Login to the **Management Account**
2. Open **AWS Organizations**
3. Click **Create organization**
4. Select **All features** (required for SCPs & SSO)

✅ Outcome: AWS Organization created

---

## A4. Create Organizational Units (OUs)

### Recommended Structure

```
Root
├── Security
├── Infrastructure
├── Development
└── Production
```

### Steps

1. AWS Organizations → Organizational units
2. Create OUs as per structure

✅ Outcome: Logical separation of accounts

---

## A5. Add AWS Accounts to Organization

There are **two supported methods**.

---

### Option 1: Invite Existing AWS Account

Use this when an AWS account already exists.

#### Steps

1. AWS Organizations → Accounts
2. Click **Invite account**
3. Enter **Account ID or email**
4. Send invitation
5. Accept invitation from the target account

✅ Outcome: Existing account joins the organization

---

### Option 2: Create New AWS Account (Recommended)

Use this for Dev / Prod / Security accounts.

#### Steps

1. AWS Organizations → Accounts
2. Click **Add an AWS account**
3. Choose **Create account**
4. Provide:

   * Account name
   * Unique email address
5. Create account

⏳ Takes ~2–5 minutes

✅ Outcome: New AWS account created and managed

---

## A6. Move Accounts into OUs

### Steps

1. AWS Organizations → Accounts
2. Select the account
3. Move it into the appropriate OU

✅ Outcome: Accounts aligned with governance model

---

## A7. How IAM Identity Center Uses AWS Organizations

When IAM Identity Center is enabled:

* It automatically discovers all org accounts
* Permission sets create **IAM roles** in each account
* Users assume roles via the AWS access portal

### Access Flow

```
User → IAM Identity Center → Permission Set → IAM Role → AWS Account
```

---

## A8. Common Interview & Real-World Questions

**Q: Can IAM Identity Center work without AWS Organizations?**
A: Yes, but it is **not recommended** for multi-account environments.

**Q: Where is IAM Identity Center configured?**
A: Only in the **Management Account**.

**Q: Does IAM Identity Center create IAM users?**
A: No. It creates and manages **IAM roles** automatically.

---

## A9. Best Practices

* Always enable **All features** in AWS Organizations
* Separate Prod and Dev accounts
* Centralize logs in Security account
* Combine Organizations + SCPs + IAM Identity Center

---

This document is a **step-by-step practical guide** to learn **AWS IAM Identity Center (formerly AWS SSO)** from **basics → intermediate → expert level**, with **clear labs** you can run in your own AWS account.

---

## Prerequisites

* Basic AWS knowledge (IAM, EC2, accounts)
* An AWS Organization (recommended)
* One **Management Account**
* At least 2 member accounts (Dev, Prod)
* Email access for test users

---

## Learning Path

* Level 1: Basics (Concepts + Simple Setup)
* Level 2: Practical Core (Users, Groups, Permission Sets)
* Level 3: Multi‑Account Access with AWS Organizations
* Level 4: Enterprise Patterns (IdP, SCIM, MFA)
* Level 5: Expert (Security, Automation, Troubleshooting)

---

# LEVEL 1 – BASICS

## 1. What is AWS IAM Identity Center?

AWS IAM Identity Center provides **centralized authentication and authorization** for:

* Multiple AWS accounts
* AWS console & CLI access
* Cloud & SaaS applications

### Key Components

* **Identity Source**: Where users come from (AWS SSO Directory / External IdP)
* **Users & Groups**: Who can log in
* **Permission Sets**: What they can do
* **IAM Roles**: Created automatically in target accounts
* **AWS Organizations**: Account structure

---

## 2. Core Architecture (Conceptual)

```
User → Identity Provider → IAM Identity Center → Permission Set → IAM Role → AWS Account
```

---

# LEVEL 2 – CORE PRACTICAL (LABS)

## LAB 1: Enable IAM Identity Center

### Steps

1. Login to **Management Account**
2. Go to **IAM Identity Center**
3. Click **Enable**
4. Choose **Identity source = IAM Identity Center directory**
5. Note the **AWS access portal URL**

✅ Outcome: SSO is enabled

---

## LAB 2: Create Users

### Steps

1. IAM Identity Center → Users → Add user
2. Create:

   * user1-dev
   * user2-readonly
3. Provide email and username
4. Save

✅ Outcome: Users created

---

## LAB 3: Create Groups

### Steps

1. IAM Identity Center → Groups → Create group
2. Create groups:

   * Dev-Team
   * ReadOnly-Team
3. Add users to respective groups

✅ Outcome: User management via groups

---

## LAB 4: Create Permission Sets

### Dev Permission Set

1. Permission sets → Create
2. Type: **AWS managed policy**
3. Attach: `AdministratorAccess`
4. Name: `Dev-Admin`

### ReadOnly Permission Set

1. Create new permission set
2. Attach: `ReadOnlyAccess`
3. Name: `ReadOnly-Access`

✅ Outcome: Permissions defined

---

# LEVEL 3 – MULTI ACCOUNT ACCESS

## LAB 5: Attach AWS Accounts

### Steps

1. IAM Identity Center → AWS Accounts
2. Select **Development Account**
3. Assign:

   * Group: Dev-Team → Dev-Admin
   * Group: ReadOnly-Team → ReadOnly-Access
4. Repeat for Production account (restricted access)

✅ Outcome: Cross‑account access configured

---

## LAB 6: Login & Verify Access

### Steps

1. Open AWS access portal URL
2. Login as `user1-dev`
3. Select Development account
4. Verify admin access
5. Login as `user2-readonly`
6. Confirm read-only behavior

✅ Outcome: End‑to‑end SSO working

---

# LEVEL 4 – ENTERPRISE SETUP

## LAB 7: Enable MFA

### Steps

1. IAM Identity Center → Settings → Authentication
2. Enable **MFA required**
3. Configure:

   * Authenticator app
   * Security key (optional)

✅ Outcome: MFA enforced

---

## LAB 8: External Identity Provider (Okta / Azure AD)

### High-Level Steps

1. Configure IdP (Okta / Azure AD)
2. Create SAML 2.0 app
3. Exchange metadata with AWS
4. Enable SCIM provisioning
5. Test login via IdP

✅ Outcome: Corporate SSO integration

---

# LEVEL 5 – EXPERT / ADVANCED

## LAB 9: CLI Access using SSO

### Configure CLI

```bash
aws configure sso
```

### Verify

```bash
aws sts get-caller-identity
```

✅ Outcome: Secure CLI access without access keys

---

## LAB 10: Audit & Logging

### Services Used

* AWS CloudTrail
* AWS CloudWatch Logs
* AWS Config
* AWS Security Hub

### What to Monitor

* AssumeRole events
* PermissionSet changes
* Login failures

---

## LAB 11: Permission Set Best Practices

* Avoid AdministratorAccess in Prod
* Use job‑based permission sets
* Separate Dev / Prod roles
* Short session durations

---

## LAB 12: Troubleshooting

### Common Issues

| Issue              | Cause               | Fix                   |
| ------------------ | ------------------- | --------------------- |
| No account visible | No assignment       | Assign permission set |
| Access denied      | SCP or IAM conflict | Check SCP             |
| CLI fails          | SSO expired         | Re-login              |

---

# Best Practices (Expert)

* Always use Groups (not users)
* Enforce MFA
* Use least privilege
* Centralize logs in Security account
* Automate via Terraform / CloudFormation

---
# Prepared by:
**shaik Moulali**
