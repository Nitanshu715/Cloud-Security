# 🔐 AWS VPC & VPC Peering — Cyber Security Laboratory

> **Experiment 1 · AWS Network Security**
>
> Build two isolated Virtual Private Clouds, establish a private VPC Peering connection, configure bidirectional routing, and verify private network connectivity between workloads.

---

## 🧭 Experiment Overview

This experiment demonstrates the practical creation and configuration of **two independent Amazon Virtual Private Clouds (VPCs)** and their controlled interconnection using **VPC Peering**.

The first VPC acts as the primary laboratory network and the second VPC acts as an independent peer network. Because the two VPCs use **non-overlapping IPv4 CIDR ranges**, AWS can establish a private peering relationship between them. Route tables are then configured on both sides so that traffic destined for the remote VPC is forwarded through the peering connection.

The experiment goes beyond simply creating a VPC. It demonstrates how cloud networks can remain logically isolated while still allowing **explicitly controlled communication** between trusted network boundaries.

---

## 🎯 Aim

To create and configure two logically isolated AWS VPCs, establish a VPC Peering connection between them, configure the required bidirectional routes, and verify private connectivity between resources using their internal IPv4 addresses.

---

## 🧠 Learning Objectives

- Create custom AWS VPCs.
- Configure private IPv4 CIDR blocks.
- Create subnets inside a VPC.
- Understand logical network isolation.
- Establish VPC Peering between independent VPCs.
- Understand the requirement for non-overlapping CIDRs.
- Create and accept a peering request.
- Configure route tables for inter-VPC traffic.
- Configure bidirectional routing.
- Apply Security Group based traffic controls.
- Verify connectivity using private IP addresses.
- Apply defense-in-depth and least-exposure principles.

---

## ☁️ AWS Services Used

| Service | Purpose |
|---|---|
| **Amazon VPC** | Creates isolated virtual networks in AWS. |
| **Amazon VPC Peering** | Provides private connectivity between two VPCs. |
| **Amazon EC2** | Provides test workloads for validating private connectivity. |
| **Amazon VPC Route Tables** | Determines where network traffic is forwarded. |
| **Amazon VPC Subnets** | Provides smaller IP ranges inside each VPC. |
| **Amazon EC2 Security Groups** | Controls traffic reaching test EC2 instances. |
| **EC2 Instance Connect** | Provides browser-based access to the test instance. |

---

## 🌐 Laboratory Architecture

```text
                         AWS ACCOUNT
                              │
              ┌───────────────┴───────────────┐
              │                               │
              ▼                               ▼
      ┌─────────────────┐             ┌─────────────────┐
      │      VPC-A      │             │      VPC-B      │
      │                 │             │                 │
      │ nitanshu-cloud  │             │    my-vpc-01    │
      │ 10.0.0.0/24     │             │ 10.0.1.0/24     │
      │                 │             │                 │
      │  Subnet A       │             │  Subnet B       │
      │ 10.0.0.0/24     │             │ 10.0.1.0/25     │
      │                 │             │                 │
      └────────┬────────┘             └────────┬────────┘
               │                               │
               └──────── VPC PEERING ──────────┘
                         pcx-xxxxxxxx
                              │
                    Private IP Connectivity
```

### Traffic Flow

```text
VPC-A → 10.0.1.0/24 → VPC Peering → VPC-B

VPC-B → 10.0.0.0/24 → VPC Peering → VPC-A
```

---

## ⚙️ Configuration Used

### VPC-A — Primary Network

| Parameter | Value |
|---|---|
| **Name** | `nitanshu-cloud-vpc` |
| **VPC ID** | `vpc-0135da4033af34c92` |
| **IPv4 CIDR** | `10.0.0.0/24` |
| **Region** | `ap-south-1` — Mumbai |
| **State** | Available |
| **IPv6** | None |
| **Tenancy** | Default |
| **DNS Resolution** | Enabled |
| **DNS Hostnames** | Disabled |
| **Default VPC** | No |
| **Subnet** | `my-subnet-01` |
| **Main Route Table** | `rtb-0ef78ea124e9fd4c1` |

### VPC-B — Peer Network

| Parameter | Value |
|---|---|
| **Name** | `my-vpc-01` |
| **VPC ID** | `vpc-02b128ceff9115c7a` |
| **IPv4 CIDR** | `10.0.1.0/24` |
| **Region** | `ap-south-1` — Mumbai |
| **State** | Available |
| **IPv6** | None |
| **Tenancy** | Default |
| **DNS Resolution** | Enabled |
| **DNS Hostnames** | Disabled |
| **Default VPC** | No |
| **Subnet** | `my-vpc-01-subnet` |
| **Main Route Table** | `rtb-0c162c3c5af181d83` |

---

## 🧩 CIDR Design

The two VPCs use non-overlapping address spaces:

```text
VPC-A → 10.0.0.0/24
VPC-B → 10.0.1.0/24
```

This separation allows AWS to distinguish traffic belonging to the local network from traffic belonging to the remote network.

The subnet inside VPC-B uses:

```text
10.0.1.0/25
```

which is a smaller address range contained within VPC-B's `10.0.1.0/24` network.

---

# 🔗 VPC PEERING

## What is VPC Peering?

VPC Peering is a private networking relationship that allows resources in two VPCs to communicate using private IP addresses, subject to routing and security controls.

For this experiment:

```text
nitanshu-cloud-vpc
        │
        │ VPC Peering
        ▼
my-vpc-01
```

The peering connection does not merge the two VPCs into one network. They remain independent VPCs with separate address spaces and routing configurations.

---

## 🛡️ Security Perspective

VPC Peering allows controlled private communication between separate network boundaries without requiring workloads to communicate through publicly exposed paths.

It is useful for:

- Private communication between trusted VPC environments.
- Shared-service architectures.
- Controlled application-to-service communication.
- Reducing unnecessary public exposure.
- Connecting independent cloud network environments.

A peering connection alone does not automatically permit all traffic. Route tables and security controls still determine whether communication is possible.

---

# 🛠️ Procedure

## 1. Verify VPC-A

The existing custom VPC was used as the primary network:

```text
Name: nitanshu-cloud-vpc
CIDR: 10.0.0.0/24
Region: ap-south-1
```

The VPC was verified as an available, non-default VPC.

---

## 2. Create VPC-B

A second custom VPC was created:

```text
Name: my-vpc-01
CIDR: 10.0.1.0/24
Region: ap-south-1
```

The CIDR was deliberately selected so that it does not overlap with VPC-A.

---

## 3. Create a Subnet in VPC-B

A subnet was created inside VPC-B:

```text
Subnet Name:
my-vpc-01-subnet

CIDR:
10.0.1.0/25
```

The subnet provides an address range inside VPC-B where a test workload can be deployed.

VPC-A already contained:

```text
my-subnet-01
```

---

## 4. Create the VPC Peering Connection

Navigate to:

```text
AWS Console
→ VPC
→ Peering connections
→ Create peering connection
```

Configuration:

```text
Peering Name:
nitanshu-vpc-peering

Requester VPC:
nitanshu-cloud-vpc

Requester CIDR:
10.0.0.0/24

Accepter VPC:
my-vpc-01

Accepter CIDR:
10.0.1.0/24
```

The peering request was created successfully.

---

## 5. Accept the Peering Request

The created peering connection initially required acceptance.

Navigate to:

```text
VPC
→ Peering connections
→ Actions
→ Accept request
```

After acceptance, the connection status became:

```text
Active
```

An **Active** status confirms that the peering relationship was successfully established.

---

# 🛣️ ROUTING CONFIGURATION

## 6. Configure VPC-A Route Table

VPC-A must know that traffic destined for VPC-B should use the peering connection.

The route added to VPC-A was:

| Destination | Target |
|---|---|
| `10.0.1.0/24` | VPC Peering Connection |

Conceptually:

```text
VPC-A Route Table

10.0.0.0/24 → local
10.0.1.0/24 → pcx-xxxxxxxx
```

This directs traffic for VPC-B through the peering connection.

---

## 7. Configure VPC-B Route Table

The reverse route was added to VPC-B:

| Destination | Target |
|---|---|
| `10.0.0.0/24` | VPC Peering Connection |

Conceptually:

```text
VPC-B Route Table

10.0.1.0/24 → local
10.0.0.0/24 → pcx-xxxxxxxx
```

This provides the return path from VPC-B to VPC-A.

---

## 🔄 Why Both Routes Are Required

Network communication requires a valid path in both directions:

```text
                 REQUEST
VPC-A ─────────────────────────► VPC-B
       10.0.1.0/24 → Peering


                 RESPONSE
VPC-B ─────────────────────────► VPC-A
       10.0.0.0/24 → Peering
```

Without the reverse route, return traffic may not reach its source.

---

# 🔒 SECURITY GROUP CONFIGURATION

For the connectivity demonstration, ICMP traffic was permitted only from the remote VPC's private CIDR.

### VPC-A Test Instance

```text
Inbound ICMP source:
10.0.1.0/24
```

### VPC-B Test Instance

```text
Inbound ICMP source:
10.0.0.0/24
```

This avoids unnecessarily exposing the test traffic to:

```text
0.0.0.0/0
```

and demonstrates a more restrictive trusted-source model.

---

# 💻 PRIVATE CONNECTIVITY TEST

Two EC2 instances can be used to validate the configuration:

```text
┌───────────────────────┐
│        EC2-A          │
│        VPC-A          │
│                       │
│ Private IP: 10.0.0.x  │
└──────────┬────────────┘
           │
           │ ICMP
           ▼
     VPC PEERING
           │
           ▼
┌──────────┴────────────┐
│        EC2-B          │
│        VPC-B          │
│                       │
│ Private IP: 10.0.1.x  │
└───────────────────────┘
```

From EC2-A, the private address of EC2-B can be tested with:

```bash
ping <EC2-B-private-IP>
```

Example:

```bash
ping 10.0.1.57
```

A successful result resembles:

```text
PING 10.0.1.57 (10.0.1.57) 56(84) bytes of data.
64 bytes from 10.0.1.57: icmp_seq=1 ttl=64 time=...
64 bytes from 10.0.1.57: icmp_seq=2 ttl=64 time=...
64 bytes from 10.0.1.57: icmp_seq=3 ttl=64 time=...
64 bytes from 10.0.1.57: icmp_seq=4 ttl=64 time=...

4 packets transmitted, 4 received, 0% packet loss
```

A successful private-IP test provides practical evidence that the peering connection, routes, subnet placement, and relevant security rules are working together.

---

# 🔍 Verification Checklist

| Component | Expected Result |
|---|---|
| VPC-A | Available |
| VPC-B | Available |
| VPC-A CIDR | `10.0.0.0/24` |
| VPC-B CIDR | `10.0.1.0/24` |
| CIDR overlap | None |
| VPC-B subnet | Created |
| Peering connection | Active |
| VPC-A route to VPC-B | Present |
| VPC-B route to VPC-A | Present |
| EC2-A | Running in VPC-A |
| EC2-B | Running in VPC-B |
| ICMP permissions | Restricted to peer CIDR |
| Private ping | Successful |
| Packet loss | `0%` |

---

# 📸 Practical Evidence

The experiment screenshots should document the following:

1. **VPC-B configuration** showing `my-vpc-01` and `10.0.1.0/24`.
2. **VPC-B subnet** showing `my-vpc-01-subnet` and `10.0.1.0/25`.
3. **VPC Peering connection** after creation.
4. **VPC Peering connection — Active**.
5. **VPC-A route table** showing `10.0.1.0/24 → Peering Connection`.
6. **VPC-B route table** showing `10.0.0.0/24 → Peering Connection`.
7. **EC2 instances** deployed in the respective VPCs.
8. **Security Group rules** permitting ICMP from the trusted peer CIDR.
9. **EC2 Instance Connect terminal** showing successful private-IP communication.
10. **Final connectivity verification** showing received packets and `0% packet loss`.

---

# 🧪 Observation

The experiment demonstrated that two independently configured VPCs can remain separate network environments while communicating through a specifically established VPC Peering connection.

The key observation is:

```text
Peering Connection
        +
Correct Routes
        +
Security Rules
        =
Private Connectivity
```

The peering connection establishes the relationship, route tables define the traffic paths, and Security Groups provide resource-level traffic control.

---

# 🛡️ Cyber Security Analysis

### 1. Network Isolation

Each VPC remains an independent logical network boundary.

```text
VPC-A ≠ VPC-B
```

They retain separate CIDR ranges, subnets, route tables, and security controls.

### 2. Controlled Connectivity

Communication is introduced deliberately through a VPC Peering connection rather than assuming unrestricted connectivity between networks.

### 3. Least Exposure

Private IP addresses are used for inter-VPC communication, while ICMP access is restricted to the known remote VPC CIDR.

### 4. Defense in Depth

The experiment demonstrates layered controls:

```text
VPC Boundary
     ↓
Subnet
     ↓
Route Table
     ↓
VPC Peering
     ↓
Security Group
     ↓
EC2 Workload
```

### 5. Attack Surface Reduction

Avoiding unnecessary public exposure reduces the attack surface. Private peering allows controlled communication without requiring the test workloads to use the public Internet for their inter-VPC traffic.

---

# ⚠️ Important Networking Notes

### Peering does not automatically create routes

Creating a peering connection does not automatically add the required routes to the route tables. Routes to the remote CIDR must be explicitly configured.

### Security Groups still apply

VPC Peering does not bypass EC2 Security Groups. The destination workload must still permit the required traffic.

### CIDRs must not overlap

VPCs intended for peering must use non-overlapping address ranges.

### Peering is not transitive

If:

```text
VPC-A ↔ VPC-B
VPC-B ↔ VPC-C
```

this does not automatically create:

```text
VPC-A ↔ VPC-C
```

### Private IP testing matters

Testing with private addresses demonstrates that the communication uses the internal AWS networking path rather than relying on public Internet connectivity.

---

# 📊 Final Configuration Summary

```text
REGION
ap-south-1 (Mumbai)

VPC-A
Name: nitanshu-cloud-vpc
CIDR: 10.0.0.0/24

VPC-B
Name: my-vpc-01
CIDR: 10.0.1.0/24

PEERING
Name: nitanshu-vpc-peering
Status: Active

ROUTE A → B
10.0.1.0/24 → Peering Connection

ROUTE B → A
10.0.0.0/24 → Peering Connection

CONNECTIVITY
Private IPv4 communication verified
```

---

# ✅ Result

Two separate AWS VPCs were successfully configured with non-overlapping private IPv4 address ranges. A VPC Peering connection was established and accepted successfully, after which bidirectional routes were configured in the respective route tables. Security Group rules were used to restrict test traffic to the trusted peer network, and private-IP connectivity between workloads in the two VPCs was verified.

The experiment successfully demonstrated **VPC creation, network isolation, subnet configuration, VPC Peering, route-table configuration, Security Group based traffic control, and private inter-VPC communication**.

---

# 🏁 Conclusion

This experiment established how secure cloud networking can be designed by combining isolated VPC environments with explicitly controlled private connectivity. The implementation showed that VPC Peering does not eliminate network isolation; instead, it provides a controlled communication mechanism between otherwise independent VPCs.

The practical verification of private-IP connectivity confirmed that the peering connection, routing configuration, and security controls were correctly aligned. From a cybersecurity perspective, the experiment reinforces the importance of intentional network boundaries, minimal exposure, explicit routing, trusted-source filtering, and layered access controls when designing cloud infrastructure.

---

# 🎓 Viva Questions

### What is an AWS VPC?

An AWS VPC is a logically isolated virtual network in AWS where cloud resources can be deployed and network traffic can be controlled.

### What is VPC Peering?

VPC Peering is a private networking connection that allows resources in two VPCs to communicate using private IP addresses.

### Why must VPC CIDRs not overlap?

Overlapping address ranges create ambiguous routing and prevent the required peering configuration.

### Does creating a peering connection automatically configure routing?

No. Routes to the remote VPC CIDR must be explicitly added to the relevant route tables.

### Why are two routes required?

One route provides the path from VPC-A to VPC-B, while the other provides the return path from VPC-B to VPC-A.

### Why are private IP addresses used for the test?

Private IP testing verifies communication through the internal AWS networking path rather than public Internet connectivity.

### What is a route table?

A route table contains rules that determine where network traffic is sent based on its destination IP range.

### What is a Security Group?

A Security Group is a stateful virtual firewall associated with AWS resources such as EC2 instances.

### Why restrict ICMP to the peer CIDR?

It follows least-privilege principles by permitting required test traffic only from the known trusted network.

### Is VPC Peering transitive?

No. A peering relationship between two VPCs does not automatically provide routing through another peered VPC.

### Why is VPC Peering useful from a security perspective?

It enables controlled private communication between separate network boundaries without requiring workloads to communicate through publicly exposed network paths.

---

## 📁 Repository Structure

```text
experiment-2-vpc-peering/
│
├── README.md
│
└── screenshots/
    ├── 01-vpc-b.png
    ├── 02-vpc-b-subnet.png
    ├── 03-peering-created.png
    ├── 04-peering-active.png
    ├── 05-vpc-a-route.png
    ├── 06-vpc-b-route.png
    ├── 07-ec2-instances.png
    ├── 08-security-groups.png
    └── 09-private-connectivity-test.png
```

---

## 🔐 Key Takeaways

```text
1. VPC = isolated cloud network boundary

2. Subnet = smaller address range inside a VPC

3. VPC Peering = private connection between two VPCs

4. Non-overlapping CIDRs = required for clean peering

5. Peering alone ≠ routing

6. Route tables define traffic paths

7. Security Groups control workload access

8. Private-IP testing verifies internal connectivity

9. Least privilege should apply to network rules

10. Secure cloud networking requires layered controls
```

---

<p align="center">

### 🔐 AWS Network Security Lab

**VPC Creation · VPC Peering · Routing · Security Controls · Private Connectivity**

</p>

<p align="center">

`AWS` · `Amazon VPC` · `VPC Peering` · `EC2` · `Cloud Security`

</p>

