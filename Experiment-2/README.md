# 🔐 EXPERIMENT 2 — AWS IAM SECURITY, AUTHORIZATION & POLICY ENFORCEMENT

 

> **Cloud Security Laboratory — Experiment 2**  

> **Primary Service:** AWS Identity and Access Management (IAM)  

> **Supporting Services:** Amazon EC2 and Amazon S3  

> **Core Topics:** IAM Users • Authentication • Authorization • Managed Policies • Custom Policies • Allow/Deny Evaluation • Explicit Deny • Least Privilege • Security Controls • Resource Cleanup

 

---

 

## 📌 Table of Contents

 

- [1. Experiment Overview](#1-experiment-overview)

- [2. Aim](#2-aim)

- [3. Objectives](#3-objectives)

- [4. Problem Statement](#4-problem-statement)

- [5. Security Scenario](#5-security-scenario)

- [6. AWS IAM Introduction](#6-aws-iam-introduction)

- [7. Authentication vs Authorization](#7-authentication-vs-authorization)

- [8. IAM Users](#8-iam-users)

- [9. IAM Policies](#9-iam-policies)

- [10. Policy Elements](#10-policy-elements)

- [11. Allow and Deny](#11-allow-and-deny)

- [12. IAM Policy Evaluation Logic](#12-iam-policy-evaluation-logic)

- [13. AWS Managed Policies](#13-aws-managed-policies)

- [14. Custom IAM Policies](#14-custom-iam-policies)

- [15. Experiment Architecture](#15-experiment-architecture)

- [16. Part A — EC2 IAM User](#16-part-a--ec2-iam-user)

- [17. EC2 Permission Verification](#17-ec2-permission-verification)

- [18. Part B — S3 IAM User](#18-part-b--s3-iam-user)

- [19. S3 Permission Verification](#19-s3-permission-verification)

- [20. Part C — Explicit Deny](#20-part-c--explicit-deny)

- [21. Custom S3 Deny Policy](#21-custom-s3-deny-policy)

- [22. Detailed JSON Explanation](#22-detailed-json-explanation)

- [23. Why Full S3 Access Was Retained](#23-why-full-s3-access-was-retained)

- [24. Bucket Deletion Verification](#24-bucket-deletion-verification)

- [25. Security Analysis](#25-security-analysis)

- [26. Least Privilege](#26-least-privilege)

- [27. Separation of Duties](#27-separation-of-duties)

- [28. Defense in Depth](#28-defense-in-depth)

- [29. Destructive Action Protection](#29-destructive-action-protection)

- [30. Managed vs Custom Policies](#30-managed-vs-custom-policies)

- [31. IAM Users vs IAM Roles](#31-iam-users-vs-iam-roles)

- [32. Why IAM Users Should Be Minimized](#32-why-iam-users-should-be-minimized)

- [33. Security Risks of Full Access](#33-security-risks-of-full-access)

- [34. Resource Scope and Wildcards](#34-resource-scope-and-wildcards)

- [35. Conditions and Advanced Restrictions](#35-conditions-and-advanced-restrictions)

- [36. Policy Evaluation Examples](#36-policy-evaluation-examples)

- [37. Practical Observations](#37-practical-observations)

- [38. Verification Matrix](#38-verification-matrix)

- [39. Screenshot Evidence](#39-screenshot-evidence)

- [40. Cleanup and Cost Control](#40-cleanup-and-cost-control)

- [41. Important Resource Ownership Concept](#41-important-resource-ownership-concept)

- [42. Common Mistakes](#42-common-mistakes)

- [43. Troubleshooting](#43-troubleshooting)

- [44. Security Best Practices](#44-security-best-practices)

- [45. Real-World Application](#45-real-world-application)

- [46. Viva Questions and Answers](#46-viva-questions-and-answers)

- [47. Final Result](#47-final-result)

- [48. Conclusion](#48-conclusion)

- [49. Key Takeaways](#49-key-takeaways)

- [50. Experiment Metadata](#50-experiment-metadata)

 

---

 

# 1. Experiment Overview

 

This experiment demonstrates the use of **AWS Identity and Access Management (IAM)** to implement and verify authorization controls in an AWS account.

 

The experiment begins with two independent IAM users. Each user is assigned a different service-specific permission set:

 

```text

User 1

cloud-user-ec2

        |

        +---- AmazonEC2FullAccess

        |

        +---- EC2 operations

 

 

User 2

cloud-user-s3

        |

        +---- AmazonS3FullAccess

        |

        +---- S3 operations

```

 

After the normal S3 access is verified, the experiment introduces an additional security restriction.

 

A custom policy is created that explicitly denies:

 

```text

s3:DeleteBucket

```

 

The custom policy is attached to the same S3 user while the existing:

 

```text

AmazonS3FullAccess

```

 

policy remains attached.

 

The resulting permission model is intentionally contradictory:

 

```text

AmazonS3FullAccess

        |

        +---- ALLOW S3 actions

        |

        +---- includes DeleteBucket

 

Custom Security Policy

        |

        +---- DENY s3:DeleteBucket

```

 

When the user attempts to delete an S3 bucket, AWS denies the request.

 

This provides a practical demonstration of one of the most important IAM concepts:

 

> **An explicit Deny takes precedence over an Allow.**

 

The experiment therefore moves beyond simply creating IAM users. It demonstrates how identity-based policies can be combined to create security guardrails around sensitive operations.

 

---

 

# 2. Aim

 

To create and configure AWS IAM users with service-specific permissions, verify access to Amazon EC2 and Amazon S3, and implement an explicit IAM Deny policy that prevents S3 bucket deletion while preserving general S3 access.

 

---

 

# 3. Objectives

 

The experiment has the following objectives:

 

1. Understand the purpose of AWS IAM.

2. Create IAM users.

3. Understand identity-based authorization.

4. Attach AWS managed policies to IAM users.

5. Grant EC2-specific permissions to one user.

6. Grant S3-specific permissions to another user.

7. Verify permissions through actual AWS service access.

8. Create a custom IAM policy.

9. Use an explicit `Deny` statement.

10. Restrict the `s3:DeleteBucket` action.

11. Demonstrate that explicit Deny overrides Allow.

12. Understand the security implications of broad permissions.

13. Understand the principle of least privilege.

14. Understand separation of responsibilities.

15. Understand why destructive operations may require additional controls.

16. Understand why IAM identities and AWS resources are separate objects.

17. Perform appropriate cleanup after the experiment.

 

---

 

# 4. Problem Statement

 

Cloud environments contain many resources and services. A single AWS account can contain:

 

- EC2 instances

- S3 buckets

- Databases

- Lambda functions

- VPCs

- IAM identities

- CloudWatch resources

- Load balancers

- Secrets

- Encryption keys

- Application infrastructure

 

If every user receives unrestricted access to all of these resources, a compromised account or accidental command could cause significant damage.

 

Therefore, cloud security requires **controlled authorization**.

 

The problem addressed by this experiment is:

 

> How can AWS allow a user to perform required operations while preventing a specific dangerous operation?

 

The experiment solves this using IAM policies.

 

The S3 user receives broad S3 permissions, but an explicit Deny prevents bucket deletion.

 

Conceptually:

 

```text

Required access

      +

Security restriction

      =

Controlled authorization

```

 

---

 

# 5. Security Scenario

 

Consider a simplified organization.

 

An EC2 administrator needs to work with EC2.

 

An S3 administrator needs to work with S3.

 

However, the organization does not want the S3 administrator to delete entire buckets because bucket deletion is a destructive administrative action.

 

The desired permission model is:

 

```text

                    AWS ACCOUNT

                         |

             +-----------+-----------+

             |                       |

             v                       v

      EC2 Administrator        S3 Administrator

             |                       |

             v                       v

       EC2 permissions          S3 permissions

                                     |

                                     v

                              DeleteBucket

                                     |

                                     v

                                   DENY

```

 

This is a simple but realistic example of a security guardrail.

 

---

 

# 6. AWS IAM Introduction

 

## What is IAM?

 

AWS Identity and Access Management, commonly called **IAM**, is the AWS service used to manage identities and permissions.

 

IAM allows administrators to determine:

 

- Who can access AWS.

- What actions an identity can perform.

- Which resources those actions can affect.

- Under which conditions an action is allowed.

- Which operations must be explicitly denied.

 

IAM is therefore a core component of AWS security.

 

A simplified model is:

 

```text

Identity

   |

   v

Policy

   |

   v

Authorization Decision

   |

   +---- Allow

   |

   +---- Deny

```

 

---

 

## Why IAM is important

 

Without access control, a user with credentials could potentially perform dangerous operations.

 

For example:

 

```text

Delete EC2 instance

Delete S3 bucket

Delete database

Modify security group

Change IAM permissions

Read sensitive object

```

 

IAM allows these capabilities to be separated.

 

Instead of:

 

```text

EVERY USER → EVERYTHING

```

 

the security model becomes:

 

```text

USER → REQUIRED PERMISSIONS → REQUIRED RESOURCES

```

 

This reduces unnecessary access.

 

---

 

# 7. Authentication vs Authorization

 

Authentication and authorization are related but different.

 

## Authentication

 

Authentication answers:

 

> Who are you?

 

Examples include:

 

- Username/password

- Access keys

- Federated identity

- Temporary credentials

- IAM roles

 

A successful authentication establishes an identity.

 

---

 

## Authorization

 

Authorization answers:

 

> What are you allowed to do?

 

For example:

 

```text

User authenticated

        |

        v

Request: Delete S3 bucket

        |

        v

IAM policy evaluation

        |

        v

Allowed or denied

```

 

A user can therefore successfully log into AWS but still receive:

 

```text

AccessDenied

```

 

when attempting a particular operation.

 

---

 

## Important distinction

 

```text

Authentication

    =

Identity verification

 

Authorization

    =

Permission verification

```

 

This distinction is fundamental to cloud security.

 

---

 

# 8. IAM Users

 

An IAM user is an AWS identity that can have permissions assigned to it.

 

For this experiment, two users were created.

 

## User 1

 

```text

Username:

cloud-user-ec2

```

 

Purpose:

 

```text

EC2 administration / experimentation

```

 

Policy:

 

```text

AmazonEC2FullAccess

```

 

---

 

## User 2

 

```text

Username:

cloud-user-s3

```

 

Purpose:

 

```text

S3 administration / experimentation

```

 

Policies:

 

```text

AmazonS3FullAccess

```

 

and later:

 

```text

s3-without-bucket-deletion

```

 

---

 

# 9. IAM Policies

 

An IAM policy is a JSON document describing permissions.

 

A policy can contain one or more statements.

 

Basic structure:

 

```json

{

  "Version": "2012-10-17",

  "Statement": [

    {

      "Effect": "Allow",

      "Action": "service:Action",

      "Resource": "*"

    }

  ]

}

```

 

A policy can therefore describe:

 

```text

Effect

Action

Resource

Condition

```

 

These elements form the foundation of AWS authorization.

 

---

 

# 10. Policy Elements

 

## 10.1 Version

 

Example:

 

```json

"Version": "2012-10-17"

```

 

This identifies the IAM policy language version.

 

It is part of the standard policy format.

 

---

 

## 10.2 Statement

 

A statement contains an individual permission rule.

 

Example:

 

```json

"Statement": [

  {

    "Effect": "Deny",

    "Action": "s3:DeleteBucket",

    "Resource": "*"

  }

]

```

 

Multiple statements can exist in one policy.

 

---

 

## 10.3 Effect

 

The effect determines whether an operation is:

 

```text

Allow

```

 

or:

 

```text

Deny

```

 

---

 

## 10.4 Action

 

The Action identifies an AWS API operation.

 

Examples:

 

```text

s3:ListBucket

s3:GetObject

s3:PutObject

s3:DeleteObject

s3:DeleteBucket

```

 

For EC2:

 

```text

ec2:RunInstances

ec2:StartInstances

ec2:StopInstances

ec2:TerminateInstances

```

 

---

 

## 10.5 Resource

 

The Resource identifies which AWS resources the policy applies to.

 

A wildcard:

 

```text

*

```

 

means the statement is not restricted to a specific resource.

 

In production, resource-level permissions should be scoped as narrowly as the service and use case allow.

 

---

 

## 10.6 Condition

 

A Condition can add additional requirements.

 

For example, a policy could restrict an operation based on:

 

- IP address

- Region

- Tags

- Time

- MFA

- Request attributes

 

Conditions are useful for creating context-aware authorization.

 

---

 

# 11. Allow and Deny

 

## Allow

 

An Allow statement grants permission.

 

Example:

 

```json

{

  "Effect": "Allow",

  "Action": "s3:GetObject",

  "Resource": "*"

}

```

 

This permits object retrieval where the policy applies.

 

---

 

## Deny

 

A Deny statement blocks permission.

 

Example:

 

```json

{

  "Effect": "Deny",

  "Action": "s3:DeleteBucket",

  "Resource": "*"

}

```

 

This prevents bucket deletion where the statement applies.

 

---

 

# 12. IAM Policy Evaluation Logic

 

AWS does not simply look for one Allow statement and immediately permit an action.

 

Multiple applicable policies can influence the final decision.

 

The most important rule demonstrated in this experiment is:

 

```text

Explicit Deny overrides Allow.

```

 

Consider:

 

```text

Policy A

Allow s3:DeleteBucket

 

Policy B

Deny s3:DeleteBucket

```

 

Final decision:

 

```text

DENY

```

 

The conceptual flow is:

 

```text

                Request

                   |

                   v

          Policy evaluation

                   |

        +----------+----------+

        |                     |

        v                     v

      ALLOW                  DENY

        |                     |

        +----------+----------+

                   |

                   v

             Explicit Deny

                   |

                   v

                REJECT

```

 

Therefore:

 

```text

ALLOW + EXPLICIT DENY

        =

DENIED

```

 

---

 

# 13. AWS Managed Policies

 

AWS provides managed policies for common permission requirements.

 

Two policies used in this experiment are:

 

```text

AmazonEC2FullAccess

AmazonS3FullAccess

```

 

These are AWS managed policies.

 

They provide broad access to the corresponding services.

 

---

 

## Why use managed policies in the lab?

 

Managed policies make the experiment easier to understand because the initial authorization behavior is straightforward.

 

For User 1:

 

```text

AmazonEC2FullAccess

```

 

provides broad EC2 access.

 

For User 2:

 

```text

AmazonS3FullAccess

```

 

provides broad S3 access.

 

The lab then adds a custom policy to demonstrate a more precise security control.

 

---

 

# 14. Custom IAM Policies

 

A custom policy is created by the administrator for a specific requirement.

 

In this experiment:

 

```text

Policy Name:

s3-without-bucket-deletion

```

 

Purpose:

 

```text

Allow normal S3 access

BUT

deny bucket deletion

```

 

The custom policy is:

 

```json

{

  "Version": "2012-10-17",

  "Statement": [

    {

      "Effect": "Deny",

      "Action": "s3:DeleteBucket",

      "Resource": "*"

    }

  ]

}

```

 

---

 

# 15. Experiment Architecture

 

The overall experiment can be represented as:

 

```text

                           AWS ACCOUNT

                                |

                                v

                         +-------------+

                         |     IAM     |

                         +------+------+

                                |

                 +--------------+--------------+

                 |                             |

                 v                             v

        +------------------+          +------------------+

        | cloud-user-ec2   |          | cloud-user-s3    |

        +--------+---------+          +--------+---------+

                 |                             |

                 v                             v

    AmazonEC2FullAccess             AmazonS3FullAccess

                 |                             |

                 v                             v

                EC2                           S3

                                               |

                                               v

                                  +-------------------------+

                                  | Custom Security Policy  |

                                  +------------+------------+

                                               |

                                               v

                                  Deny s3:DeleteBucket

                                               |

                                               v

                                      Bucket deletion

                                               |

                                               v

                                           DENIED

```

 

---

 

# 16. Part A — EC2 IAM User

 

## Objective

 

Create an IAM user with EC2 permissions and verify that the user can access and operate Amazon EC2.

 

---

 

## User configuration

 

Username:

 

```text

cloud-user-ec2

```

 

Policy:

 

```text

AmazonEC2FullAccess

```

 

---

 

## Step 1 — Open IAM

 

Navigate to:

 

```text

AWS Console

    |

    +---- IAM

          |

          +---- Users

```

 

Choose:

 

```text

Create user

```

 

---

 

## Step 2 — Specify user details

 

Enter:

 

```text

cloud-user-ec2

```

 

The username identifies the IAM identity.

 

---

 

## Step 3 — Set permissions

 

Choose:

 

```text

Attach policies directly

```

 

Search:

 

```text

AmazonEC2FullAccess

```

 

Select the policy.

 

---

 

## Step 4 — Review

 

Confirm that:

 

```text

cloud-user-ec2

```

 

has:

 

```text

AmazonEC2FullAccess

```

 

Create the user.

 

---

 

# 17. EC2 Permission Verification

 

The purpose of this step is to verify that the IAM policy actually controls access.

 

The user was used to access the EC2 console.

 

The experiment included creating an EC2 instance.

 

This demonstrated that the user had sufficient EC2 permissions.

 

The instance was subsequently terminated during cleanup.

 

---

 

## Authorization flow

 

```text

cloud-user-ec2

        |

        v

AmazonEC2FullAccess

        |

        v

EC2 authorization

        |

        v

Operation permitted

```

 

Expected:

 

```text

EC2 Access = ALLOWED

```

 

---

 

## What this proves

 

The test demonstrates that permissions attached to an IAM identity can determine whether the identity can interact with an AWS service.

 

It also demonstrates why service-specific permission assignments are useful.

 

A user intended for EC2 does not need to be given S3 permissions merely because both services exist in the same AWS account.

 

---

 

# 18. Part B — S3 IAM User

 

## Objective

 

Create an IAM user with S3 permissions and verify S3 access.

 

---

 

## User configuration

 

Username:

 

```text

cloud-user-s3

```

 

Policy:

 

```text

AmazonS3FullAccess

```

 

---

 

## Step 1 — Create the user

 

Navigate to:

 

```text

IAM

  |

  +---- Users

        |

        +---- Create user

```

 

Enter:

 

```text

cloud-user-s3

```

 

---

 

## Step 2 — Attach S3 policy

 

Select:

 

```text

Attach policies directly

```

 

Search:

 

```text

AmazonS3FullAccess

```

 

Select it.

 

---

 

## Step 3 — Create

 

Review the configuration.

 

Expected permission:

 

```text

AmazonS3FullAccess

```

 

Create the user.

 

---

 

# 19. S3 Permission Verification

 

Sign in using the S3 user's credentials.

 

Navigate to:

 

```text

Amazon S3

```

 

The user should be able to access S3 resources permitted by the attached policy.

 

A temporary test bucket can be created for the experiment.

 

Example:

 

```text

nitanshu-s3-bucket-2

```

 

The exact bucket name can vary because S3 bucket names are globally unique.

 

---

 

## Expected behavior before the Deny policy

 

With:

 

```text

AmazonS3FullAccess

```

 

the user has broad S3 permissions.

 

Conceptually:

 

```text

Create bucket       → ALLOWED

List bucket         → ALLOWED

Upload object       → ALLOWED

Read object         → ALLOWED

Delete object       → ALLOWED

Delete bucket       → ALLOWED

```

 

The important point is that bucket deletion is initially permitted by the broad policy.

 

This establishes the baseline.

 

---

 

# 20. Part C — Explicit Deny

 

The final part changes the authorization behavior.

 

The requirement is:

 

> The user should retain S3 access but must not be allowed to delete an S3 bucket.

 

The solution is an explicit Deny.

 

---

 

## Desired final state

 

```text

S3 operations

      |

      +---- Normal operations → ALLOWED

      |

      +---- DeleteBucket      → DENIED

```

 

This is preferable to simply removing all S3 access because the user still needs to work with S3.

 

---

 

# 21. Custom S3 Deny Policy

 

Policy name:

 

```text

s3-without-bucket-deletion

```

 

Policy JSON:

 

```json

{

  "Version": "2012-10-17",

  "Statement": [

    {

      "Effect": "Deny",

      "Action": "s3:DeleteBucket",

      "Resource": "*"

    }

  ]

}

```

 

Attach this policy to:

 

```text

cloud-user-s3

```

 

---

 

# 22. Detailed JSON Explanation

 

## Line 1 — Version

 

```json

"Version": "2012-10-17"

```

 

This specifies the policy language version.

 

---

 

## Statement

 

```json

"Statement": [

```

 

This begins the list of permission statements.

 

---

 

## Effect

 

```json

"Effect": "Deny"

```

 

This is the security control.

 

It says that the specified operation must be rejected.

 

---

 

## Action

 

```json

"Action": "s3:DeleteBucket"

```

 

This identifies the specific S3 API operation being blocked.

 

It is much more precise than denying all S3 operations.

 

---

 

## Resource

 

```json

"Resource": "*"

```

 

This applies the rule broadly to S3 bucket resources to which the action is applicable.

 

A production policy should generally be evaluated for whether a narrower resource scope is possible.

 

---

 

# 23. Why Full S3 Access Was Retained

 

This is a deliberate part of the experiment.

 

The S3 user initially has:

 

```text

AmazonS3FullAccess

```

 

This means the user has an Allow path for the bucket deletion operation.

 

If we immediately removed that policy and replaced it with only the Deny policy, we would not demonstrate the conflict between Allow and Deny.

 

Instead, the experiment keeps:

 

```text

AmazonS3FullAccess

```

 

and adds:

 

```text

s3-without-bucket-deletion

```

 

The final permission set is therefore:

 

```text

cloud-user-s3

    |

    +---- AmazonS3FullAccess

    |          |

    |          +---- broad S3 Allow

    |

    +---- s3-without-bucket-deletion

               |

               +---- explicit Deny DeleteBucket

```

 

This creates the exact condition required to demonstrate IAM precedence.

 

---

 

# 24. Bucket Deletion Verification

 

The final verification attempts to delete the test bucket.

 

The user navigates to:

 

```text

Amazon S3

    |

    +---- Test Bucket

          |

          +---- Delete

```

 

AWS requires confirmation of the bucket name.

 

After entering the bucket name, the deletion request is submitted.

 

Because the custom policy contains:

 

```text

"Deny"

```

 

for:

 

```text

"s3:DeleteBucket"

```

 

AWS rejects the operation.

 

The observed result is an authorization error indicating that the user does not have permission to delete the bucket.

 

---

 

## Expected result

 

```text

DeleteBucket

     |

     v

Explicit Deny

     |

     v

Access Denied

```

 

This is the key verification of the experiment.

 

---

 

# 25. Security Analysis

 

The experiment demonstrates that IAM is not simply an on/off switch for an AWS service.

 

Permissions can be combined to create more sophisticated authorization behavior.

 

For example:

 

```text

Broad permission

       +

Specific security restriction

       =

Controlled access

```

 

This is especially useful for protecting sensitive operations.

 

---

 

## Example

 

Suppose an S3 administrator needs:

 

```text

Read objects

Upload objects

Modify objects

List buckets

```

 

but should not be able to:

 

```text

Delete an entire bucket

```

 

A security control can explicitly deny the destructive action.

 

This illustrates how authorization can be aligned with business responsibilities.

 

---

 

# 26. Least Privilege

 

The principle of least privilege means:

 

> An identity should receive only the permissions necessary to perform its intended task.

 

In this experiment, the initial Full Access policies are intentionally broad for demonstration.

 

In a production environment, a better approach could be:

 

```text

User

 |

 +---- List required bucket

 |

 +---- Read required objects

 |

 +---- Write required objects

 |

 +---- No unnecessary delete permissions

```

 

Rather than:

 

```text

User

 |

 +---- Full S3 access

```

 

---

 

## Why least privilege matters

 

Excessive permissions increase the impact of:

 

- Stolen credentials

- Compromised accounts

- Malware

- Insider threats

- Accidental commands

- Misconfigured automation

 

If an identity has fewer permissions, the potential blast radius is reduced.

 

---

 

# 27. Separation of Duties

 

The experiment also demonstrates a basic form of separation of duties.

 

There are two identities:

 

```text

cloud-user-ec2

```

 

and:

 

```text

cloud-user-s3

```

 

They have different responsibilities.

 

This avoids treating every user as a universal administrator.

 

In enterprise environments, separation of duties can be much more detailed.

 

For example:

 

```text

Developer

    |

    +---- Application deployment

 

Database Administrator

    |

    +---- Database management

 

Security Administrator

    |

    +---- IAM and security controls

 

Network Administrator

    |

    +---- VPC and network management

```

 

Each role receives appropriate permissions.

 

---

 

# 28. Defense in Depth

 

Defense in depth means using multiple security controls instead of relying on one control.

 

In this experiment:

 

```text

IAM identity

     +

Service-specific policy

     +

Custom explicit Deny

     =

Additional protection

```

 

The explicit Deny acts as a guardrail.

 

Even though a broad Allow exists, the destructive operation is still blocked.

 

---

 

# 29. Destructive Action Protection

 

Not all actions have equal security impact.

 

Consider:

 

```text

Read object

```

 

versus:

 

```text

Delete bucket

```

 

The second operation can have much greater consequences.

 

Therefore, security designs often pay special attention to destructive operations.

 

Examples:

 

```text

s3:DeleteBucket

ec2:TerminateInstances

rds:DeleteDBInstance

iam:DeleteUser

iam:DeleteRole

kms:ScheduleKeyDeletion

```

 

The exact permissions required depend on the environment.

 

The experiment uses:

 

```text

s3:DeleteBucket

```

 

as the destructive operation.

 

---

 

# 30. Managed vs Custom Policies

 

## AWS Managed Policy

 

Example:

 

```text

AmazonS3FullAccess

```

 

Advantages:

 

- Easy to use.

- AWS maintains the policy.

- Useful for learning and common use cases.

- Quickly provides a known permission set.

 

Disadvantages:

 

- Can be broader than necessary.

- May grant permissions that are not required for a particular job.

- Less precise than a carefully designed custom policy.

 

---

 

## Custom Policy

 

Example:

 

```text

s3-without-bucket-deletion

```

 

Advantages:

 

- Tailored to a specific requirement.

- Can target individual actions.

- Useful for security guardrails.

- Gives administrators fine-grained control.

 

Disadvantages:

 

- Requires careful design.

- Requires maintenance.

- Incorrect policies can accidentally grant or deny access.

 

---

 

# 31. IAM Users vs IAM Roles

 

An IAM user represents an identity with long-term credentials.

 

IAM roles are commonly used to provide temporary credentials to AWS services, applications, workloads, and users through federation.

 

A simplified distinction:

 

```text

IAM User

    |

    +---- Long-term identity

 

IAM Role

    |

    +---- Assumable identity

    |

    +---- Temporary credentials

```

 

For modern AWS architectures, roles are generally preferred over embedding long-term access keys into applications.

 

The experiment uses IAM users because the laboratory objective specifically concerns user creation and user permissions.

 

---

 

# 32. Why IAM Users Should Be Minimized

 

For human access, organizations increasingly prefer federation and centralized identity providers.

 

IAM users can introduce credential-management challenges.

 

Potential risks include:

 

- Forgotten passwords

- Long-lived access keys

- Credential leakage

- Poor rotation practices

- Shared credentials

- Excessive permissions

 

For a classroom exercise, IAM users are useful because their permissions are easy to demonstrate directly.

 

For production environments, stronger identity-management patterns should be considered.

 

---

 

# 33. Security Risks of Full Access

 

The policies used in this experiment are intentionally broad.

 

For example:

 

```text

AmazonEC2FullAccess

```

 

and:

 

```text

AmazonS3FullAccess

```

 

can provide much more permission than a specific application needs.

 

If credentials are compromised, an attacker may be able to perform many operations.

 

Therefore:

 

```text

Full Access

```

 

should not automatically be considered a best practice.

 

It is primarily used here to establish a clear baseline and then demonstrate how an explicit Deny can restrict one sensitive operation.

 

---

 

# 34. Resource Scope and Wildcards

 

The policy uses:

 

```json

"Resource": "*"

```

 

This is convenient for a laboratory because the Deny applies broadly.

 

However, the wildcard should be treated carefully.

 

A more restrictive design may attempt to scope access to:

 

```text

Specific bucket

Specific object prefix

Specific resources

Specific account

Specific conditions

```

 

The exact ARN structure depends on the AWS service and action.

 

The principle is:

 

```text

Broad scope

    ↓

More potential impact

 

Narrow scope

    ↓

Smaller blast radius

```

 

---

 

# 35. Conditions and Advanced Restrictions

 

IAM policies can also use conditions.

 

A condition can make a permission dependent on additional context.

 

Examples of conditions include:

 

```text

Source IP

MFA presence

AWS Region

Resource tags

Request properties

Principal attributes

```

 

Conceptually:

 

```text

Allow operation

      +

Condition satisfied

      =

Allowed

```

 

If the condition is not satisfied:

 

```text

Permission does not apply

```

 

Conditions are useful for implementing more advanced security policies.

 

---

 

# 36. Policy Evaluation Examples

 

## Example 1 — Only Allow

 

```text

Policy A:

ALLOW s3:DeleteBucket

```

 

Result:

 

```text

ALLOWED

```

 

assuming no other applicable restriction blocks it.

 

---

 

## Example 2 — Only Deny

 

```text

Policy A:

DENY s3:DeleteBucket

```

 

Result:

 

```text

DENIED

```

 

---

 

## Example 3 — Allow + Explicit Deny

 

```text

Policy A:

ALLOW s3:DeleteBucket

 

Policy B:

DENY s3:DeleteBucket

```

 

Result:

 

```text

DENIED

```

 

This is exactly what the experiment demonstrates.

 

---

 

## Example 4 — Allow S3 but Deny Bucket Deletion

 

```text

AmazonS3FullAccess

        +

Deny s3:DeleteBucket

```

 

Result:

 

```text

Normal S3 operations

        → generally allowed by the broad policy

 

Bucket deletion

        → denied by explicit Deny

```

 

---

 

# 37. Practical Observations

 

The experiment produced several important observations.

 

### Observation 1

 

The EC2 user could access EC2 because the required EC2 policy was attached.

 

### Observation 2

 

The S3 user could access S3 because the S3 Full Access policy was attached.

 

### Observation 3

 

The S3 user initially had broad S3 permissions.

 

### Observation 4

 

A custom Deny policy was created.

 

### Observation 5

 

The custom policy specifically targeted:

 

```text

s3:DeleteBucket

```

 

### Observation 6

 

The broad S3 policy was intentionally retained.

 

### Observation 7

 

Bucket deletion was attempted.

 

### Observation 8

 

AWS returned an authorization failure.

 

### Observation 9

 

This verified that the explicit Deny overrides the Allow.

 

---

 

# 38. Verification Matrix

 

| Test Case | Identity | Permission | Expected Result | Observed Result |

|---|---|---|---|---|

| EC2 access | `cloud-user-ec2` | `AmazonEC2FullAccess` | Allowed | Successful |

| EC2 operation | `cloud-user-ec2` | EC2 permissions | Allowed | Successful |

| S3 access | `cloud-user-s3` | `AmazonS3FullAccess` | Allowed | Successful |

| S3 normal operations | `cloud-user-s3` | S3 permissions | Allowed | Successful |

| Bucket deletion before restriction | `cloud-user-s3` | Full S3 access | Allowed | Permission available |

| Bucket deletion after restriction | `cloud-user-s3` | Explicit Deny | Denied | Access Denied |

| EC2 cleanup | Account administrator | Terminate instance | Successful | Completed |

 

---

 

# 39. Screenshot Evidence

 

Screenshots should be added to the final experiment documentation.

 

Recommended evidence:

 

### Screenshot 1 — IAM User 1 Creation

 

Show:

 

```text

cloud-user-ec2

```

 

---

 

### Screenshot 2 — User 1 Permissions

 

Show:

 

```text

AmazonEC2FullAccess

```

 

---

 

### Screenshot 3 — EC2 Verification

 

Show successful EC2 access or the EC2 instance created during the experiment.

 

---

 

### Screenshot 4 — IAM User 2

 

Show:

 

```text

cloud-user-s3

```

 

with:

 

```text

AmazonS3FullAccess

```

 

---

 

### Screenshot 5 — S3 Verification

 

Show the S3 console and the test bucket.

 

---

 

### Screenshot 6 — Custom Policy

 

Show:

 

```text

s3-without-bucket-deletion

```

 

---

 

### Screenshot 7 — JSON Policy

 

Show the policy containing:

 

```json

"Effect": "Deny",

"Action": "s3:DeleteBucket"

```

 

---

 

### Screenshot 8 — Policy Attached

 

Show both:

 

```text

AmazonS3FullAccess

```

 

and:

 

```text

s3-without-bucket-deletion

```

 

attached to the S3 user.

 

---

 

### Screenshot 9 — Access Denied

 

Show the S3 bucket deletion failure.

 

This is the most important verification screenshot.

 

---

 

### Screenshot 10 — Cleanup

 

Show the EC2 instance being terminated or the final cleaned-up state.

 

---

 

# 40. Cleanup and Cost Control

 

AWS resources should be deleted or terminated when the experiment is finished.

 

This is important for two reasons:

 

1. Security.

2. Cost control.

 

Resources that may continue to consume resources or generate charges include:

 

```text

EC2 instances

EBS volumes

Elastic IPs

NAT Gateways

S3 storage

Data transfer

```

 

IAM users and policies are not equivalent to compute resources.

 

However, temporary identities should also be removed when they are no longer required.

 

---

 

## EC2 cleanup

 

The EC2 instance created for the experiment was terminated.

 

The final state showed that termination had been successfully initiated and the instance entered the shutting-down state.

 

---

 

## S3 cleanup

 

Temporary buckets should be removed after screenshots are captured.

 

If the S3 user has an explicit Deny on bucket deletion, the bucket may need to be deleted using an identity that is authorized to perform that operation.

 

This is itself an important practical lesson:

 

> Security controls can also affect cleanup operations.

 

---

 

# 41. Important Resource Ownership Concept

 

A very important AWS concept is:

 

> **An IAM user is not the owner object that controls the lifetime of the AWS resources created by that user.**

 

Suppose:

 

```text

IAM User

   |

   +---- creates EC2 instance

```

 

If the IAM user is deleted:

 

```text

IAM User → deleted

```

 

the EC2 instance does not automatically disappear.

 

The instance is an independent AWS resource.

 

Similarly:

 

```text

IAM User

   |

   +---- creates S3 bucket

```

 

Deleting the user does not automatically delete the bucket.

 

Therefore:

 

```text

Delete IAM user

        ≠

Delete AWS resources

```

 

This is important for both cost management and security.

 

---

 

# 42. Common Mistakes

 

## Mistake 1 — Removing the S3 Full Access policy too early

 

If the goal is to demonstrate:

 

```text

Allow vs Deny

```

 

the broad Allow should remain during the verification.

 

---

 

## Mistake 2 — Using Allow instead of Deny

 

The custom policy must contain:

 

```json

"Effect": "Deny"

```

 

not:

 

```json

"Effect": "Allow"

```

 

---

 

## Mistake 3 — Using the wrong action

 

The experiment targets:

 

```text

s3:DeleteBucket

```

 

This is different from:

 

```text

s3:DeleteObject

```

 

Deleting an object and deleting an entire bucket are different operations.

 

---

 

## Mistake 4 — Forgetting that bucket names are globally unique

 

S3 bucket names must follow AWS naming requirements and have global uniqueness within the S3 namespace.

 

---

 

## Mistake 5 — Forgetting cleanup

 

Running EC2 instances and other billable infrastructure should not be left running unnecessarily.

 

---

 

## Mistake 6 — Assuming deleting a user deletes resources

 

It does not.

 

Resources must be cleaned up separately.

 

---

 

# 43. Troubleshooting

 

## Problem: User cannot access EC2

 

Check:

 

```text

IAM user

    |

    +---- Attached policies

```

 

Verify:

 

```text

AmazonEC2FullAccess

```

 

is attached.

 

Also verify that the user is actually signed into the intended AWS account.

 

---

 

## Problem: User cannot access S3

 

Check:

 

```text

AmazonS3FullAccess

```

 

and verify that the correct identity is being used.

 

---

 

## Problem: Bucket deletion is unexpectedly allowed

 

Check:

 

1. Is the custom policy attached?

2. Does the policy contain `Effect: Deny`?

3. Does it target `s3:DeleteBucket`?

4. Is the policy active?

5. Are you signed in as the correct user?

 

---

 

## Problem: Bucket deletion is unexpectedly denied

 

This can occur if the custom Deny is active.

 

Use an administrator or another authorized identity for cleanup.

 

---

 

## Problem: EC2 still exists after user deletion

 

This is expected.

 

IAM user deletion does not terminate EC2 instances.

 

---

 

# 44. Security Best Practices

 

A production AWS environment should follow stronger controls than this simple classroom experiment.

 

Recommended practices include:

 

### 1. Use least privilege

 

Avoid unnecessary Full Access policies.

 

### 2. Prefer roles where appropriate

 

Use IAM roles and temporary credentials for AWS workloads.

 

### 3. Enable MFA

 

Especially for privileged identities.

 

### 4. Avoid sharing credentials

 

Each identity should have appropriate individual accountability.

 

### 5. Monitor access

 

Use AWS logging and auditing services where appropriate.

 

### 6. Review permissions regularly

 

Remove unused permissions.

 

### 7. Protect destructive operations

 

Use appropriate policy controls and organizational guardrails.

 

### 8. Scope resources

 

Avoid unnecessary:

 

```text

Resource: "*"

```

 

when a narrower scope is practical.

 

### 9. Rotate and protect credentials

 

Long-lived credentials should be managed carefully.

 

### 10. Separate administrative responsibilities

 

Do not give every user unrestricted administrative access.

 

---

 

# 45. Real-World Application

 

The experiment represents a simplified version of a real cloud security requirement.

 

Imagine an organization where:

 

```text

Storage Team

```

 

needs S3 access.

 

They should be able to:

 

```text

Upload files

Download files

Manage objects

```

 

But the organization wants to prevent accidental deletion of entire storage buckets.

 

A policy-based security design can create this boundary.

 

Similarly, an EC2 administrator can be given EC2 permissions without automatically receiving S3 permissions.

 

This reduces the blast radius of compromised credentials.

 

---

 

## Example enterprise design

 

```text

                    AWS ACCOUNT

                         |

       +-----------------+-----------------+

       |                 |                 |

       v                 v                 v

   Developers        Operations        Security

       |                 |                 |

       v                 v                 v

   App access         EC2 access        IAM access

       |                 |

       v                 v

    Limited             Limited

    S3 access            EC2 access

```

 

Additional guardrails can then be applied using:

 

```text

IAM policies

IAM roles

Permission boundaries

Service Control Policies

Resource policies

Conditions

MFA

Logging

Monitoring

```

 

---

 

# 46. Viva Questions and Answers

 

## Q1. What is AWS IAM?

 

AWS IAM is the AWS service used to manage identities and control access to AWS resources.

 

---

 

## Q2. What is authentication?

 

Authentication is the process of verifying an identity.

 

---

 

## Q3. What is authorization?

 

Authorization determines whether an authenticated identity is permitted to perform a requested action.

 

---

 

## Q4. What is an IAM policy?

 

An IAM policy is a JSON document that defines permissions.

 

---

 

## Q5. What is an IAM user?

 

An IAM user is an AWS identity that can have permissions assigned to it.

 

---

 

## Q6. Why were two IAM users created?

 

To demonstrate separate service-specific permissions.

 

---

 

## Q7. What policy was assigned to User 1?

 

```text

AmazonEC2FullAccess

```

 

---

 

## Q8. What policy was assigned to User 2 initially?

 

```text

AmazonS3FullAccess

```

 

---

 

## Q9. Why was `AmazonS3FullAccess` retained?

 

To demonstrate the conflict between an existing Allow and a new explicit Deny.

 

---

 

## Q10. What custom policy was created?

 

```text

s3-without-bucket-deletion

```

 

---

 

## Q11. What action was denied?

 

```text

s3:DeleteBucket

```

 

---

 

## Q12. What happens if Allow and explicit Deny both apply?

 

The explicit Deny wins.

 

---

 

## Q13. Why is bucket deletion considered sensitive?

 

It is a destructive operation that can remove an entire storage container and potentially affect application data.

 

---

 

## Q14. What is least privilege?

 

Least privilege means providing only the permissions necessary for the intended task.

 

---

 

## Q15. Is Full Access recommended for production?

 

Generally, no. Production permissions should be scoped to the minimum required actions and resources.

 

---

 

## Q16. What is the difference between managed and custom policies?

 

Managed policies are reusable policies maintained by AWS or managed centrally, while custom policies are created specifically for a requirement.

 

---

 

## Q17. What is an explicit Deny?

 

It is a policy rule that directly blocks an operation and overrides applicable Allows.

 

---

 

## Q18. What does `s3:DeleteBucket` represent?

 

It represents the S3 API action for deleting a bucket.

 

---

 

## Q19. What does `Resource: "*"` mean?

 

It represents a wildcard resource scope. In this laboratory policy it is used to apply the Deny broadly.

 

---

 

## Q20. Does deleting an IAM user delete their EC2 instance?

 

No.

 

---

 

## Q21. Does deleting an IAM user delete their S3 bucket?

 

No.

 

---

 

## Q22. Why is cleanup important?

 

To reduce security exposure and prevent unnecessary resource usage or charges.

 

---

 

## Q23. What is defense in depth?

 

Defense in depth is the use of multiple security controls so that failure or excessive permission in one layer does not automatically result in unrestricted access.

 

---

 

## Q24. What is separation of duties?

 

It means distributing responsibilities so that one identity does not automatically have every administrative capability.

 

---

 

## Q25. Why are IAM roles often preferred for workloads?

 

Roles can provide temporary credentials and reduce the need for long-lived access keys.

 

---

 

# 47. Final Result

 

The final experiment produced the following authorization model:

 

```text

+------------------------------------------------+

|                AWS ACCOUNT                     |

+------------------------------------------------+

                    |

          +---------+---------+

          |                   |

          v                   v

+----------------+   +----------------+

| cloud-user-ec2 |   | cloud-user-s3  |

+----------------+   +----------------+

        |                    |

        v                    v

AmazonEC2FullAccess   AmazonS3FullAccess

        |                    |

        v                    v

       EC2                  S3

                             |

                             v

                  s3-without-bucket-deletion

                             |

                             v

                    DENY DeleteBucket

                             |

                             v

                         Access Denied

```

 

---

 

# 48. Conclusion

 

This experiment successfully demonstrated how AWS IAM can be used to control access to cloud services and enforce security restrictions.

 

First, an IAM user named:

 

```text

cloud-user-ec2

```

 

was created and assigned:

 

```text

AmazonEC2FullAccess

```

 

The user was then used to verify EC2 access.

 

Second, an IAM user named:

 

```text

cloud-user-s3

```

 

was created and assigned:

 

```text

AmazonS3FullAccess

```

 

The user was used to verify S3 access.

 

Finally, a custom policy named:

 

```text

s3-without-bucket-deletion

```

 

was created.

 

The policy contained an explicit Deny:

 

```json

{

  "Effect": "Deny",

  "Action": "s3:DeleteBucket",

  "Resource": "*"

}

```

 

The custom policy was attached while the original S3 Full Access policy remained present.

 

When bucket deletion was attempted, AWS returned an authorization failure.

 

This demonstrated:

 

```text

AmazonS3FullAccess

        |

        +---- ALLOW

        |

        +---- DeleteBucket

 

Custom policy

        |

        +---- DENY

        |

        +---- DeleteBucket

 

Final

        |

        +---- DENIED

```

 

Therefore, the experiment successfully demonstrated the fundamental IAM rule:

 

> **An explicit Deny takes precedence over an Allow.**

 

The experiment also highlighted broader cloud security principles such as:

 

- Least privilege

- Separation of responsibilities

- Defense in depth

- Protection of destructive operations

- Policy-based authorization

- Resource cleanup

- Identity/resource separation

 

---

 

# 49. Key Takeaways

 

> 🔐 **IAM controls access to AWS resources.**

 

> 👤 **IAM users represent identities.**

 

> 📜 **Policies define permissions.**

 

> ✅ **Allow grants access.**

 

> 🚫 **Explicit Deny blocks access.**

 

> ⚡ **Explicit Deny overrides Allow.**

 

> 🎯 **Least privilege reduces unnecessary access.**

 

> 🛡️ **Sensitive operations can be protected with additional policy controls.**

 

> 🧩 **Managed policies are convenient but may be broad.**

 

> 📝 **Custom policies provide fine-grained control.**

 

> 🧹 **AWS resources must be cleaned up separately from IAM identities.**

 

> 💡 **Deleting an IAM user does not automatically delete resources created by that user.**

 

---

 

# 50. Experiment Metadata

 

| Field | Value |

|---|---|

| Experiment Number | 2 |

| Subject | Cloud Security |

| Cloud Provider | AWS |

| Primary Service | IAM |

| Supporting Service 1 | Amazon EC2 |

| Supporting Service 2 | Amazon S3 |

| User 1 | `cloud-user-ec2` |

| User 1 Policy | `AmazonEC2FullAccess` |

| User 2 | `cloud-user-s3` |

| User 2 Initial Policy | `AmazonS3FullAccess` |

| User 2 Security Policy | `s3-without-bucket-deletion` |

| Restricted Action | `s3:DeleteBucket` |

| Security Mechanism | Explicit Deny |

| Core Principle | Explicit Deny overrides Allow |

| EC2 Verification | Successful |

| S3 Verification | Successful |

| DeleteBucket Verification | Access Denied |

| EC2 Cleanup | Completed |

| Temporary S3 Cleanup | Performed/required after evidence capture |

 

---

 

# 📸 Screenshot Placement Guide

 

When using this README in the GitHub repository, screenshots can be added under an evidence directory such as:

 

```text

experiment-2/

├── README.md

├── screenshots/

│   ├── 01-user1-created.png

│   ├── 02-user1-ec2-policy.png

│   ├── 03-ec2-verification.png

│   ├── 04-user2-created.png

│   ├── 05-user2-s3-policy.png

│   ├── 06-s3-verification.png

│   ├── 07-custom-deny-policy.png

│   ├── 08-policy-json.png

│   ├── 09-access-denied.png

│   └── 10-cleanup.png

└── ...

```

 

A GitHub README can then display them using Markdown:

 

```markdown

## 📸 Evidence

 

### User 1 — EC2 Permissions

 

![User 1 EC2 permissions](screenshots/02-user1-ec2-policy.png)

 

### User 2 — S3 Permissions

 

![User 2 S3 permissions](screenshots/05-user2-s3-policy.png)

 

### Explicit Deny Policy

 

![Explicit Deny](screenshots/08-policy-json.png)

 

### Access Denied Verification

 

![Access Denied](screenshots/09-access-denied.png)

```

 

---

 

# 🧾 Final One-Line Summary

 

> **Experiment 2 demonstrated AWS IAM-based authorization by creating separate EC2 and S3 users, verifying service-specific permissions, and enforcing an explicit `s3:DeleteBucket` Deny to prove that a security Deny overrides an existing Allow.**

 

---

 

<div align="center">

 

# 🔐 AWS IAM SECURITY LAB

 

### Experiment 2 — Authorization & Policy Enforcement

 

**Cloud Security • IAM • EC2 • S3 • Explicit Deny • Least Privilege**

 

**Completed by Nitanshu Tak**

 

</div>
