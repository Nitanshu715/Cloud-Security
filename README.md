# 🔐 Cloud Security Lab


> **AWS practical laboratory repository for Cloud Security**


<p align="center">

![AWS](https://img.shields.io/badge/AWS-Cloud%20Security-orange?style=for-the-badge&logo=amazonaws&logoColor=white) ![IAM](https://img.shields.io/badge/AWS-IAM-blue?style=for-the-badge&logo=amazonaws&logoColor=white) ![VPC](https://img.shields.io/badge/Amazon-VPC-purple?style=for-the-badge&logo=amazonaws&logoColor=white) ![EC2](https://img.shields.io/badge/Amazon-EC2-red?style=for-the-badge&logo=amazonaws&logoColor=white) ![S3](https://img.shields.io/badge/Amazon-S3-yellow?style=for-the-badge&logo=amazons3&logoColor=white) ![NACL](https://img.shields.io/badge/VPC-NACLs-darkgreen?style=for-the-badge) ![SSM](https://img.shields.io/badge/AWS-Session%20Manager-success?style=for-the-badge&logo=amazonaws&logoColor=white) ![Security](https://img.shields.io/badge/Focus-Cloud%20Security-success?style=for-the-badge) ![Status](https://img.shields.io/badge/Lab-Completed-success?style=for-the-badge)

</p>


<p align="center">

  <b>Identity • Network • Resource Authorization • Traffic Filtering • Secure Administration</b><br>

  Practical AWS security experiments performed through the AWS Management Console.

</p>


---


## 📌 Overview


This repository contains the practical laboratory work completed for the **Cloud Security** subject using **Amazon Web Services (AWS)**.


The experiments progressively move through multiple layers of cloud security:


```text

                         ☁️ CLOUD SECURITY

                                │

          ┌─────────────────────┼─────────────────────┐

          │                     │                     │

          ▼                     ▼                     ▼

   🔐 IDENTITY             🌐 NETWORK             🪣 RESOURCE

     SECURITY               SECURITY             AUTHORIZATION

          │                     │                     │

          ▼                     ▼                     ▼

     IAM & Roles          VPC & Peering       S3 Resource Policies

          │                     │                     │

          └─────────────────────┬─────────────────────┘

                                │

                                ▼

                       🛡️ DEFENSE IN DEPTH

                                │

                                ▼

                    🔒 VPC TRAFFIC SECURITY

                                │

                                ▼

                Security Groups • NACLs • Bastion

                                │

                                ▼

                     AWS Session Manager

```


The complete laboratory now covers four experiments:


```text

01 → IAM Roles & Access Control

02 → AWS VPC & VPC Peering

03 → S3 Resource-Based Policies

04 → VPC Security Groups & Secure Administration

```


Together, they demonstrate that cloud security is not a single control. It is a layered model involving:


- Identity management

- Authentication and authorization

- IAM roles and policies

- Least-privilege permissions

- Network isolation

- Routing and private connectivity

- Resource-based authorization

- Security groups

- Security-group references

- Network ACLs

- Bastion-based administration

- SSH agent forwarding

- Session Manager

- Defense in depth


---


# 🧪 Experiments


| # | Experiment | Primary Focus | AWS Services / Concepts | Status |

|---|---|---|---|---|

| **01** | **IAM Roles & Access Control** | Identity, permissions, roles, authorization | IAM, Policies, Roles | ✅ Completed |

| **02** | **AWS VPC & VPC Peering** | Network isolation and controlled connectivity | VPC, Subnets, Route Tables, Peering | ✅ Completed |

| **03** | **Using Resource-Based Policies to Secure an S3 Bucket** | Resource-level authorization and role assumption | IAM, S3, Bucket Policies | ✅ Completed |

| **04** | **Securing VPC Resources by Using Security Groups** | Traffic filtering and secure private-resource access | EC2, VPC, Security Groups, NACLs, Bastion, Session Manager | ✅ Completed |


---


# 🔐 Experiment 01 — IAM Roles & Access Control


**Focus:** Identity, permissions, role-based access, trust relationships, and AWS authorization.


📁 `experiment-1/`


The first experiment establishes the **identity and access-control foundation** of the Cloud Security laboratory.


It focuses on how AWS identities receive permissions and how **IAM roles** can be used to provide controlled access to AWS resources.


## Covered


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


## Security Model


```text

                 IAM Identity

                      │

                      ▼

                IAM Permissions

                      │

                      ▼

                AWS Resource

                      │

             ┌────────┴────────┐

             ▼                 ▼

          Allowed            Denied

```


The experiment establishes the distinction between:


```text

Authentication

     │

     └── Who are you?


Authorization

     │

     └── What are you allowed to do?

```


## Key Security Concept


> **An IAM role separates the identity requesting access from the permission set used to perform a task.**


---


# 🌐 Experiment 02 — AWS VPC & VPC Peering


**Focus:** Cloud networking, network isolation, private connectivity, routing, and controlled communication between VPCs.


📁 `experiment-2/`


The second experiment moves from identity security to **network-level security**.


It demonstrates how separate virtual networks can be isolated and then connected through an explicitly configured VPC peering relationship.


## Covered


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


## Network Architecture


```text

┌───────────────────────────────┐

│            VPC-A              │

│                               │

│        Private CIDR           │

│              │                │

│              ▼                │

│       Private Subnet          │

│              │                │

│              ▼                │

│            EC2-A              │

└──────────────┬────────────────┘

               │

               │ VPC Peering

               │

               ▼

┌───────────────────────────────┐

│            VPC-B              │

│                               │

│        Separate CIDR          │

│              │                │

│              ▼                │

│       Private Subnet          │

│              │                │

│              ▼                │

│            EC2-B              │

└───────────────────────────────┘

```


Creating the peering connection alone does not automatically make application traffic work.


Controlled connectivity requires:


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


## Security Perspective


VPC boundaries can be used to:


- Isolate resources

- Segment workloads

- Control traffic paths

- Reduce unnecessary exposure

- Limit the blast radius of incidents

- Control communication between independent networks


## Key Security Concept


> **Network connectivity should be explicitly established and controlled rather than assumed.**


---


# 🪣 Experiment 03 — Using Resource-Based Policies to Secure an S3 Bucket


**Focus:** IAM authorization, role assumption, S3 object permissions, and resource-based policies.


📁 `experiment-3/`


This experiment demonstrates how AWS authorization can be controlled through multiple policy layers.


The practical begins with a restricted IAM identity:


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


Finally, the experiment analyzes an S3 bucket policy that grants selected access directly at the resource level.


## Access Model


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

                              ▼

                       Restricted Access

                              │

                              │ Assume Role

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

                   GetObject        PutObject

                        │              │

                        ▼              ▼

                    Download        Upload

```


## Identity-Based Policies


Identity-based policies are attached to IAM identities such as:


- Users

- Groups

- Roles


The experiment uses:


```text

devuser

   ↓

DeveloperGroup

   ↓

DeveloperGroupPolicy

```


The practical demonstrates that permissions are **action-specific**.


For example:


```text

s3:CreateBucket

      ≠

s3:PutObject

```


Permission for one S3 operation does not automatically grant permission for another.


## Resource-Based Policies


For Amazon S3, resource-based authorization is commonly implemented using:


```text

S3 Bucket Policy

```


Conceptually:


```text

bucket2

   │

   └── Bucket Policy

           │

           └── BucketsAccessRole

                    │

                    └── s3:PutObject

```


This demonstrates why an S3 object upload can be authorized through the bucket policy even when a separate identity policy does not provide that same action.


## IAM Role Assumption


The experiment changes the active authorization context from:


```text

devuser

```


to:


```text

BucketsAccessRole

```


The role is evaluated through its trust relationship and permission policies.


```text

Trust Policy

     │

     ▼

Who can assume the role?

     │

     ▼

BucketsAccessRole

     │

     ▼

Permission Policies

     │

     ▼

What can the role do?

```


## Explicit Deny


The broader Cloud Security laboratory also demonstrates the precedence of explicit Deny.


```text

Allow

  +

Explicit Deny

  ↓

DENIED

```


For example:


```text

s3:DeleteBucket

```


can be explicitly denied even when broader S3 permissions exist.


## Key Security Concept


> **AWS authorization can be influenced by identity-based policies, role permissions, trust relationships, and resource-based policies working together.**


---


# 🛡️ Experiment 04 — Securing VPC Resources by Using Security Groups


**Focus:** EC2 traffic filtering, least privilege, security-group references, NACLs, Bastion access, SSH agent forwarding, and Session Manager.


📁 `experiment-4/`


The fourth experiment extends the VPC security model from basic network connectivity into **layered traffic control and secure administration of private resources**.


The environment contains:


```text

LabVPC

10.0.0.0/16

```


with:


```text

PublicSubnetA

PublicSubnetB

PrivateSubnet

```


and three Linux EC2 instances:


```text

ProxyServer1

ProxyServer2

AppServer

```


The public proxy servers forward HTTP traffic toward the private AppServer.


---


## 🏗️ Experiment 04 Architecture


```text

                              🌍 Internet

                                  │

                                  ▼

                         ┌─────────────────┐

                         │ Internet Gateway│

                         └────────┬────────┘

                                  │

                     ┌────────────┴────────────┐

                     │        LabVPC           │

                     │      10.0.0.0/16        │

                     │                         │

             ┌───────▼─────────┐   ┌────────▼─────────┐

             │ PublicSubnetA   │   │ PublicSubnetB   │

             │                 │   │                  │

             │ ProxyServer1    │   │ ProxyServer2     │

             │ ProxySG         │   │ ProxySG2         │

             │                 │   │       ↓          │

             │                 │   │    Bastion       │

             └────────┬────────┘   └──────────────────┘

                      │

                      │ NAT Gateway

                      ▼

             ┌─────────────────────┐

             │   PrivateSubnet     │

             │                     │

             │     AppServer       │

             │    AppServerSG      │

             │                     │

             │   10.0.11.34        │

             └─────────────────────┘

```


---


## 🔍 Task 1 — Analyze the VPC and Private Subnet


The private subnet uses:


```text

10.0.0.0/16 → local

0.0.0.0/0   → NAT Gateway

```


The private AppServer has:


```text

Private IPv4: 10.0.11.34

Public IPv4: None

```


This establishes that the application server is not directly exposed through a public IPv4 address.


The private route table was renamed:


```text

changeme → Private

```


### Security Principle


```text

Private Workload

      │

      ▼

No Direct Public IPv4

      │

      ▼

Controlled Network Path

```


---


## 🌐 Task 2 — Analyze Public Subnets


Public subnets use an Internet Gateway for Internet-bound traffic:


```text

0.0.0.0/0 → Internet Gateway

```


### ProxyServer1


```text

Subnet:

PublicSubnetA


Security Group:

ProxySG

```


### ProxyServer2


```text

Subnet:

PublicSubnetB


Security Group:

ProxySG2

```


`ProxySG2` is configured to permit:


```text

HTTP

TCP

Port 80

Source: Anywhere-IPv4

```


---


## 🌍 Task 3 — Test HTTP Connectivity


Initial testing demonstrates that both proxy servers can reach the private application through the configured forwarding path.


```text

ProxyServer1 → AppServer → Webpage

ProxyServer2 → AppServer → Webpage

```


Expected:


```text

ProxyServer1 → ✅

ProxyServer2 → ✅

```


---


## 🔒 Task 4 — Restrict HTTP Using an IP Address


The AppServer HTTP rule is narrowed from:


```text

0.0.0.0/0

```


to:


```text

10.0.1.94/32

```


The `/32` represents one exact IPv4 address.


### Result


```text

ProxyServer1

10.0.1.94

      │

      ▼

AppServerSG

10.0.1.94/32

      │

      ▼

✅ ALLOWED

```


while:


```text

ProxyServer2

      │

      ▼

Different Source IP

      │

      ▼

AppServerSG

10.0.1.94/32

      │

      ▼

❌ BLOCKED

```


### Security Lesson


A specific source address dramatically reduces the allowed attack surface compared with:


```text

0.0.0.0/0

```


---


## 🛡️ Task 5 — Security-Group Reference


The IP-based rule is replaced with a Security Group reference.


AppServerSG becomes conceptually:


```text

HTTP

TCP

80

Source: ProxySG

```


ProxyServer2 is changed from:


```text

ProxySG2

```


to:


```text

ProxySG

```


### Final Relationship


```text

ProxyServer1 ──┐

               ├── ProxySG ────► AppServerSG ────► AppServer

ProxyServer2 ──┘

```


### Result


```text

ProxyServer1 → ✅

ProxyServer2 → ✅

```


### Why Use an SG Reference?


It avoids hardcoding individual source IP addresses.


```text

IP-based model:


Proxy IP 1

Proxy IP 2

Proxy IP 3

Proxy IP 4

...


SG-reference model:


ProxySG

   │

   └── Approved proxy instances

```


This is easier to scale and maintain.


---


# 🚧 Task 6 — Network ACL Filtering


The experiment introduces a second network-filtering layer.


## Rule 99 — HTTP Deny


```text

Rule: 99

Type: HTTP

Protocol: TCP

Port: 80

Action: DENY

```


Even though the Security Group allows the traffic:


```text

Security Group → ALLOW

NACL             → DENY

Result           → ❌

```


The HTTP request therefore fails.


## Rule 98 — HTTP Allow


A lower-numbered rule is then added:


```text

Rule: 98

Type: HTTP

Protocol: TCP

Port: 80

Action: ALLOW

```


Evaluation:


```text

98 → ALLOW

99 → DENY

```


Because NACL rules are evaluated from the lowest number upward, rule 98 matches first.


### Result


```text

HTTP → ✅ Allowed

```


### Key Concept


> **The first matching NACL rule is applied.**


---


# 🏰 Task 7 — Bastion Host and SSH


ProxyServer2 is repurposed as:


```text

Bastion

```


A dedicated Security Group is created:


```text

BastionSG

```


with:


```text

SSH

TCP

22

Anywhere-IPv4

```


The Bastion receives `BastionSG`, while the previous `ProxySG` association is removed.


---


## 🔐 Bastion → AppServer


AppServerSG receives:


```text

SSH

TCP

22

Source:

10.0.2.83/32

```


This allows SSH from the Bastion's private IP.


```text

Bastion

10.0.2.83

     │

     │ TCP 22

     ▼

AppServer

10.0.11.34

```


---


## 💻 SSH Agent Forwarding


The lab terminal starts an SSH agent:


```bash

exec ssh-agent bash

```


Then loads the provided lab key:


```bash

ssh-add ~/.ssh/labsuser.pem

```


The Bastion is accessed using:


```bash

ssh -i ~/.ssh/labsuser.pem -A ec2-user@54.205.190.113

```


The `-A` flag enables SSH agent forwarding.


From Bastion:


```bash

ssh ec2-user@10.0.11.34

```


On AppServer:


```bash

touch newfile.txt

```


Verification:


```bash

ls -l newfile.txt

```


Then:


```bash

exit

exit

```


### Security Model


```text

Private Key

    │

    ▼

Original Lab Terminal

    │

    │ SSH Agent Forwarding

    ▼

Bastion

    │

    ▼

Private AppServer

```


The private key does not need to be copied onto the Bastion.


---


# 🛰️ Task 8 — AWS Systems Manager Session Manager


Session Manager provides another administration path to the private AppServer.


### Bastion Approach


```text

Terminal

   │

   ▼

Bastion

   │

   ▼

Private AppServer

```


### Session Manager Approach


```text

AWS Console

     │

     ▼

Session Manager

     │

     ▼

Private AppServer

```


This avoids the need for direct inbound SSH access for the Session Manager workflow.


---


## ✏️ Webpage Modification


The supplied command is:


```bash

sudo sed -i 's/instance!/instance! Session manager was used to edit this file./g' /var/www/html/index.html

```


The modification is then verified through ProxyServer1:


```text

http://3.89.227.84

```


Expected:


```text

✅ Webpage loads

✅ Session Manager modification visible

```


---


# 📊 Experiment 04 Security-Control Matrix


| Control | Scope | Purpose | Demonstrated |

|---|---|---|---|

| Security Group | EC2 network interface | Stateful traffic filtering | ✅ |

| SG Reference | EC2-to-EC2 authorization | Scalable source authorization | ✅ |

| Network ACL | Subnet | Additional traffic filtering | ✅ |

| NAT Gateway | Network routing | Private outbound connectivity | ✅ |

| Internet Gateway | VPC routing | Public Internet connectivity | ✅ |

| Bastion Host | Administrative access | Controlled private-network entry | ✅ |

| SSH Agent Forwarding | SSH authentication | Avoid private-key storage on Bastion | ✅ |

| Session Manager | EC2 management | Managed private-instance access | ✅ |


---


# 🧠 Experiment 04 Security Flow


```text

Broad HTTP Access

       │

       ▼

IP-Based Restriction

       │

       ▼

Security-Group Reference

       │

       ▼

NACL Deny / Allow

       │

       ▼

Bastion-Based SSH

       │

       ▼

SSH Agent Forwarding

       │

       ▼

Session Manager

       │

       ▼

Layered Private-Resource Security

```


---


# 🧩 Cross-Experiment Security Model


The four experiments form a layered security path:


```text

┌─────────────────────────────────────────────┐

│              🔐 IDENTITY LAYER              │

│                                             │

│  IAM Users • Roles • Policies • Trust       │

└──────────────────────┬──────────────────────┘

                       │

                       ▼

┌─────────────────────────────────────────────┐

│              🌐 NETWORK LAYER               │

│                                             │

│  VPC • Subnets • Routes • Peering           │

└──────────────────────┬──────────────────────┘

                       │

                       ▼

┌─────────────────────────────────────────────┐

│            🛡️ TRAFFIC CONTROL LAYER         │

│                                             │

│  Security Groups • SG References • NACLs    │

└──────────────────────┬──────────────────────┘

                       │

                       ▼

┌─────────────────────────────────────────────┐

│             🪣 RESOURCE LAYER               │

│                                             │

│  S3 Bucket Policies • Object Permissions    │

└──────────────────────┬──────────────────────┘

                       │

                       ▼

┌─────────────────────────────────────────────┐

│          🔑 ADMINISTRATION LAYER            │

│                                             │

│  Bastion • SSH Agent • Session Manager      │

└──────────────────────┬──────────────────────┘

                       │

                       ▼

              🛡️ DEFENSE IN DEPTH

```


---


# 🔑 Core Cloud Security Concepts


## Authentication vs Authorization


```text

Authentication

      │

      └── Who are you?


Authorization

      │

      └── What are you allowed to do?

```


## Least Privilege


```text

Required Access

      │

      ▼

Grant Only What Is Needed

      │

      ▼

Reduce Attack Surface

```


This appears throughout the repository:


```text

IAM permissions

      +

S3 actions

      +

VPC routes

      +

Security Group sources

      +

NACL rules

      =

Controlled Access

```


## Defense in Depth


```text

Identity Security

       +

Network Isolation

       +

Resource Authorization

       +

Traffic Filtering

       +

Secure Administration

       =

Defense in Depth

```


## Explicit Deny


```text

ALLOW

  +

EXPLICIT DENY

  ↓

DENIED

```


## Network Isolation


```text

VPC

 │

 ├── CIDR

 ├── Subnets

 ├── Route Tables

 ├── Internet Gateway

 ├── NAT Gateway

 ├── Security Groups

 └── Network ACLs

```


## Resource-Level Authorization


```text

AWS Resource

     │

     ▼

Resource Policy

     │

     ▼

Principal

     │

     ▼

Allowed Actions

```


---


# ☁️ AWS Services & Concepts Used


| Service / Concept | Role in Laboratory |

|---|---|

| **AWS IAM** | Identity and access management |

| **IAM Users** | Human identities used for authorization testing |

| **IAM Roles** | Role-based / temporary permission context |

| **IAM Policies** | Define permitted and denied actions |

| **Amazon VPC** | Logical private networking and isolation |

| **Subnets** | Public/private network segmentation |

| **Route Tables** | Control network traffic paths |

| **VPC Peering** | Controlled private connectivity between VPCs |

| **Amazon EC2** | Compute resources and connectivity testing |

| **Security Groups** | Stateful EC2 traffic filtering |

| **Security-Group References** | Scalable instance-to-instance authorization |

| **Network ACLs** | Subnet-level traffic filtering |

| **Internet Gateway** | Public Internet routing |

| **NAT Gateway** | Outbound Internet access for private resources |

| **Amazon S3** | Object storage and resource-based authorization |

| **S3 Bucket Policies** | Resource-level access control |

| **Bastion Host** | Controlled administrative access to private resources |

| **SSH Agent Forwarding** | Secure key-use path without copying the private key |

| **AWS Systems Manager Session Manager** | Managed access to EC2 instances |


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

└── README.md

```


Each experiment directory is intended to contain:


```text

README.md

Documentation / Report

Screenshots / Evidence

```


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


Overall Cloud Security Laboratory

████████████████████ 100% ✅

```


---


# 📚 Security Learning Path


The repository follows a deliberate progression from identity to network, resource, traffic, and administration security:


```text

01

IAM & Roles

   │

   │ Who can access?

   ▼

02

VPC & Peering

   │

   │ Where can resources communicate?

   ▼

03

S3 Resource-Based Policies

   │

   │ What can a principal do to a resource?

   ▼

04

VPC Security Groups

   │

   │ Which traffic is allowed?

   ▼

Secure Administration

   │

   ├── Bastion

   ├── SSH Agent Forwarding

   └── Session Manager

   │

   ▼

🛡️ Layered Cloud Security

```


The complete progression can be summarized as:


```text

Identity

   ↓

Network

   ↓

Resource

   ↓

Traffic

   ↓

Administration

   ↓

Defense in Depth

```


---


# 📝 Documentation


| Experiment | Documentation |

|---|---|

| **01** | IAM Roles & Access Control |

| **02** | AWS VPC & VPC Peering |

| **03** | Using Resource-Based Policies to Secure an S3 Bucket |

| **04** | Securing VPC Resources by Using Security Groups |


Each experiment contains practical implementation details, security concepts, observations, results, conclusions, and supporting evidence.


---


# 🎯 Learning Outcomes


After completing the laboratory, the following capabilities are demonstrated:


- Understand IAM users, roles, policies, and trust relationships.

- Differentiate authentication from authorization.

- Apply least-privilege access control.

- Understand AWS policy-based authorization.

- Use explicit Deny to override broad permissions.

- Build and inspect VPC network boundaries.

- Configure VPC peering and corresponding routes.

- Understand public versus private subnets.

- Analyze Internet Gateway and NAT Gateway routing.

- Use S3 resource-based policies.

- Understand identity-based versus resource-based authorization.

- Configure EC2 Security Groups.

- Restrict traffic using specific IP addresses.

- Use Security Group references.

- Configure and reason about NACL rule precedence.

- Understand Bastion-based private-resource access.

- Use SSH agent forwarding.

- Access private EC2 resources with Session Manager.

- Apply defense-in-depth principles to cloud infrastructure.


---


# 🛡️ Final Security Model


```text

                  🔐 CLOUD SECURITY

                         │

          ┌──────────────┼──────────────┐

          │              │              │

          ▼              ▼              ▼

       Identity        Network       Resource

          │              │              │

          ▼              ▼              ▼

        IAM/VPC       Routing        S3 Policy

          │              │              │

          └──────────────┼──────────────┘

                         │

                         ▼

                  Traffic Security

                         │

                  ┌──────┴──────┐

                  ▼             ▼

             Security       Network ACL

               Groups

                  │

                  ▼

             Administration

                  │

           ┌──────┴──────┐

           ▼             ▼

        Bastion       Session Manager

           │

           ▼

       Private Resources

           │

           ▼

    🛡️ DEFENSE IN DEPTH

```


---


# 🏁 Conclusion


This Cloud Security laboratory demonstrates how multiple AWS security mechanisms can be combined to protect cloud resources.


The experiments progress from:


```text

IAM

 ↓

VPC

 ↓

S3 Authorization

 ↓

Security Groups

 ↓

NACLs

 ↓

Bastion / SSH

 ↓

Session Manager

```


The resulting security model is based on:


```text

Identity

   +

Least Privilege

   +

Network Isolation

   +

Resource Authorization

   +

Traffic Filtering

   +

Secure Administration

   =

🛡️ Defense in Depth

```


> **Cloud security is strongest when identity, network, resource, and administrative controls work together rather than operating as isolated configurations.**


---


# 👨‍💻 Author


**Nitanshu Tak**


B.Tech — Computer Science Engineering  

Major: **Cloud Computing & Virtualization Technology**


---


<p align="center">


### 🔐 Cloud Security Laboratory


**AWS • IAM • VPC • EC2 • S3 • Security Groups • NACLs • Bastion • Session Manager**


</p>
