# 🔐 Experiment 4 — Securing VPC Resources by Using Security Groups


> **AWS Cloud Security Practical | VPC Network Security | EC2 Security Groups | NACLs | Bastion SSH | Session Manager**


<p align="center">

[![AWS](https://img.shields.io/badge/AWS-Cloud%20Security-orange?logo=amazonaws&logoColor=white)](https://aws.amazon.com/) [![Amazon VPC](https://img.shields.io/badge/Amazon%20VPC-Networking-blue?logo=amazonaws&logoColor=white)](https://aws.amazon.com/vpc/) [![Amazon EC2](https://img.shields.io/badge/Amazon%20EC2-Compute-red?logo=amazonaws&logoColor=white)](https://aws.amazon.com/ec2/) [![Security Groups](https://img.shields.io/badge/Security%20Groups-Virtual%20Firewall-purple)](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html) [![Session Manager](https://img.shields.io/badge/SSM-Session%20Manager-success?logo=amazonaws&logoColor=white)](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)

</p>
---


## 📌 Overview


This experiment demonstrates how to secure resources inside an AWS VPC by combining:


- **EC2 Security Groups**

- **Security-Group References**

- **Network Access Control Lists (NACLs)**

- **Public and Private Subnet Routing**

- **NAT Gateway and Internet Gateway**

- **Bastion Host / Jump Server**

- **SSH Agent Forwarding**

- **AWS Systems Manager Session Manager**

- **Least-Privilege Access Control**

- **Defense in Depth**


The lab uses a preconfigured VPC containing two public proxy servers and one private application server. The security configuration is progressively tightened and tested so that the effect of each control can be observed.


---


## 🎯 Objectives


By completing this experiment, the following practical objectives are achieved:


- Examine security groups and determine allowed traffic.

- Change security groups attached to EC2 instances.

- Create a dedicated security group for a Bastion host.

- Configure inbound rules according to least privilege.

- Reference one security group from another security group's inbound rule.

- Configure a NACL to deny and allow HTTP traffic.

- Understand NACL rule-number precedence.

- Access a private EC2 instance using SSH through a Bastion.

- Use SSH agent forwarding without copying the private key to the Bastion.

- Access a private EC2 instance directly with Session Manager.

- Modify and verify a webpage hosted on the private AppServer.


---


## 🧠 Core Security Concepts


### 1. Security Groups


Security Groups act as virtual firewalls for EC2 network interfaces.


They control:


- Inbound traffic

- Outbound traffic

- Allowed protocols

- Allowed ports

- Allowed source or destination


Security Groups are **stateful**, meaning response traffic for an allowed connection is automatically permitted.


### 2. Principle of Least Privilege


The experiment progressively changes the AppServer HTTP rule:


```text

0.0.0.0/0

     ↓

10.0.1.94/32

     ↓

ProxySG

```


This demonstrates the progression from broad access to narrowly defined access.


### 3. Security-Group References


Instead of hardcoding individual source IP addresses, the AppServer can trust a source Security Group.


```text

ProxyServer1 ──┐

               ├── ProxySG ──────► AppServerSG ──────► AppServer

ProxyServer2 ──┘

```


Any EC2 instance carrying `ProxySG` can satisfy the AppServerSG HTTP source condition.


### 4. Network ACLs


NACLs provide an additional filtering layer at the subnet level.


Rules are evaluated by ascending rule number:


```text

98 → evaluated first

99 → evaluated second

```


The first matching rule is applied.


### 5. Bastion Host


A Bastion Host, or jump server, provides a controlled entry point into a private network.


```text

External Terminal

       │

       │ SSH

       ▼

   Bastion

       │

       │ SSH

       ▼

  Private AppServer

```


### 6. SSH Agent Forwarding


The `-A` option forwards the SSH authentication agent.


```text

Private Key

    │

    ▼

Ubuntu Terminal

    │

    │ SSH Agent Forwarding

    ▼

Bastion

    │

    ▼

AppServer

```


The private key itself does **not** need to be copied onto the Bastion.


### 7. Session Manager


Session Manager provides direct managed access to the private AppServer without:


- Opening inbound SSH port 22

- Connecting through a Bastion

- Copying an SSH private key


---


# 🏗️ Lab Architecture


```text

                         Internet

                             │

                             ▼

                    ┌─────────────────┐

                    │ Internet Gateway│

                    └────────┬────────┘

                             │

                 ┌───────────┴───────────┐

                 │        LabVPC          │

                 │      10.0.0.0/16       │

                 │                        │

       ┌─────────▼────────┐     ┌────────▼─────────┐

       │  PublicSubnetA   │     │  PublicSubnetB   │

       │                  │     │                  │

       │ ProxyServer1     │     │ ProxyServer2     │

       │ ProxySG          │     │ ProxySG2         │

       │                  │     │ → later Bastion  │

       └─────────┬────────┘     └──────────────────┘

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


# 📋 Environment


| Component | Configuration |

|---|---|

| VPC | `LabVPC` |

| VPC CIDR | `10.0.0.0/16` |

| Private Subnet | `PrivateSubnet` |

| Public Subnet A | `PublicSubnetA` |

| Public Subnet B | `PublicSubnetB` |

| Private Server | `AppServer` |

| Public Proxy 1 | `ProxyServer1` |

| Public Proxy 2 | `ProxyServer2` |

| Bastion | `ProxyServer2` renamed to `Bastion` |

| Proxy SG | `ProxySG` |

| Initial Proxy 2 SG | `ProxySG2` |

| Application SG | `AppServerSG` |

| Bastion SG | `BastionSG` |

| ProxyServer1 Public IP | `3.89.227.84` |

| ProxyServer1 Private IP | `10.0.1.94` |

| Bastion Public IP | `54.205.190.113` |

| Bastion Private IP | `10.0.2.83` |

| AppServer Private IP | `10.0.11.34` |

| ProxySG ID | `sg-020c2dad74d85cb37` |

| AWS Region | `us-east-1` |


> ⚠️ **Note:** Public/private IP addresses and temporary resource identifiers are specific to the lab session and may change in another session.


---


# 🧪 Task 1 — Analyze VPC and Private Subnet


## 🔍 What was examined?


- `LabVPC`

- `PrivateSubnet`

- Private route table

- NAT Gateway

- `AppServer`

- `AppServerSG`


## ⚙️ Key Configuration


```text

LabVPC CIDR:

10.0.0.0/16


PrivateSubnet:

0.0.0.0/0 → NAT Gateway


10.0.0.0/16 → local

```


The private subnet does not route Internet-bound traffic directly to an Internet Gateway.


Instead:


```text

AppServer

   │

   ▼

Private Route Table

   │

   ▼

NAT Gateway

   │

   ▼

Internet Gateway

   │

   ▼

Internet

```


## 📝 Route Table Rename


The provided route table was renamed:


```text

changeme → Private

```


## 🔐 AppServer Initial Security


Initially:


```text

AppServerSG

└── HTTP / TCP / 80

    Source: 0.0.0.0/0

```


This means HTTP traffic was initially allowed from any IPv4 source.


---


# 🌐 Task 2 — Analyze Public Subnets


## PublicSubnetA


The public subnet routes Internet-bound traffic through an Internet Gateway:


```text

0.0.0.0/0 → Internet Gateway

```


## ProxyServer1


```text

Subnet: PublicSubnetA

Security Group: ProxySG

Access: HTTP / TCP 80

Source: Anywhere-IPv4

```


## ProxyServer2


```text

Subnet: PublicSubnetB

Security Group: ProxySG2

```


`ProxySG2` was modified to allow:


```text

Type: HTTP

Protocol: TCP

Port: 80

Source: Anywhere-IPv4

```


This initially gives both proxy servers equivalent HTTP ingress behavior.


---


# 🌍 Task 3 — Test HTTP Connectivity


The proxy servers forward HTTP requests to the private AppServer.


### ProxyServer1


```text

http://3.89.227.84

```


Expected:


```text

✅ Webpage loads

```


### ProxyServer2


```text

http://54.205.190.113

```


Expected:


```text

✅ Webpage loads

```


The webpage is hosted by the private AppServer even though the browser connects to a public proxy.


---


# 🔒 Task 4 — Restrict HTTP Using an IP Address


The initial AppServerSG rule:


```text

HTTP

TCP

80

0.0.0.0/0

```


was replaced with:


```text

HTTP

TCP

80

10.0.1.94/32

```


## Why `/32`?


`/32` represents one exact IPv4 address.


Therefore:


```text

10.0.1.94/32

```


means:


> Allow HTTP traffic only from ProxyServer1's private IPv4 address.


## 🧪 Test Matrix


| Source | Expected |

|---|---|

| ProxyServer1 | ✅ Allowed |

| ProxyServer2 | ❌ Blocked / Timeout |


### ProxyServer1


```text

10.0.1.94

     │

     ▼

AppServerSG

10.0.1.94/32

     │

     ▼

✅ HTTP Allowed

```


### ProxyServer2


```text

ProxyServer2

     │

     ▼

Different source IP

     │

     ▼

AppServerSG

10.0.1.94/32

     │

     ▼

❌ HTTP Blocked

```


---


# 🛡️ Task 5 — Use a Security-Group Reference


IP-based authorization becomes harder to maintain when more proxy instances are introduced.


Instead of:


```text

ProxyServer1 IP

ProxyServer2 IP

ProxyServer3 IP

ProxyServer4 IP

...

```


the lab uses:


```text

ProxySG

```


## AppServerSG Configuration


The old IP rule is deleted.


A new rule is created:


```text

Type: HTTP

Protocol: TCP

Port: 80

Source: Custom

Source SG: ProxySG

```


Lab-session Security Group ID:


```text

sg-020c2dad74d85cb37

```


## ProxyServer2 Configuration


Before:


```text

ProxyServer2

└── ProxySG2

```


After:


```text

ProxyServer2

└── ProxySG

```


Therefore:


```text

ProxyServer1 ──► ProxySG ──┐

                           ├──► AppServerSG ──► AppServer

ProxyServer2 ──► ProxySG ──┘

```


## 🧪 Result


```text

ProxyServer1 → ✅

ProxyServer2 → ✅

```


### Why this is better


The destination does not need to maintain a list of individual source IP addresses.


Adding another approved proxy becomes conceptually:


```text

New Proxy

    │

    ▼

Attach ProxySG

    │

    ▼

AppServerSG recognizes ProxySG

    │

    ▼

HTTP permitted

```


---


# 🚧 Task 6 — Restrict HTTP with a Network ACL


The VPC's network ACL was modified to demonstrate an additional security layer.


## Rule 99 — Deny HTTP


```text

Rule Number: 99

Type: HTTP

Protocol: TCP

Port: 80

Action: DENY

```


### Test


```text

ProxyServer1

     │

     ▼

ProxySG → allows HTTP

     │

     ▼

NACL Rule 99 → denies HTTP

     │

     ▼

❌ Connection timeout

```


This proves that a Security Group allowing traffic does not override a NACL that denies it.


---


## Rule 98 — Allow HTTP


A second rule was then added:


```text

Rule Number: 98

Type: HTTP

Protocol: TCP

Port: 80

Action: ALLOW

```


The final relevant ordering becomes:


```text

98 → HTTP → ALLOW

99 → HTTP → DENY

```


Because lower rule numbers are evaluated first:


```text

98

↓

MATCH

↓

ALLOW

↓

Rule 99 is not reached

```


### Final Test


```text

ProxyServer1 → ✅ Webpage loads

```


---


# 🏰 Task 7 — Bastion Host + SSH


ProxyServer2 is repurposed as a Bastion Host.


## Rename Instance


```text

ProxyServer2

      ↓

Bastion

```


## Create BastionSG


```text

Name:

BastionSG


Description:

BastionSG


VPC:

LabVPC

```


Inbound rule:


```text

Type: SSH

Protocol: TCP

Port: 22

Source: Anywhere-IPv4

```


## Attach BastionSG


Remove:


```text

ProxySG

```


Add:


```text

BastionSG

```


---


# 🔐 Allow Bastion → AppServer SSH


AppServerSG receives an additional rule:


```text

Type: SSH

Protocol: TCP

Port: 22

Source: 10.0.2.83/32

```


This means:


```text

Only BastionPrivateIP

        │

        ▼

TCP 22

        │

        ▼

AppServer

```


---


# 💻 SSH Agent Setup


Run in the provided lab terminal:


```bash

exec ssh-agent bash

```


Then:


```bash

ssh-add ~/.ssh/labsuser.pem

```


The key is loaded into the SSH authentication agent.


---


# 🔑 Connect to Bastion


```bash

ssh -i ~/.ssh/labsuser.pem -A ec2-user@54.205.190.113

```


The `-A` option enables SSH agent forwarding.


When prompted:


```text

Are you sure you want to continue connecting?

```


enter:


```text

yes

```


Expected prompt:


```text

[ec2-user@bastion ~]$

```


---


# 🔗 Connect from Bastion to AppServer


Run:


```bash

ssh ec2-user@10.0.11.34

```


When prompted for host authenticity:


```text

yes

```


Expected prompt:


```text

[ec2-user@appserver ~]$

```


---


# 📄 Create Verification File


Run:


```bash

touch newfile.txt

```


Optional verification:


```bash

ls -l newfile.txt

```


Expected result resembles:


```text

-rw-rw-r-- 1 ec2-user ec2-user 0 ... newfile.txt

```


---


# 🚪 Exit SSH Sessions


First:


```bash

exit

```


This returns from AppServer to Bastion.


Then:


```bash

exit

```


This returns from Bastion to the external lab terminal.


Expected flow:


```text

AppServer

   │

   │ exit

   ▼

Bastion

   │

   │ exit

   ▼

Ubuntu / runweb terminal

```


---


# 🧠 Why SSH Agent Forwarding Matters


The private key:


```text

labsuser.pem

```


remains on the original terminal.


It is **not copied onto Bastion**.


The authentication path is:


```text

Original Terminal

       │

       │ SSH Agent

       ▼

    Bastion

       │

       │ forwarded authentication

       ▼

   AppServer

```


This is preferable to placing a private SSH key directly on the intermediate Bastion.


---


# 🛰️ Task 8 — AWS Systems Manager Session Manager


Session Manager provides an alternative to Bastion-based administration.


## Traditional Approach


```text

Internet

   │

   ▼

Bastion

   │

   ▼

Private AppServer

```


## Session Manager Approach


```text

AWS Console

     │

     ▼

Session Manager

     │

     ▼

Private AppServer

```


No direct inbound SSH connection from the Internet is required.


---


# 🖥️ Connect Using Session Manager


Navigate:


```text

EC2

  ↓

Instances

  ↓

AppServer

  ↓

Connect

  ↓

Session Manager

  ↓

Connect

```


A terminal opens with a prompt similar to:


```text

sh-4.2$

```


---


# ✏️ Modify the Webpage


Run the exact lab command:


```bash

sudo sed -i 's/instance!/instance! Session manager was used to edit this file./g' /var/www/html/index.html

```


This replaces the matching text in:


```text

/var/www/html/index.html

```


with:


```text

instance! Session manager was used to edit this file.

```


---


# 🌐 Verify the Modification


Open:


```text

http://3.89.227.84

```


Expected:


```text

✅ Webpage loads

```


The page should now display the Session Manager modification.


If the old page appears:


```text

Hard Refresh

```


the browser to avoid cached content.


---


# 📊 Complete Experiment Flow


```text

┌──────────────────────────────────────────────┐

│              INITIAL NETWORK                 │

│                                              │

│ Public Proxy 1 ───────┐                      │

│                       ├────► Private AppServer│

│ Public Proxy 2 ───────┘                      │

└──────────────────────────────────────────────┘

                       │

                       ▼

              Restrict by IP /32

                       │

                       ▼

             ProxyServer1 only

                       │

                       ▼

            Replace IP with ProxySG

                       │

                       ▼

          ProxyServer1 + ProxyServer2

                       │

                       ▼

             Add NACL HTTP DENY

                       │

                       ▼

                  HTTP blocked

                       │

                       ▼

             Add lower rule 98 ALLOW

                       │

                       ▼

                  HTTP allowed

                       │

                       ▼

           Repurpose ProxyServer2

                  as Bastion

                       │

                       ▼

             SSH Agent Forwarding

                       │

                       ▼

              Private AppServer

                       │

                       ▼

              Session Manager

                       │

                       ▼

        Direct private-instance access

```


---


# 📈 Security Control Comparison


| Control | Scope | Main Purpose | Demonstrated |

|---|---|---|---|

| Security Group | EC2 network interface | Instance traffic filtering | ✅ |

| SG Reference | EC2-to-EC2 authorization | Scalable source authorization | ✅ |

| NACL | Subnet | Additional network filtering | ✅ |

| NAT Gateway | Subnet routing | Private outbound Internet access | ✅ |

| Internet Gateway | VPC routing | Public Internet connectivity | ✅ |

| Bastion | Administrative access | Controlled private-network entry | ✅ |

| SSH Agent Forwarding | SSH authentication | Avoid private-key storage on Bastion | ✅ |

| Session Manager | Managed EC2 access | Direct private-instance administration | ✅ |


---


# 🧪 Final Results


### Security Group Testing


```text

Initial HTTP:

ProxyServer1 → PASS

ProxyServer2 → PASS


IP restriction:

ProxyServer1 → PASS

ProxyServer2 → FAIL


SG reference:

ProxyServer1 → PASS

ProxyServer2 → PASS

```


### NACL Testing


```text

Rule 99 DENY:

HTTP → FAIL


Rule 98 ALLOW + Rule 99 DENY:

HTTP → PASS

```


### SSH Testing


```text

Ubuntu → Bastion → AppServer

             │

             └── SSH Agent Forwarding

```


Result:


```text

PASS

```


### Session Manager Testing


```text

AWS Console → Session Manager → AppServer

```


Result:


```text

PASS

```


### File Verification


```text

newfile.txt

```


was created on the private AppServer.


### Webpage Verification


The webpage was modified through Session Manager and successfully verified through ProxyServer1.


---


# 🛡️ Security Takeaways


> **1. Never rely on a single network control.**


Security Groups and NACLs can work together as layers.


> **2. Prefer least privilege.**


Avoid:


```text

0.0.0.0/0

```


when a narrower source is possible.


> **3. Security-group references scale better than individual IP rules.**


They avoid maintaining a growing list of source IP addresses.


> **4. Keep private workloads private.**


AppServer does not require a public IPv4 address.


> **5. Protect SSH credentials.**


SSH agent forwarding avoids copying the private key to the Bastion.


> **6. Consider Session Manager for administration.**


It can eliminate the need for inbound SSH and a dedicated Bastion for many management scenarios.


> **7. Layered security provides defense in depth.**


```text

Routing

   +

Security Groups

   +

NACLs

   +

Controlled Administration

   =

Defense in Depth

```


---


# 📝 Key Commands


```bash

# Start SSH agent

exec ssh-agent bash


# Add lab SSH key

ssh-add ~/.ssh/labsuser.pem


# Connect to Bastion with agent forwarding

ssh -i ~/.ssh/labsuser.pem -A ec2-user@54.205.190.113


# Connect to private AppServer

ssh ec2-user@10.0.11.34


# Create verification file

touch newfile.txt


# Verify file

ls -l newfile.txt


# Exit

exit


# Exit Bastion

exit


# Modify AppServer webpage through Session Manager

sudo sed -i 's/instance!/instance! Session manager was used to edit this file./g' /var/www/html/index.html

```


---


# 📸 Evidence Checklist


Recommended screenshots for this experiment:


- [ ] LabVPC and CIDR

- [ ] PrivateSubnet

- [ ] Private route table

- [ ] NAT Gateway

- [ ] AppServer details

- [ ] Initial AppServerSG

- [ ] PublicSubnetA route table

- [ ] ProxyServer1 / ProxySG

- [ ] ProxyServer2 / ProxySG2

- [ ] ProxySG2 HTTP rule

- [ ] ProxyServer1 webpage

- [ ] ProxyServer2 webpage

- [ ] AppServerSG with `10.0.1.94/32`

- [ ] ProxyServer1 success after IP restriction

- [ ] ProxyServer2 blocked after IP restriction

- [ ] AppServerSG with ProxySG source

- [ ] ProxyServer2 changed to ProxySG

- [ ] Both proxy servers successful

- [ ] NACL rule 99 DENY

- [ ] NACL rule 98 ALLOW

- [ ] BastionSG SSH rule

- [ ] Bastion attached to BastionSG

- [ ] AppServerSG SSH rule

- [ ] `ssh-agent` command

- [ ] `ssh-add` command

- [ ] Successful Bastion SSH session

- [ ] Successful AppServer SSH session

- [ ] `newfile.txt`

- [ ] Session Manager terminal

- [ ] `sed` command

- [ ] Final webpage

- [ ] Submission / Grades / Submission Report


---


# 📂 Suggested Repository Structure


```text

Cloud-Security/

│

├── Experiment-01-IAM/

│

├── Experiment-02-VPC-Peering/

│

├── Experiment-03-S3-Resource-Policies/

│

└── Experiment-04.1-VPC-Security-Groups/

    │

    ├── README.md

    ├── Experiment-4.1-Report.docx

    │

    └── screenshots/

        ├── task-01/

        ├── task-02/

        ├── task-03/

        ├── task-04/

        ├── task-05/

        ├── task-06/

        ├── task-07/

        └── task-08/

```


---


# 🏁 Conclusion


Experiment 4.1 demonstrates practical VPC resource security using multiple AWS controls.


The experiment begins with broad HTTP access and progressively applies stronger controls:


```text

Open HTTP

   ↓

IP-based restriction

   ↓

Security-group reference

   ↓

NACL filtering

   ↓

Bastion-based SSH

   ↓

SSH Agent Forwarding

   ↓

Session Manager

```


The final architecture keeps the application server private while providing controlled application and administrative access.


The experiment therefore demonstrates:


```text

Least Privilege

       +

Network Segmentation

       +

Security Groups

       +

NACLs

       +

Secure Administration

       +

Managed Access

       ↓

Layered AWS VPC Security

```


---


## ✅ Experiment Status


| Component | Status |

|---|---|

| VPC Analysis | ✅ Completed |

| Private Subnet Analysis | ✅ Completed |

| Public Subnet Analysis | ✅ Completed |

| HTTP Connectivity Testing | ✅ Completed |

| IP-Based Restriction | ✅ Completed |

| Security-Group Reference | ✅ Completed |

| Network ACL Filtering | ✅ Completed |

| Bastion Host | ✅ Completed |

| SSH Agent Forwarding | ✅ Completed |

| Private AppServer SSH | ✅ Completed |

| Session Manager | ✅ Completed |

| Webpage Modification | ✅ Completed |

| Final Verification | ✅ Completed |


---


## 👤 Author


**Cloud Security Practical Laboratory**


> Experiment 4.1 — Securing VPC Resources by Using Security Groups
