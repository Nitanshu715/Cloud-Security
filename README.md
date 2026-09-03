# 🔐 Cloud Security Lab

> **AWS practical laboratory repository for Cloud Security**

<p align="center">

![AWS](https://img.shields.io/badge/AWS-Cloud%20Security-orange?style=for-the-badge&logo=amazonaws&logoColor=white)
![IAM](https://img.shields.io/badge/AWS-IAM-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![VPC](https://img.shields.io/badge/Amazon-VPC-purple?style=for-the-badge&logo=amazonaws&logoColor=white)
![S3](https://img.shields.io/badge/Amazon-S3-red?style=for-the-badge&logo=amazons3&logoColor=white)
![Security](https://img.shields.io/badge/Focus-Cloud%20Security-success?style=for-the-badge)
![Status](https://img.shields.io/badge/Lab-Completed-success?style=for-the-badge)

</p>

<p align="center">
  <b>Identity • Access Control • Network Isolation • Resource Authorization</b><br>
  Practical AWS security experiments performed through the AWS Management Console.
</p>

---

## 📌 Overview

This repository contains the practical laboratory work completed for the **Cloud Security** subject using the **Amazon Web Services (AWS)** Management Console.

The lab is organized into three experiments that progressively explore different layers of cloud security:

```text
                    ☁️ CLOUD SECURITY
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
     🔐 IAM & Roles    🌐 VPC Security    🪣 S3 Authorization
          │                │                │
          ▼                ▼                ▼
     Identity &        Network         Resource-Based
     Permissions       Isolation          Policies
```

Together, the experiments demonstrate that cloud security is not a single configuration. It is a combination of:

- Identity management
- Authentication and authorization
- Role-based access
- Policy enforcement
- Network isolation
- Resource-level access control
- Least-privilege permissions
- Secure cloud resource management

---

## 🧪 Experiments

| # | Experiment | Primary Focus | Status |
|---|---|---|---|
| **01** | **IAM Roles & Access Control** — Configure and understand IAM roles, permissions, and role-based access | Identity & Access Management | ✅ Completed |
| **02** | **AWS VPC & VPC Peering** — Build isolated virtual networks and establish controlled connectivity between VPCs | Network Security & Isolation | ✅ Completed |
| **03** | **Using Resource-Based Policies to Secure an S3 Bucket** — Analyze IAM policies, assume roles, and use S3 resource-based authorization | Resource-Based Authorization | ✅ Completed |

---

# 🔐 Experiment 01 — IAM Roles & Access Control

**Focus:** Identity, permissions, role-based access, and AWS authorization.

📁 `experiment-1/`

The first experiment establishes the identity and access-control foundation of the Cloud Security laboratory.

It focuses on understanding how AWS identities receive permissions and how **IAM roles** can be used to provide controlled access to AWS resources.

### Covered

- AWS Identity and Access Management (IAM)
- IAM users
- IAM roles
- IAM policies
- Role-based access control
- Permissions
- Trust relationships
- Access authorization
- Least-privilege principles
- Switching / assuming IAM roles
- Verification of granted permissions

### Security Model

```text
IAM Identity
     │
     ▼
Permissions
     │
     ▼
AWS Resource
     │
     ▼
Authorized / Unauthorized
```

The experiment demonstrates the principle that authentication establishes an identity, while authorization determines what that identity is permitted to do.

### Key Security Concept

> **An IAM role separates the identity that requests access from the permission set used to perform the task.**

---

# 🌐 Experiment 02 — AWS VPC & VPC Peering

**Focus:** Cloud networking, network isolation, controlled connectivity, and secure communication between private networks.

📁 `experiment-2/`

The second experiment moves from identity security to **network-level security** by creating isolated Amazon Virtual Private Clouds and configuring VPC peering.

### Covered

- Amazon VPC
- Private IPv4 CIDR blocks
- VPC network boundaries
- Subnets
- Route tables
- VPC peering
- Peering routes
- Security groups
- Private IP communication
- Network isolation
- Controlled cross-VPC connectivity
- Connectivity testing

### Network Architecture

```text
┌───────────────────────────────┐
│           VPC-A               │
│                               │
│       Private CIDR            │
│             │                 │
│             ▼                 │
│       Private Subnet          │
│             │                 │
│             ▼                 │
│           EC2-A               │
└──────────────┬────────────────┘
               │
               │ VPC Peering
               │
               ▼
┌───────────────────────────────┐
│           VPC-B               │
│                               │
│       Separate CIDR           │
│             │                 │
│             ▼                 │
│       Private Subnet          │
│             │                 │
│             ▼                 │
│           EC2-B               │
└───────────────────────────────┘
```

The experiment demonstrates that creating a peering connection alone is not sufficient for communication.

Controlled connectivity requires the appropriate combination of:

```text
VPC Peering
     +
Route Table Entries
     +
Security Group Rules
     +
Network ACL Compatibility
     =
Private Connectivity
```

### Security Perspective

VPCs provide logical network boundaries that can be used to:

- Isolate resources
- Segment workloads
- Control traffic paths
- Reduce unnecessary exposure
- Limit the blast radius of security incidents
- Control communication between independent networks

### Key Security Concept

> **Network connectivity should be explicitly established and controlled rather than assumed.**

---

# 🪣 Experiment 03 — Using Resource-Based Policies to Secure an S3 Bucket

**Focus:** IAM authorization, role assumption, S3 object permissions, and resource-based policies.

📁 `experiment-3/`

This experiment demonstrates how AWS authorization can be controlled through multiple policy layers.

The practical work begins with a restricted IAM identity:

```text
devuser
   │
   ▼
DeveloperGroup
   │
   ▼
DeveloperGroupPolicy
```

The experiment then introduces:

```text
BucketsAccessRole
```

and demonstrates how assuming that role changes the effective permissions available to the operator.

Finally, the experiment analyzes an S3 bucket policy that grants the role access directly at the resource level.

---

## 🧩 Experiment 03 Access Model

```text
                         AWS ACCOUNT
                              │
                              ▼
                           devuser
                              │
                              ▼
                     DeveloperGroup
                              │
                              ▼
                   DeveloperGroupPolicy
                              │
                       Restricted Access
                              │
                       Assume IAM Role
                              ▼
                    BucketsAccessRole
                         /          \
                        /            \
                       ▼              ▼
              Role-Based Policy   S3 Bucket Policy
                       │              │
                       ▼              ▼
                    bucket1         bucket2
                       │              │
                  GetObject       PutObject
                       │              │
                       ▼              ▼
                   Download        Upload
```

---

## 🔑 Identity-Based Policies

Identity-based policies are attached to IAM identities such as:

- Users
- Groups
- Roles

In this experiment:

```text
devuser
   ↓
DeveloperGroup
   ↓
DeveloperGroupPolicy
```

The policy controls what permissions are available to the user through group membership.

The experiment demonstrates that permissions are **action-specific** and that access to one S3 operation does not automatically imply access to another.

For example:

```text
s3:CreateBucket
        ≠
s3:PutObject
```

A user can therefore have permission to create a bucket while still receiving:

```text
Access Denied
```

when attempting to upload an object.

---

## 🛡️ Resource-Based Policies

Resource-based policies are attached directly to AWS resources.

For Amazon S3, this is commonly implemented through an:

```text
S3 Bucket Policy
```

The experiment demonstrates a bucket policy that identifies:

```text
BucketsAccessRole
```

as the principal and grants selected actions.

The important authorization relationship is:

```text
bucket2
   │
   └── Bucket Policy
          │
          └── BucketsAccessRole
                   │
                   └── s3:PutObject
```

This explains why an object upload can succeed even when the role's separate `GrantBucket1Access` policy does not contain `s3:PutObject`.

---

## 🔄 IAM Role Assumption

The experiment switches the active identity from:

```text
devuser
```

to:

```text
BucketsAccessRole
```

The role contains policies such as:

```text
ListAllBucketsPolicy
GrantBucket1Access
```

### `GrantBucket1Access`

The experiment demonstrates selected permissions including:

```text
s3:GetObject
s3:ListObjects
s3:ListBucket
```

while:

```text
s3:PutObject
```

is not granted by this policy.

This makes the subsequent bucket-policy analysis important.

---

## 🤝 Trust Relationship

The experiment also examines the role's trust relationship.

Conceptually:

```text
Trust Policy
     │
     ▼
Who can assume the role?

Permission Policy
     │
     ▼
What can the role do?
```

The lab configuration allows the relevant trusted principal to assume:

```text
BucketsAccessRole
```

After role assumption, the active AWS authorization context changes.

---

## 📊 Practical Authorization Demonstration

The experiment observes the following access pattern:

```text
┌─────────────────────────────┬────────────────────┐
│ Operation                   │ Result             │
├─────────────────────────────┼────────────────────┤
│ EC2 access as devuser       │ Restricted         │
│ S3 object access as devuser │ Restricted         │
│ Create S3 bucket            │ Allowed            │
│ Upload object as devuser    │ Access Denied      │
│ Assume BucketsAccessRole    │ Allowed            │
│ Download from bucket1       │ Allowed            │
│ IAM access under role       │ Restricted         │
│ Upload to bucket2 as role   │ Allowed            │
└─────────────────────────────┴────────────────────┘
```

The exact final result of the `bucket3` challenge is documented in the experiment's own report and screenshots.

---

# 🧠 Core Security Concepts

The three experiments collectively reinforce the following cloud-security concepts.

### 🔐 Authentication vs Authorization

```text
Authentication
      │
      └── Who are you?

Authorization
      │
      └── What are you allowed to do?
```

### 🎯 Least Privilege

Permissions should be limited to what is required.

```text
Required Permission
        │
        ▼
Grant Only What Is Needed
        │
        ▼
Reduce Attack Surface
```

### 🛡️ Defense in Depth

Cloud security should not depend on a single control.

```text
Identity Security
       +
Network Security
       +
Resource Security
       +
Policy Enforcement
       =
Defense in Depth
```

The experiments demonstrate these layers through:

- IAM permissions
- IAM roles
- VPC isolation
- Route controls
- Security groups
- S3 resource policies

### 📜 Policy-Based Authorization

AWS policies define the actions a principal may perform on resources.

```text
Principal
   │
   ▼
Action
   │
   ▼
Resource
   │
   ▼
Allow / Deny
```

### 🚫 Explicit Deny

An explicit `Deny` takes precedence over an `Allow`.

```text
Allow
  +
Explicit Deny
  ↓
DENIED
```

This principle was demonstrated in the IAM security work through:

```text
s3:DeleteBucket
```

where an explicit Deny blocks bucket deletion despite broader S3 permissions.

### 🌐 Network Isolation

VPCs provide logical network boundaries.

```text
VPC
 │
 ├── CIDR
 ├── Subnets
 ├── Route Tables
 ├── Security Groups
 └── Network ACLs
```

### 🪣 Resource-Level Authorization

S3 demonstrates that permissions can be controlled directly on the resource.

```text
S3 Bucket
    │
    └── Bucket Policy
           │
           └── Principal + Actions + Resources
```

---

# ☁️ AWS Services Used

| Service | Security Purpose |
|---|---|
| **AWS IAM** | Identity and access management |
| **Amazon VPC** | Network isolation and private virtual networking |
| **Amazon EC2** | Compute access and private connectivity testing |
| **Amazon S3** | Object storage and resource-based authorization |
| **IAM Roles** | Temporary / role-based permissions |
| **S3 Bucket Policies** | Resource-based access control |

---

# 📂 Repository Structure

```text
cloud-security/
│
├── experiment-1/
│   └── README.md
│
├── experiment-2/
│   └── README.md
│
├── experiment-3/
│   └── README.md
│
└── README.md
```

Each experiment directory contains its own documentation and practical evidence.

---

# 📊 Lab Progress

```text
Experiment 01 — IAM Roles
████████████████████ 100% ✅

Experiment 02 — VPC & Peering
████████████████████ 100% ✅

Experiment 03 — S3 Resource-Based Policies
████████████████████ 100% ✅

Overall Cloud Security Lab
████████████████████ 100% ✅
```

---

# 📚 Security Learning Path

The repository follows a layered progression:

```text
01
IAM & Roles
   │
   │  Who can access?
   ▼
02
VPC & Peering
   │
   │  Where can resources communicate?
   ▼
03
S3 Resource Policies
   │
   │  What can a principal do to a resource?
   ▼
Cloud Security
```

This progression connects:

```text
Identity
   ↓
Network
   ↓
Resource
   ↓
Authorization
   ↓
Secure Cloud Architecture
```

---

# 📝 Experiment Documentation

| Experiment | Documentation |
|---|---|
| **01** | IAM Roles & Access Control |
| **02** | AWS VPC & VPC Peering |
| **03** | Using Resource-Based Policies to Secure an S3 Bucket |

Each experiment includes practical implementation details, observations, security analysis, results, conclusions, and supporting screenshots.

---

# 👨‍💻 Author

**Nitanshu Tak**

B.Tech — Computer Science Engineering  
Major: **Cloud Computing & Virtualization Technology**

---

## 🔐 Cloud Security Laboratory

```text
AWS
│
├── IAM
│   └── Identity & Access Control
│
├── VPC
│   └── Network Isolation
│
├── EC2
│   └── Compute & Connectivity
│
└── S3
    └── Resource-Based Authorization
```

> **Cloud security is strongest when identity, network, and resource-level controls work together.**

---

<p align="center">

### 🔐 Cloud Security Lab
**AWS • IAM • VPC • EC2 • S3 • Access Control • Network Security**

</p>

