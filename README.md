# 🔐 Cloud Security Lab

> **AWS practical laboratory repository for Cloud Security**

<p align="center">

![AWS](https://img.shields.io/badge/AWS-Cloud%20Security-orange?style=for-the-badge&logo=amazonaws&logoColor=white) ![IAM](https://img.shields.io/badge/AWS-IAM-blue?style=for-the-badge&logo=amazonaws&logoColor=white) ![VPC](https://img.shields.io/badge/Amazon-VPC-purple?style=for-the-badge&logo=amazonaws&logoColor=white) ![EC2](https://img.shields.io/badge/Amazon-EC2-red?style=for-the-badge&logo=amazonaws&logoColor=white) ![S3](https://img.shields.io/badge/Amazon-S3-yellow?style=for-the-badge&logo=amazons3&logoColor=white) ![KMS](https://img.shields.io/badge/AWS-KMS-darkblue?style=for-the-badge&logo=amazonaws&logoColor=white) ![Config](https://img.shields.io/badge/AWS-Config-orange?style=for-the-badge&logo=amazonaws&logoColor=white) ![Lambda](https://img.shields.io/badge/AWS-Lambda-purple?style=for-the-badge&logo=awslambda&logoColor=white) ![CloudWatch](https://img.shields.io/badge/Amazon-CloudWatch-blue?style=for-the-badge&logo=amazonaws&logoColor=white) ![Security](https://img.shields.io/badge/Focus-Cloud%20Security-success?style=for-the-badge) ![Status](https://img.shields.io/badge/Lab-Completed-success?style=for-the-badge)

</p>

<p align="center">
  <b>Identity • Network • Authorization • Encryption • Monitoring • Automated Remediation</b><br>
  Practical AWS security experiments performed through the AWS Management Console.
</p>

---

## 📌 Overview

This repository contains the practical laboratory work completed for the **Cloud Security** subject using **Amazon Web Services (AWS)**.

The seven experiments progress from identity and networking fundamentals to encryption, monitoring, and automated security remediation.

```text
                         ☁️ CLOUD SECURITY
                                │
                                ▼
                    ┌──────────────────────┐
                    │  Identity & Access    │
                    │  IAM • Roles • Policy │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Network Security   │
                    │ VPC • Peering • SG   │
                    │       • NACLs        │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Resource & Data      │
                    │ S3 • KMS • EBS       │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Monitoring & Response│
                    │ CloudTrail • CW      │
                    │ Config • Lambda      │
                    └──────────┬───────────┘
                               │
                               ▼
                       🛡️ DEFENSE IN DEPTH
```

---

# 🧪 Experiments

| # | Experiment | Primary Focus | AWS Services / Concepts | Status |
|---|---|---|---|---|
| **01** | **IAM Roles & Access Control** | Identity, permissions and authorization | IAM, Users, Roles, Policies | ✅ Completed |
| **02** | **AWS VPC & VPC Peering** | Network isolation and private connectivity | VPC, Subnets, Route Tables, Peering | ✅ Completed |
| **03** | **Using Resource-Based Policies to Secure an S3 Bucket** | Resource-level authorization | IAM, S3, Bucket Policies | ✅ Completed |
| **04** | **Securing VPC Resources by Using Security Groups** | Traffic filtering and secure administration | EC2, VPC, Security Groups, NACLs, Bastion, Session Manager | ✅ Completed |
| **05** | **Encrypting Data at Rest by Using AWS KMS** | Encryption and key management | KMS, S3, EBS, EC2, CloudTrail | ✅ Completed |
| **06** | **Monitoring and Alerting with CloudTrail and CloudWatch** | Logging, alerts and event monitoring | CloudTrail, CloudWatch, SNS, EventBridge, Logs Insights | ✅ Completed |
| **07** | **Remediating an Incident by Using AWS Config and Lambda** | Configuration monitoring and automated remediation | AWS Config, Lambda, EC2, IAM, CloudWatch Logs | ✅ Completed |

---

# 🔐 Experiment 01 — IAM Roles & Access Control

📁 `experiment-1/`

This experiment establishes the identity and access-control foundation of the laboratory. It covers IAM users, roles, policies, trust relationships, permissions, role assumption, and least-privilege authorization.

**Key concepts:** Authentication, Authorization, IAM Policies, IAM Roles, Trust Policies, Least Privilege.

---

# 🌐 Experiment 02 — AWS VPC & VPC Peering

📁 `experiment-2/`

This experiment focuses on network isolation and controlled private communication. Separate VPCs, subnets, route tables, and VPC peering are configured and connectivity is verified through private networking.

**Key concepts:** VPC, CIDR, Subnets, Route Tables, VPC Peering, Private Connectivity.

---

# 🪣 Experiment 03 — Using Resource-Based Policies to Secure an S3 Bucket

📁 `experiment-3/`

This experiment demonstrates how identity-based and resource-based policies work together to control S3 access. IAM users, groups, role assumption, bucket policies, object actions, and explicit Deny are examined.

**Key concepts:** IAM Authorization, Role Assumption, S3 Bucket Policies, Object Permissions, Explicit Deny.

---

# 🛡️ Experiment 04 — Securing VPC Resources by Using Security Groups

📁 `experiment-4/`

This experiment applies layered network security to EC2 resources. Security groups, security-group references, NACL rule precedence, private subnets, Bastion access, SSH agent forwarding, and Session Manager are used to secure and administer private resources.

**Key concepts:** Security Groups, NACLs, Bastion Host, SSH Agent Forwarding, Session Manager, Defense in Depth.

---

# 🔑 Experiment 05 — Encrypting Data at Rest by Using AWS KMS

📁 `experiment-5/`

This experiment protects stored data using a customer managed KMS key. S3 SSE-KMS encryption, authenticated object access, CloudTrail KMS events, encrypted EBS storage, and the effect of disabling and re-enabling a KMS key are demonstrated.

**Key concepts:** AWS KMS, Customer Managed Keys, SSE-KMS, EBS Encryption, CloudTrail, Encryption at Rest.

---

# 📊 Experiment 06 — Monitoring and Alerting with CloudTrail and CloudWatch

📁 `experiment-6/`

This experiment implements security monitoring and alerting using CloudTrail and CloudWatch. CloudTrail events are analyzed, SNS notifications are configured, EventBridge monitors security-group changes, CloudWatch metric filters and alarms detect failed console logins, and Logs Insights is used to query authentication events.

**Key concepts:** CloudTrail, CloudWatch Logs, SNS, EventBridge, Metric Filters, Alarms, Logs Insights.

---

# 🧩 Experiment 07 — Remediating an Incident by Using AWS Config and Lambda

📁 `experiment-7/`

This experiment demonstrates automated security remediation. AWS Config monitors EC2 security groups using a custom rule, and a pre-created Lambda function removes unwanted inbound permissions from the monitored security group. CloudWatch Logs are used to verify the remediation activity.

**Key concepts:** AWS Config, Custom Rules, Lambda, Security Group Monitoring, Automated Remediation, CloudWatch Logs.

---

# 🛡️ Cross-Experiment Security Model

The seven experiments build a layered cloud-security model:

```text
Identity
   ↓
Network
   ↓
Resource Authorization
   ↓
Traffic Filtering
   ↓
Encryption
   ↓
Monitoring & Alerting
   ↓
Automated Remediation
   ↓
Defense in Depth
```

---

# ☁️ AWS Services & Concepts Used

| Service / Concept | Role in Laboratory |
|---|---|
| **AWS IAM** | Identity and access management |
| **IAM Users & Roles** | Identity and temporary permission contexts |
| **IAM Policies** | Define allowed and denied actions |
| **Amazon VPC** | Network isolation |
| **Subnets** | Public and private network segmentation |
| **Route Tables** | Control traffic paths |
| **VPC Peering** | Private VPC-to-VPC connectivity |
| **Amazon EC2** | Compute and security testing |
| **Security Groups** | Stateful traffic filtering |
| **Network ACLs** | Subnet-level traffic filtering |
| **Amazon S3** | Object storage and resource authorization |
| **S3 Bucket Policies** | Resource-based access control |
| **Bastion Host** | Controlled private-resource administration |
| **AWS Systems Manager** | Managed EC2 access |
| **AWS KMS** | Encryption-key management |
| **Amazon EBS** | Encrypted block storage |
| **AWS CloudTrail** | AWS API and security event auditing |
| **Amazon CloudWatch** | Logs, metrics and alarms |
| **Amazon SNS** | Security notifications |
| **Amazon EventBridge** | Event-driven monitoring |
| **AWS Config** | Resource configuration monitoring |
| **AWS Lambda** | Automated security remediation |

---

# 📂 Repository Structure

```text
cloud-security/
│
├── experiment-1/
│   ├── README.md
│   └── report/
│
├── experiment-2/
│   ├── README.md
│   └── report/
│
├── experiment-3/
│   ├── README.md
│   └── report/
│
├── experiment-4/
│   ├── README.md
│   └── report/
│
├── experiment-5/
│   ├── README.md
│   └── report/
│
├── experiment-6/
│   ├── README.md
│   └── report/
│
├── experiment-7/
│   ├── README.md
│   └── report/
│
└── README.md
```

Each experiment directory contains its README, documentation/report, and supporting evidence where applicable.

---

# 📊 Laboratory Progress

```text
Experiment 01 — IAM Roles & Access Control
████████████████████ 100% ✅

Experiment 02 — AWS VPC & VPC Peering
████████████████████ 100% ✅

Experiment 03 — S3 Resource-Based Policies
████████████████████ 100% ✅

Experiment 04 — VPC Security Groups
████████████████████ 100% ✅

Experiment 05 — AWS KMS Data-at-Rest Encryption
████████████████████ 100% ✅

Experiment 06 — CloudTrail & CloudWatch Monitoring
████████████████████ 100% ✅

Experiment 07 — AWS Config & Lambda Remediation
████████████████████ 100% ✅

Overall Cloud Security Laboratory
████████████████████ 100% ✅
```

---

# 📚 Security Learning Path

```text
01
IAM & Roles
   │
   ▼
02
VPC & Peering
   │
   ▼
03
S3 Resource-Based Policies
   │
   ▼
04
Security Groups & Secure Administration
   │
   ▼
05
KMS & Encryption at Rest
   │
   ▼
06
CloudTrail & CloudWatch Monitoring
   │
   ▼
07
AWS Config & Lambda Remediation
   │
   ▼
🛡️ Layered Cloud Security
```

---

# 🎯 Learning Outcomes

After completing the laboratory, the following capabilities are demonstrated:

- Understand IAM users, roles, policies, and trust relationships.
- Differentiate authentication from authorization.
- Apply least-privilege access control.
- Build and inspect VPC network boundaries.
- Configure VPC peering and private routes.
- Use S3 resource-based policies.
- Configure EC2 Security Groups and NACLs.
- Secure private resources using Bastion and Session Manager.
- Apply encryption at rest using AWS KMS.
- Encrypt S3 and EBS resources.
- Inspect AWS API activity using CloudTrail.
- Monitor security events using CloudWatch and EventBridge.
- Create metric filters and alarms for security events.
- Query CloudTrail logs using CloudWatch Logs Insights.
- Monitor resource configurations using AWS Config.
- Automate security remediation using AWS Lambda.
- Apply defense-in-depth principles to cloud infrastructure.

---

# 🏁 Conclusion

This Cloud Security laboratory demonstrates how multiple AWS security mechanisms can work together to protect cloud resources.

```text
IAM
 ↓
VPC
 ↓
S3 Authorization
 ↓
Security Groups & NACLs
 ↓
KMS Encryption
 ↓
CloudTrail & CloudWatch
 ↓
AWS Config & Lambda
 ↓
🛡️ Defense in Depth
```

The seven experiments provide practical exposure to identity security, network isolation, resource authorization, traffic filtering, encryption, monitoring, alerting, configuration compliance, and automated remediation.

> **Cloud security is strongest when identity, network, resource, data, monitoring, and remediation controls work together.**

---

# 👨‍💻 Author

**Nitanshu Tak**

B.Tech — Computer Science Engineering  
Major: **Cloud Computing & Virtualization Technology**

<p align="center">

### 🔐 Cloud Security Laboratory

**AWS • IAM • VPC • EC2 • S3 • KMS • CloudTrail • CloudWatch • Config • Lambda**

</p>
