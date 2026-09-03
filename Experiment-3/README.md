# 🔐 AWS Cloud Security Lab — Experiment 3
## Using Resource-Based Policies to Secure an Amazon S3 Bucket

<p align="center">

![AWS](https://img.shields.io/badge/AWS-Cloud%20Security-orange?style=for-the-badge&logo=amazonaws&logoColor=white)
![IAM](https://img.shields.io/badge/AWS-IAM-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![S3](https://img.shields.io/badge/Amazon-S3-Resource%20Policies-red?style=for-the-badge&logo=amazons3&logoColor=white)
![Security](https://img.shields.io/badge/Focus-Authorization-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

</p>

<p align="center">
  <b>Identity-Based Policies × IAM Roles × Resource-Based Policies</b><br>
  A practical AWS security experiment for understanding fine-grained cloud authorization.
</p>

---

## 📌 Experiment Overview

This experiment explores how **AWS Identity and Access Management (IAM)** and **Amazon Simple Storage Service (Amazon S3)** work together to implement fine-grained access control.

Rather than treating AWS access as simply "allowed" or "denied", the experiment investigates **why** a particular action succeeds or fails by tracing permissions through:

- IAM users
- IAM groups
- Identity-based policies
- IAM roles
- Role trust relationships
- S3 bucket policies
- S3 bucket-level actions
- S3 object-level actions
- Resource-specific authorization

The practical workflow begins with a restricted IAM user named `devuser`. The user belongs to `DeveloperGroup`, which has `DeveloperGroupPolicy` attached. The policy provides selected permissions while intentionally omitting other actions.

The experiment then switches the same operator into `BucketsAccessRole`. This changes the active AWS identity and therefore changes the effective permissions available in the console.

The most important part of the experiment is the interaction between the role and an S3 bucket policy. `GrantBucket1Access` gives the assumed role selected access to `bucket1`, while a **resource-based bucket policy attached to `bucket2`** grants the role `s3:PutObject`, explaining why an upload succeeds even though the role's `GrantBucket1Access` policy does not contain `s3:PutObject`.

> **Core security lesson:** AWS authorization must be analyzed from both sides — the permissions associated with the requesting identity and the policies attached to the resource being accessed.

---

## 🎯 Aim

To understand and demonstrate **identity-based and resource-based access control in AWS** by analyzing IAM permissions, testing Amazon S3 access, assuming an IAM role, inspecting role policies and trust relationships, and examining an S3 bucket policy that grants resource-specific permissions.

---

## 🧠 Learning Objectives

By completing this experiment, the following concepts are demonstrated:

- Understanding AWS IAM as an authorization and identity-management service.
- Understanding IAM users and IAM groups.
- Understanding identity-based policies.
- Understanding resource-based policies.
- Understanding Amazon S3 bucket policies.
- Understanding IAM roles and temporary role-based access.
- Understanding role trust relationships.
- Distinguishing authentication from authorization.
- Distinguishing bucket-level and object-level S3 permissions.
- Understanding the difference between `s3:CreateBucket` and `s3:PutObject`.
- Understanding `s3:GetObject`, `s3:ListBucket`, and related permissions.
- Applying the principle of least privilege.
- Troubleshooting `AccessDenied` authorization failures.
- Understanding how multiple policy layers can explain an apparently unexpected successful action.

---

## ☁️ AWS Services Used

| Service / Component | Purpose |
|---|---|
| **AWS IAM** | Identity and access management |
| **IAM User — `devuser`** | Restricted starting identity |
| **IAM Group — `DeveloperGroup`** | Group through which permissions are assigned to `devuser` |
| **`DeveloperGroupPolicy`** | Identity-based policy attached to the group |
| **IAM Role — `BucketsAccessRole`** | Preconfigured role assumed during the experiment |
| **IAM Trust Relationship** | Defines who is trusted to assume the role |
| **Amazon S3** | Object storage and authorization testing |
| **S3 Bucket Policy** | Resource-based policy controlling access to a bucket |
| **AWS Management Console** | Interface used to perform and verify the lab |

---

## 🏗️ Lab Environment

The AWS lab environment is preconfigured with:

```text
IAM User
└── devuser
      │
      ▼
IAM Group
└── DeveloperGroup
      │
      ▼
Identity-Based Policy
└── DeveloperGroupPolicy


Preconfigured S3 Resources
├── bucket1
│   └── Image2.jpg
│
├── bucket2
│   └── Image1.jpg
│
└── bucket3


Preconfigured IAM Role
└── BucketsAccessRole
      ├── ListAllBucketsPolicy
      └── GrantBucket1Access
```

The exact S3 bucket names are unique to the lab environment and contain identifiers such as `bucket1`, `bucket2`, and `bucket3`.

---

## 🔑 Authorization Model

The experiment demonstrates two major policy categories.

### 1. Identity-Based Policy

An identity-based policy is attached to an IAM identity such as:

- User
- Group
- Role

In this experiment:

```text
devuser
   │
   ▼
DeveloperGroup
   │
   ▼
DeveloperGroupPolicy
```

Because `devuser` belongs to `DeveloperGroup`, the permissions defined in `DeveloperGroupPolicy` contribute to the user's effective permissions.

---

### 2. Resource-Based Policy

A resource-based policy is attached directly to an AWS resource.

For Amazon S3, this is commonly an **S3 bucket policy**.

The simplified relationship is:

```text
Principal
   │
   │ requests action
   ▼
AWS Resource
   │
   └── Resource-Based Policy
```

In this experiment, `bucket2` contains a policy that directly identifies:

```text
BucketsAccessRole
```

as a principal and grants selected S3 actions.

---

## 🔄 Identity-Based vs Resource-Based Policies

| Feature | Identity-Based Policy | Resource-Based Policy |
|---|---|---|
| Attached to | User, Group, Role | AWS Resource |
| Main purpose | Define what an identity can do | Define who can access a resource and what they can do |
| Example in this lab | `DeveloperGroupPolicy` | `bucket2` bucket policy |
| Principal specified | Implicit through attached identity | Explicitly specified in resource policy |
| Example permission | S3 bucket operations | `s3:PutObject` on `bucket2` |

---

## 👤 Authentication vs Authorization

### Authentication

Authentication answers:

> **"Who are you?"**

In this lab:

```text
IAM Username → devuser
```

The user authenticates using the credentials supplied by the lab environment.

### Authorization

Authorization answers:

> **"What are you allowed to do?"**

For example:

```text
devuser
   │
   ├── EC2 actions → Restricted
   ├── IAM account summary → Restricted
   ├── S3 bucket creation → Allowed
   └── S3 object upload → Denied
```

Therefore:

> Successful authentication does **not** mean unrestricted authorization.

---

# 🧪 Practical Procedure

## Task 1 — Access the AWS Console as `devuser`

The lab environment was started and the AWS Details panel was used to obtain the IAM login information.

The AWS console was accessed using:

```text
IAM User: devuser
```

The lab environment's supplied IAM login URL and password were used.

### Security Observation

The experiment intentionally begins with a restricted identity instead of an administrator account.

This allows authorization behavior to be observed directly.

---

## Task 2 — Test Read-Level Access

### Amazon EC2

The EC2 console was opened while authenticated as `devuser`.

The following restrictions were observed:

- EC2 Dashboard produced API errors.
- Instances could not be viewed normally.
- Launching an instance showed unauthorized-operation messages.
- The Key pair name selection could not be retrieved.

This demonstrates that `devuser` does not have the required EC2 permissions.

### Amazon S3

The S3 console was then opened.

The preconfigured buckets were visible, but the Access column indicated insufficient permissions.

This demonstrates that:

```text
Console visibility ≠ unrestricted resource access
```

---

## Task 3 — Analyze `DeveloperGroupPolicy`

The IAM console was opened.

The following path was used:

```text
IAM
  ↓
User groups
  ↓
DeveloperGroup
```

The Users tab confirmed:

```text
devuser ∈ DeveloperGroup
```

The Permissions tab confirmed:

```text
DeveloperGroup
      │
      └── DeveloperGroupPolicy
```

Because the policy is attached to the group, it applies to users who belong to that group.

### Important Policy Observations

The policy was reviewed to understand the previously observed behavior.

Important observations included:

- No Amazon EC2 actions were granted.
- `iam:GetAccountSummary` was not granted.
- Some S3 bucket-level actions were allowed.
- Object-related S3 actions were not granted.

The policy was copied and saved locally as:

```text
DeveloperGroupPolicy.json
```

---

## Task 4 — Test Write-Level S3 Access

A new S3 bucket was created using the lab's required naming pattern.

The lab specifies:

```text
Region:
US East (N. Virginia)

Region Code:
us-east-1
```

The bucket was successfully created.

### Object Upload Test

The following file was selected:

```text
DeveloperGroupPolicy.json
```

An upload was attempted.

The operation failed with:

```text
Upload failed
```

and:

```text
Access Denied
```

### Why?

Creating an S3 bucket and uploading an S3 object are different API actions.

Conceptually:

```text
Create Bucket
     │
     ▼
s3:CreateBucket
     │
     └── Allowed


Upload Object
     │
     ▼
s3:PutObject
     │
     └── Not Granted
              │
              ▼
         Access Denied
```

This is a direct demonstration of fine-grained permissions.

---

## Task 5 — Test Object Access as `devuser`

The following object access operations were attempted:

```text
bucket1
└── Image2.jpg
```

Download attempt:

```text
AccessDenied
```

Then:

```text
bucket2
└── Image1.jpg
```

Download attempt:

```text
AccessDenied
```

### Security Interpretation

The current identity did not possess the object-level permissions required to retrieve those objects.

This reinforces the distinction between:

```text
Bucket-level permission
        ≠
Object-level permission
```

---

# 🔄 Task 6 — Assume `BucketsAccessRole`

The AWS identity menu was opened.

The following role was assumed:

```text
BucketsAccessRole
```

The lab Account ID was entered as required.

After switching roles, the identity shown in the upper-right corner changed from:

```text
devuser
```

to:

```text
BucketsAccessRole
```

This confirmed that the role was active.

---

## 🪣 Task 7 — Download from `bucket1` Using the Role

While `BucketsAccessRole` was active:

```text
bucket1
└── Image2.jpg
```

was opened.

The object was downloaded successfully.

### Interpretation

The successful download demonstrates that the active role has permission to perform:

```text
s3:GetObject
```

against the permitted `bucket1` resources.

---

# 🚫 Task 8 — Test IAM Access While the Role Is Active

The IAM console was opened while `BucketsAccessRole` was active.

The User groups page produced an authorization error.

The important missing permission was:

```text
iam:ListGroups
```

The role was then switched back to:

```text
devuser
```

After switching back, access to User groups was restored.

### Security Lesson

Changing roles changes the effective authorization context.

```text
devuser
   │
   └── Permission Set A

        ↓ Assume Role

BucketsAccessRole
   │
   └── Permission Set B
```

The same operator can therefore have different capabilities depending on the active IAM principal.

---

# 🔍 Task 9 — Analyze `BucketsAccessRole`

The IAM Roles section was opened and:

```text
BucketsAccessRole
```

was inspected.

Two important policies were analyzed.

---

## `ListAllBucketsPolicy`

This policy grants:

```text
s3:ListAllMyBuckets
```

This allows the role to list S3 buckets.

---

## `GrantBucket1Access`

This policy grants selected permissions related to `bucket1`.

| Action | Meaning |
|---|---|
| `s3:GetObject` | Retrieve objects |
| `s3:ListObjects` | List objects |
| `s3:ListBucket` | List bucket contents |
| `s3:PutObject` | **Not granted** |

The important observation is:

```text
GrantBucket1Access
        │
        ├── GetObject       ✅
        ├── ListObjects     ✅
        ├── ListBucket      ✅
        └── PutObject       ❌
```

The policy is scoped to the appropriate bucket/object resources rather than granting unrestricted S3 access.

A local copy was saved as:

```text
GrantBucket1Access.json
```

---

# 🤝 Task 10 — Analyze the Trust Relationship

The Trust relationships tab of:

```text
BucketsAccessRole
```

was inspected.

The lab configuration identifies:

```text
devuser
```

as a trusted entity capable of assuming the role.

### Trust vs Permission

These are two different concepts:

```text
Trust Relationship
        │
        ▼
Who can assume the role?


Permission Policy
        │
        ▼
What can the role do?
```

AWS Security Token Service (STS) provides temporary credentials when a trusted principal assumes the role.

---

# 📤 Task 11 — Upload `Image2.jpg` to `bucket2`

The role was assumed again.

The object:

```text
Image2.jpg
```

was uploaded to:

```text
bucket2
```

The upload succeeded.

At first, this appears confusing because:

```text
GrantBucket1Access
```

does **not** grant:

```text
s3:PutObject
```

So why did the upload succeed?

The answer is the **resource-based bucket policy**.

---

# 🧩 Task 12 — Analyze the `bucket2` Resource-Based Policy

The Permissions tab of `bucket2` was opened.

The bucket policy contained two important statements.

### `S3Write`

```text
Principal:
BucketsAccessRole

Actions:
s3:GetObject
s3:PutObject
```

### `ListBucket`

```text
Principal:
BucketsAccessRole

Action:
s3:ListBucket
```

Therefore:

```text
BucketsAccessRole
        │
        │ role policy
        ▼
     bucket1
   GetObject
        │
        │
        │ resource-based bucket policy
        ▼
     bucket2
   PutObject
```

### The Key Finding

`GrantBucket1Access` does not contain:

```text
s3:PutObject
```

However, the resource-based policy attached to `bucket2` does contain:

```text
s3:PutObject
```

for:

```text
BucketsAccessRole
```

Therefore the upload succeeds.

---

# 🧠 Policy Interaction

The authorization model observed during the experiment can be represented as:

```text
                     devuser
                        │
                        ▼
                DeveloperGroup
                        │
                        ▼
             DeveloperGroupPolicy
                        │
             Restricted Permissions
                        │
                        │
                 Assume Role
                        │
                        ▼
              BucketsAccessRole
                   /          \
                  /            \
                 ▼              ▼
       Role-Based Policies   Resource Policy
              │                   │
              ▼                   ▼
           bucket1             bucket2
              │                   │
          GetObject           PutObject
              │                   │
              ▼                   ▼
           Download              Upload
```

This is the central security architecture demonstrated by the experiment.

---

# 🏆 Challenge — Upload `Image2.jpg` to `bucket3`

The final challenge requires determining how the object can be uploaded to `bucket3`.

### Initial Attempt

The role was unassumed.

The active identity was:

```text
devuser
```

An upload of:

```text
Image2.jpg
```

to:

```text
bucket3
```

was attempted.

The upload failed.

The bucket policy could not be viewed under the restricted identity.

### Role-Based Investigation

`BucketsAccessRole` was then assumed.

The bucket3 Permissions section was inspected to determine whether a resource-based policy provided the required access.

The final result should be documented from the actual lab environment and supported by the screenshot showing the resulting object state.

> The exact bucket3 policy is environment-specific and should be recorded from the lab rather than guessed.

---

# 🔐 Security Analysis

## 1. Principle of Least Privilege

The experiment demonstrates why users should receive only the permissions required for their responsibilities.

Instead of granting:

```text
Full Administrator Access
```

the lab provides selected actions.

This reduces:

- Accidental changes
- Unauthorized access
- Destructive operations
- Security exposure
- Blast radius

---

## 2. Fine-Grained Authorization

AWS permissions can be controlled at the level of:

```text
Service
   ↓
Action
   ↓
Resource
```

For example:

```text
Amazon S3
    ↓
s3:GetObject
    ↓
Specific bucket / object
```

This enables precise access control.

---

## 3. Role-Based Access

IAM roles provide a mechanism for obtaining a different permission set without permanently modifying the original user's permissions.

In the experiment:

```text
devuser
   ↓
Assume
   ↓
BucketsAccessRole
   ↓
Temporary role-based access
```

---

## 4. Resource-Based Access

The bucket2 policy demonstrates that authorization can also be defined directly on the resource.

```text
bucket2
   │
   └── Bucket Policy
          │
          └── BucketsAccessRole
                   │
                   └── s3:PutObject
```

This is especially important when troubleshooting unexpected access.

---

## 5. Authorization Troubleshooting

When an AWS action succeeds or fails unexpectedly, inspect:

```text
1. Current identity
2. Identity-based policies
3. Group policies
4. Role policies
5. Trust relationship
6. Resource-based policy
7. Requested action
8. Target resource
```

Looking at only one policy may produce an incomplete explanation.

---

# 📊 Observations

| Operation | Result | Security Meaning |
|---|---|---|
| EC2 Dashboard as `devuser` | Restricted / API errors | EC2 permissions unavailable |
| EC2 Instances | Unauthorized | Required EC2 actions absent |
| S3 bucket list | Insufficient permissions | S3 access restricted |
| Create new S3 bucket | Successful | Bucket creation permitted |
| Upload `DeveloperGroupPolicy.json` | Access Denied | `s3:PutObject` not available |
| Download `Image2.jpg` as `devuser` | AccessDenied | Object access unavailable |
| Download `Image2.jpg` as role | Successful | Role grants object read access |
| IAM User groups as role | Unauthorized | `iam:ListGroups` unavailable |
| Upload `Image2.jpg` to `bucket2` as role | Successful | Resource-based policy grants `PutObject` |
| Upload to `bucket3` as `devuser` | Failed | Restricted identity cannot perform action |
| Inspect `bucket3` policy as role | Available for analysis | Different authorization context |

---

# 🧪 Important AWS Actions Demonstrated

```text
s3:CreateBucket
s3:GetObject
s3:PutObject
s3:ListBucket
s3:ListObjects
s3:ListAllMyBuckets
iam:GetAccountSummary
iam:ListGroups
```

The experiment demonstrates that permissions are **action-specific**.

For example:

```text
s3:CreateBucket
```

does not automatically imply:

```text
s3:PutObject
```

---

# 📚 Key Concepts Learned

- AWS Identity and Access Management
- IAM Users
- IAM Groups
- IAM Policies
- Identity-Based Policies
- Resource-Based Policies
- S3 Bucket Policies
- IAM Roles
- Role Assumption
- Trust Relationships
- AWS STS
- Authentication
- Authorization
- Least Privilege
- Fine-Grained Access Control
- S3 Bucket Permissions
- S3 Object Permissions
- `s3:GetObject`
- `s3:PutObject`
- `s3:ListBucket`
- `s3:ListObjects`
- AccessDenied troubleshooting
- Policy interaction

---

# 🧾 Files Produced During the Experiment

```text
DeveloperGroupPolicy.json
GrantBucket1Access.json
```

These files contain policy definitions reviewed during the experiment and can be retained as supporting evidence.

---

# 🏁 Result

The experiment successfully demonstrated the practical use of:

```text
IAM Identity-Based Policies
            +
IAM Role-Based Access
            +
S3 Resource-Based Policies
            =
Fine-Grained Cloud Authorization
```

The restricted `devuser` identity was unable to perform several EC2, IAM, and S3 object operations.

After assuming `BucketsAccessRole`, the operator obtained a different permission set and successfully accessed objects in `bucket1`.

The successful upload to `bucket2` demonstrated the importance of **resource-based policies**, because the bucket policy explicitly granted `BucketsAccessRole` the `s3:PutObject` permission even though that permission was absent from `GrantBucket1Access`.

---

# 🧠 Final Takeaway

> **Cloud security is not just about who can log in — it is about exactly what that identity is allowed to do, on which resource, through which policy, and under which authorization context.**

The experiment demonstrates that secure cloud architectures rely on:

```text
Identity
   +
Authentication
   +
Authorization
   +
Least Privilege
   +
Role-Based Access
   +
Resource-Based Policies
   +
Resource-Level Permissions
   =
Strong Cloud Security
```

---

## 📌 Experiment Status

| Parameter | Value |
|---|---|
| **Experiment** | 3 |
| **Lab** | Lab 3.1 — Using Resource-Based Policies to Secure an S3 Bucket |
| **Subject** | Cloud Security |
| **Platform** | AWS |
| **Primary Services** | IAM + Amazon S3 |
| **IAM User** | `devuser` |
| **IAM Group** | `DeveloperGroup` |
| **IAM Role** | `BucketsAccessRole` |
| **Main Security Topic** | Identity-Based + Resource-Based Authorization |
| **Status** | ✅ Completed |

---

<p align="center">
  <b>🔐 AWS Cloud Security • IAM • S3 • Resource-Based Authorization</b><br>
  <sub>Experiment 3 — Practical Cloud Security Laboratory</sub>
<p>
