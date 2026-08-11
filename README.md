# 🔐 Cloud Security Lab

 

> **AWS practical laboratory repository for Cloud Security**

>

> A structured collection of experiments covering AWS networking, identity, access control, authorization, and security policies.

 

---

 

## 📌 Overview

 

This repository contains the practical work completed for the **Cloud Security** subject using the AWS Management Console.

 

The experiments focus on understanding how cloud resources are isolated, how identities are authenticated and authorized, and how AWS IAM policies control access to cloud services.

 

---

 
## 🧪 Experiments

| # | Experiment | Status |
|---|---|---|
| 01 | **AWS VPC Creation** — Create and configure a Virtual Private Cloud | ✅ Completed |
| 02 | **IAM Security & Policy Enforcement** — Configure users, permissions, and explicit Deny policies | ✅ Completed |

 

---

 

## 🔐 Experiment 01 — AWS VPC

 

**Focus:** Cloud networking and network isolation

 

### Covered

- Amazon VPC

- IPv4 CIDR blocks

- Virtual private networking

- AWS network isolation

- VPC configuration and verification

 

📁 `experiment-1/`

 

---

 

## 🛡️ Experiment 02 — IAM Security

 

**Focus:** Identity, authorization, and policy-based access control

 

### Covered

- IAM users

- EC2 permissions

- S3 permissions

- AWS managed policies

- Custom IAM policies

- JSON policy structure

- Explicit Deny

- `s3:DeleteBucket`

- Access verification

- Least-privilege principles

 

### Practical Security Demonstration

 

Two users were configured with service-specific permissions:

 

```text

User 1

└── AmazonEC2FullAccess

    └── EC2 access verified

 

User 2

└── AmazonS3FullAccess

    └── S3 access verified

         ↓

    Custom policy added

         ↓

    Explicit Deny: s3:DeleteBucket

         ↓

    Bucket deletion blocked

```

 

This demonstrates an important AWS IAM rule:

 

> **An explicit Deny overrides an Allow.**

 

📁 `experiment-2/`

 

---

 

## ☁️ AWS Services Used

 

| Service | Purpose |

|---|---|

| **Amazon VPC** | Virtual networking and resource isolation |

| **AWS IAM** | Identity and access management |

| **Amazon EC2** | Compute-resource authorization testing |

| **Amazon S3** | Object-storage authorization testing |

 

---

 

## 🧠 Security Concepts

 

The practical work reinforces:

 

- Authentication vs. authorization

- Identity-based access control

- IAM policy evaluation

- Allow vs. Explicit Deny

- Least privilege

- Permission boundaries and policy scope

- Service-specific authorization

- Secure cloud resource management

- Network isolation

 

---

 

## 📂 Repository Structure

 

```text

cloud-security/

│

├── experiment-1/

│   └── README.md

│

├── experiment-2/

│   └── README.md

│

└── README.md

```

 

Each experiment directory contains its own detailed documentation and supporting material.

 

---

 

## 📊 Lab Progress

 

```text

Experiment 01   ████████████████████  100%

Experiment 02   ████████████████████  100%

```
 

## 👨‍💻 Author

 

**Nitanshu Tak**

 

B.Tech — Computer Science Engineering  

Major: Cloud Computing & Virtualization Technology

 

---

 

> 🔐 **Cloud Security Lab · AWS · Practical Learning**
