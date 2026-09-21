# AWS IAM & IAM Advanced Topics

## Day 1 — IAM Fundamentals

---

# 1. IAM — Identity and Access Management

IAM (Identity and Access Management) is used to create **users and groups** and control access to AWS resources.

### IAM Users and Groups

- Each user can be assigned to one or more groups.
- A group can contain multiple users.
- Groups **cannot be nested inside other groups**.
- A user can belong to multiple groups.
- IAM is a **global AWS service**.

### IAM Permissions

IAM permissions allow us to follow the **principle of least privilege**.

Each user should receive only the permissions they need.

For example, if a user only needs to:

- Describe EC2 instances
- List S3 buckets
- View CloudWatch logs

We can provide only those required permissions.

### Best Practice

> **Assign users to groups and attach policies to the groups.**

IAM policies are written in **JSON format**.

Policies define what actions an identity is allowed or denied to perform on AWS resources.

---

# 2. IAM Policy Inheritance

IAM policies can be attached in different ways.

### Inline Policy

A policy can be assigned directly to an individual user.

```text
User
 └── Inline Policy
```

### Group Policy

A policy can be attached to a group, and users can inherit the permissions through group membership.

```text
Policy
   ↓
Group
   ↓
User
```

### Multiple Groups

A user can belong to multiple groups, and each group can have different policies.

```text
              ┌── Group A ── Policy A
User ─────────┤
              ├── Group B ── Policy B
              │
              └── Group C ── Policy C
```

Therefore, a user can receive permissions from multiple policies and groups.

---

# 3. IAM Policy Structure

IAM policies are JSON documents.

```json
{
  "Version": "2012-10-17",
  "Id": "ExamplePolicy",
  "Statement": [
    {
      "Sid": "AllowEC2Describe",
      "Effect": "Allow",
      "Action": "ec2:DescribeInstances",
      "Resource": "*"
    }
  ]
}
```

## Policy Elements

### Version

Specifies the version of the policy language.

### Id

An optional identifier for the policy.

### Statement

Contains one or more permission statements.

The statement can contain:

- `Sid`
- `Effect`
- `Action`
- `Principal`
- `Resource`
- `Condition`

### Sid

Statement identifier. It is optional.

### Effect

Specifies whether an action is:

- `Allow`
- `Deny`

### Action

Specifies the AWS API actions that are allowed or denied.

Example:

```json
"Action": "ec2:DescribeInstances"
```

### Principal

Specifies **who** is allowed or denied access.

Examples:

- AWS account
- IAM user
- IAM role
- Federated principal

`Principal` is generally used in **resource-based policies** and is not normally used in identity-based policies.

### Resource

Specifies the AWS resource to which the policy applies.

Example:

```json
"Resource": "*"
```

---

# 4. IAM Password Policy

AWS IAM allows an organization to enforce password requirements.

We can configure requirements such as:

- Minimum password length
- Uppercase characters
- Lowercase characters
- Numbers
- Non-alphanumeric characters
- Password expiration
- Password reuse prevention
- Allow users to change their own passwords
- Require users to change their password after a certain period

The purpose is to enforce stronger password security.

---

# 5. MFA — Multi-Factor Authentication

MFA provides an additional authentication factor.

For example:

```text
Something you know
        ↓
     Password
        +
Something you have
        ↓
Authenticator / MFA Device
```

If a user's password is compromised, an attacker would still need access to the MFA device to authenticate.

AWS allows organizations to **enforce MFA** for users.

---

# Day 2 — IAM Roles & Security Tools

---

# 6. IAM Roles

Some AWS services need to perform actions on our behalf.

For example, an EC2 instance may need permission to access:

- S3
- CloudFormation
- CloudWatch
- DynamoDB

Instead of storing long-term credentials on the EC2 instance, we can attach an **IAM role**.

### IAM Role Definition

An IAM role is an AWS identity that has permissions similar to an IAM user, but it is designed to be **assumed by trusted principals** and provides **temporary credentials**.

A role can be assumed by:

- AWS services
  - EC2
  - Lambda
  - ECS
  - etc.
- IAM users
- Another AWS account
- Federated identities

### Example

```text
EC2 Instance
     │
     ↓
IAM Role
     │
     ↓
IAM Policy
     │
     ↓
AWS Resources
```

The EC2 instance receives temporary credentials through the role.

---

# 7. IAM Security Tools

AWS provides security tools to understand IAM credentials and permissions.

## IAM Credential Report

**Account-level report**

The IAM Credential Report provides information about the credentials of IAM users in an AWS account.

It can provide information such as:

- User status
- Password status
- Access key status
- MFA status
- Credential age
- Credential usage

### Remember

```text
Credential Report → Account Level
```

---

## IAM Access Advisor

**User-level service access information**

IAM Access Advisor shows the AWS services that a user has permissions for and when those services were last accessed.

It can help identify permissions that may no longer be required.

### Remember

```text
Access Advisor → User Level
```

> Note: AWS documentation now refers to this capability as **Last accessed information** in several IAM contexts.

---

# Day 3 — IAM Advanced

---

# 8. AWS Organizations

AWS Organizations is used to **centrally manage multiple AWS accounts**.

An AWS Organization contains:

- Management account
- Member accounts
- Organizational Units (OUs)
- Root

### Management Account

The management account is used to manage the AWS Organization.

### Member Account

A member account belongs to the AWS Organization.

A member account can belong to an OU.

An AWS account can belong to **only one organization at a time**.

### Organizational Unit — OU

An OU is used to group AWS accounts.

Example:

```text
AWS Organization
│
├── Root
│
├── Production OU
│   ├── Account A
│   └── Account B
│
└── Development OU
    ├── Account C
    └── Account D
```

### Consolidated Billing

AWS Organizations can consolidate billing across multiple accounts into a single organization-level billing structure.

---

# 9. SCP — Service Control Policy

A **Service Control Policy (SCP)** is used in AWS Organizations to define the **maximum available permissions** for accounts or OUs.

Important:

> An SCP does **not grant permissions**.

It acts as an organizational **guardrail**.

The account must still have the required IAM permission.

### Example

Suppose:

```text
Production OU
└── SCP: Deny EC2
```

And:

```text
Account A
└── IAM Policy: Allow EC2
```

Even though Account A has an IAM policy allowing EC2:

```text
IAM → Allow EC2
SCP → Deny EC2
```

The EC2 action is denied.

To perform an action:

```text
IAM permission → Must Allow
SCP → Must Not Deny
```

### Important

SCPs do **not apply to the AWS Organizations management account**.

---

# 10. Easy Way to Remember AWS Organizations

```text
AWS Organizations
    ↓
Manages multiple AWS accounts

OU
    ↓
Groups AWS accounts

SCP
    ↓
Organizational permission ceiling / guardrail

IAM
    ↓
Actually grants permissions
```

---

# 11. AWS Organizations — Tag Policies

Tag Policies are used to standardize tags across an AWS Organization.

Tags use a:

```text
Key → Value
```

structure.

Examples:

```text
owner       → payment-team
environment → production
project     → ecommerce
```

Tag policies can help organizations:

- Standardize tagging
- Identify non-compliant tags
- Improve resource organization
- Support cost allocation and billing analysis

---

# 12. Advanced IAM Policies

## Resource-Based Policies

A resource-based policy is attached directly to an AWS resource.

It controls:

- Who can access the resource
- What actions they can perform

Examples of AWS resources that support resource-based policies include:

- S3 buckets
- SQS queues
- SNS topics
- KMS keys
- Secrets Manager secrets
- Lambda functions

### Example

A resource-based policy on an S3 bucket can specify which AWS principals can access the bucket.

---

# 13. `aws:PrincipalOrgID`

`aws:PrincipalOrgID` is a global condition key that can be used in policies to restrict access to principals that belong to a specific AWS Organization.

This is especially useful with **resource-based policies**.

Example concept:

```text
AWS Organization
       │
       ├── Account A
       ├── Account B
       └── Account C
              ↓
      Resource-based policy
              ↓
     Allow principals from
     this Organization
```

This can help prevent access from accounts outside the organization.

---

# Day 4 — IAM Roles & Advanced Permissions

---

# 14. IAM Roles vs Resource-Based Policies

## IAM Role

An IAM role is an AWS identity that:

- Has permissions
- Can be assumed by trusted principals
- Provides temporary credentials

## Resource-Based Policy

A resource-based policy is attached directly to a resource.

It defines:

- Who can access the resource
- What they can do

---

# 15. Cross-Account Access Example

Suppose:

```text
Account A
User
  │
  │ Upload object
  ↓
Account B
S3 Bucket
```

The user in Account A needs to upload an object to an S3 bucket in Account B.

The required permissions can involve:

### Account A

The user needs an identity-based permission allowing:

```text
s3:PutObject
```

### Account B

The S3 bucket can have a resource-based policy allowing the principal from Account A to perform:

```text
s3:PutObject
```

Conceptually:

```text
Account A
└── User
    └── IAM Policy
        └── Allow s3:PutObject
                    │
                    ↓
              Account B
                    │
                    ↓
              S3 Bucket
                    │
                    └── Resource Policy
                        └── Allow Account A
```

This is a common pattern for **cross-account access**.

---

# 16. IAM Permission Boundaries

An IAM Permission Boundary defines the **maximum permissions that an IAM user or role can have**.

It does not directly grant permissions.

Think of it as a permission ceiling.

### Example

Suppose a user has:

```text
IAM Policy
└── AdministratorAccess
```

But the permission boundary allows only:

```text
S3
```

Then the effective permissions are limited by the permission boundary.

Conceptually:

```text
Identity Policy
      +
Permission Boundary
      ↓
Effective Permissions
```

The user cannot use permissions outside the boundary.

---

# 17. IAM Permission Evaluation Logic

AWS evaluates multiple policy types when determining whether an API request is allowed.

Important concept:

### Explicit Deny

An explicit `Deny` overrides an `Allow`.

```text
Explicit Deny
     ↓
Overrides Allow
```

### General Concept

A simplified way to remember the evaluation process is:

```text
Request
   ↓
Is there an explicit Deny?
   │
   ├── YES → DENY
   │
   └── NO
        ↓
Check applicable policy types
        ↓
Determine whether an applicable Allow exists
        ↓
Final Decision
```

Policy types that can affect the decision include:

- Identity-based policies
- Resource-based policies
- Permission boundaries
- SCPs
- Session policies
- Resource control policies where applicable

> **Exam reminder:** Don't memorize the evaluation process as a simple linear sequence such as "SCP → resource policy → identity policy." AWS policy evaluation depends on the policy types involved and the type of principal/resource. The key rule to remember is that an applicable explicit `Deny` overrides an `Allow`.

---

# 18. AWS IAM Identity Center

Previously known as:

> **AWS Single Sign-On (AWS SSO)**

AWS IAM Identity Center provides centralized access to multiple:

- AWS accounts
- Business applications
- SAML 2.0 applications
- AWS applications
- Certain EC2 Windows instances and other supported resources

Instead of maintaining separate credentials for every AWS account, users can sign in through a centralized access portal.

### Identity Source

User identities can be stored in:

- IAM Identity Center's built-in identity store
- An external identity provider
- Microsoft Active Directory through supported integration

### Basic Concept

```text
                 IAM Identity Center
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
      AWS Account A  AWS Account B   Applications
```

The user signs in once and can access the resources they have been assigned.

---

# 19. IAM Identity Center Permission Sets

A **Permission Set** is a collection of one or more IAM policies that defines the permissions a user or group receives when accessing an AWS account through IAM Identity Center.

Conceptually:

```text
User / Group
     ↓
Permission Set
     ↓
IAM Policies
     ↓
AWS Account
```

Permission sets help centrally manage access across multiple AWS accounts.

---

# Day 5 — AWS Managed Directory Services

---

# 20. Active Directory

Microsoft Active Directory is a directory service/database used to store and manage objects such as:

- User accounts
- Computers
- Printers
- Groups
- Other directory objects

Active Directory commonly organizes objects into:

```text
Domain
   ↓
Organizational Units
   ↓
Objects
```

Multiple domains can form a **tree**, and multiple trees can form a **forest**.

---

# 21. AWS Directory Service

AWS provides multiple ways to use directory services.

The main options covered are:

1. AWS Managed Microsoft AD
2. AD Connector
3. Simple AD

---

# 22. AWS Managed Microsoft AD

AWS Managed Microsoft AD is a managed Microsoft Active Directory service hosted by AWS.

It provides an actual Microsoft Active Directory environment managed by AWS.

It can be connected with an on-premises Active Directory environment using a trust relationship.

This allows users to authenticate across connected environments.

### Concept

```text
On-Premises Active Directory
          ↕
     Trust Relationship
          ↕
AWS Managed Microsoft AD
```

MFA can also be integrated/enforced depending on the authentication architecture.

---

# 23. AD Connector

AD Connector acts as a **directory proxy**.

It does not store directory information itself.

Instead, authentication requests are forwarded to the existing on-premises Active Directory.

### Concept

```text
AWS Application
      ↓
AD Connector
      ↓
On-Premises Active Directory
```

This allows AWS applications and services to use the existing on-premises directory.

MFA can be supported depending on the authentication configuration.

---

# 24. AWS Simple AD

AWS Simple AD is a lower-cost directory service option based on **Samba**, an open-source implementation of Active Directory-compatible functionality.

It provides basic directory functionality without the full capabilities of AWS Managed Microsoft AD.

---

# Day 5 — AWS Control Tower

---

# 25. AWS Control Tower

> **Correction: The AWS service is called AWS Control Tower, not AWS Control Center.**

AWS Control Tower provides an easier way to set up and govern a **secure, multi-account AWS environment** based on AWS best practices.

It can help organizations establish and govern a landing zone across multiple AWS accounts.

### Main Capabilities

AWS Control Tower can help:

- Set up a multi-account environment
- Govern AWS accounts
- Apply preventive and detective controls
- Monitor compliance
- Detect policy violations
- Automate remediation for certain violations
- Provide a centralized governance dashboard

---

# 26. AWS Control Tower — Guardrails / Controls

Control Tower uses controls to enforce governance requirements.

Controls can broadly be understood as:

- Preventive controls
- Detective controls

## Preventive Controls

Preventive controls help **prevent non-compliant actions from occurring**.

They are commonly implemented using **SCPs**.

### Example

```text
Production OU
      ↓
Preventive Control
      ↓
Prevent creation of certain resources/actions
```

For example, an SCP can prevent an account from performing a particular action.

---

## Detective Controls

Detective controls are used to **detect non-compliant resources or configurations**.

They commonly use **AWS Config**.

### Example

An organization requires specific tags:

```text
Environment = Production
Owner = Team-A
```

AWS Config can detect resources that do not comply with the organization's requirements.

The detected compliance information can then be used by governance and notification workflows.

---

# 27. IAM Quick Revision

```text
IAM
├── Users
├── Groups
├── Roles
└── Policies
```

### Users

Represent people or applications that require AWS access.

### Groups

Collections of IAM users.

### Roles

Identities that can be assumed and provide temporary credentials.

### Policies

JSON documents that define permissions.

---

# 28. IAM Policy Keywords

| Keyword | Meaning |
|---|---|
| `Version` | Policy language version |
| `Id` | Policy identifier |
| `Statement` | Contains permission statements |
| `Sid` | Statement identifier |
| `Effect` | Allow or Deny |
| `Action` | AWS API action |
| `Principal` | Who the policy applies to |
| `Resource` | Resource the policy applies to |
| `Condition` | Conditions under which the statement applies |

---

# 29. IAM Advanced Quick Revision

```text
AWS Organizations
        ↓
Manage multiple AWS accounts

OU
        ↓
Group AWS accounts

SCP
        ↓
Permission ceiling / guardrail

IAM Policy
        ↓
Grants permissions

Permission Boundary
        ↓
Limits maximum permissions of
a user or role

Resource-Based Policy
        ↓
Controls access directly
on a resource

IAM Identity Center
        ↓
Centralized access to
multiple AWS accounts/apps

AWS Directory Service
        ↓
Directory / Active Directory integration

AWS Control Tower
        ↓
Multi-account governance
```

---

# 30. Important Exam Points

## IAM

- IAM is a **global service**.
- Users can belong to multiple groups.
- Groups cannot contain other groups.
- Policies are JSON documents.
- Follow least privilege.
- Prefer groups for managing permissions for multiple users.

## IAM Roles

- Roles provide temporary credentials.
- Roles are assumed by trusted principals.
- EC2, Lambda, ECS and other AWS services can assume roles.

## SCP

- SCPs do **not grant permissions**.
- SCPs define the maximum available permissions.
- An IAM Allow is still required.
- Explicit Deny overrides Allow.
- SCPs don't apply to the Organizations management account.

## Permission Boundaries

- Permission boundaries do not grant permissions.
- They define the maximum permissions a user or role can have.

## Resource-Based Policies

- Attached directly to resources.
- Specify who can access the resource and what they can do.
- Commonly important for cross-account access.

## IAM Identity Center

- Centralized access management.
- Previously called AWS SSO.
- Uses permission sets.
- Can provide access to multiple AWS accounts and applications.

## AWS Organizations

```text
Organization
├── Management Account
├── Root
├── OUs
└── Member Accounts
```

## Control Tower

```text
Control Tower
├── Multi-account setup
├── Governance
├── Preventive Controls
├── Detective Controls
├── AWS Config
└── SCP-based controls
```
