# 🔐 Experiment 1 — AWS VPC Creation

 

> **Cyber Security Lab**  

> Create and verify a secure, isolated virtual network using Amazon VPC.

 

![AWS](https://img.shields.io/badge/AWS-VPC-orange?style=for-the-badge&logo=amazonaws)

![Region](https://img.shields.io/badge/Region-ap--south--1-blue?style=for-the-badge)

![CIDR](https://img.shields.io/badge/CIDR-10.0.0.0%2F16-green?style=for-the-badge)

 

---

 

## 🎯 Aim

 

To create a custom **Amazon Virtual Private Cloud (VPC)** and understand its importance as a foundational network-security boundary in AWS.

 

## 🧠 What This Experiment Covers

 

- Creating a custom AWS VPC

- Configuring a private IPv4 CIDR block

- Understanding network isolation

- Understanding route tables and network ACLs

- Understanding security groups as resource-level controls

- Relating VPC architecture to cloud-security principles

- Verifying the created VPC through the AWS Console

 

## ⚙️ Configuration

 

| Setting | Value |

|---|---|

| Cloud Platform | Amazon Web Services |

| Region | `ap-south-1` (Mumbai) |

| VPC Name | `nitanshu-cloud-vpc` |

| VPC ID | `vpc-094bb34c470027de3` |

| IPv4 CIDR | `10.0.0.0/16` |

| IPv6 | Disabled |

| Tenancy | Default |

| VPC Encryption Control | None |

| Creation Mode | VPC only |

 

## 🔒 Security Perspective

 

A VPC provides a **logically isolated network boundary** for AWS resources. It is the foundation on which additional security controls can be implemented.

 

The experiment demonstrates:

 

- **Isolation** — workloads can be placed inside a controlled network boundary.

- **Segmentation** — the VPC can later be divided into public and private subnets.

- **Controlled routing** — route tables determine permitted traffic paths.

- **Access control** — security groups provide stateful, resource-level traffic filtering.

- **Defense in depth** — network ACLs can provide additional subnet-level filtering.

- **Reduced exposure** — Internet connectivity should only be introduced when required.

 

> Creating a VPC does **not** automatically make an application secure. Secure routing, subnet design, access controls, monitoring, and IAM permissions are still required.

 

## 🛠️ Procedure

 

1. Open the **Amazon VPC Console**.

2. Select the **Mumbai (`ap-south-1`)** region.

3. Choose **Create VPC**.

4. Select **VPC only**.

5. Set the name to `nitanshu-cloud-vpc`.

6. Configure the IPv4 CIDR as `10.0.0.0/16`.

7. Select **No IPv6 CIDR block**.

8. Keep tenancy as **Default**.

9. Keep VPC encryption control as **None** for this foundational experiment.

10. Create the VPC.

11. Verify that the VPC state is **Available**.

12. Capture the required screenshots for documentation.

 

## ✅ Result

 

A custom AWS VPC was successfully created with the private IPv4 range:

 

```text

10.0.0.0/16

```

 

The VPC was verified in the AWS Console and reached the **Available** state.

 

## 📸 Evidence

 

Screenshots captured during the experiment are included in the accompanying lab documentation.

 

---

 

### 📚 Key Concepts

 

**VPC** → Isolated virtual network in AWS  

**Subnet** → Smaller network segment inside a VPC  

**Route Table** → Controls traffic routing  

**Security Group** → Stateful resource-level firewall  

**Network ACL** → Stateless subnet-level traffic filter  

**CIDR** → Defines the IP address range

 

---

 

## 🏁 Conclusion

 

This experiment established the basic network-security boundary required for AWS workloads. It provides the foundation for future experiments involving **subnetting, routing, security groups, network ACLs, Internet gateways, monitoring, and secure cloud architectures**.

