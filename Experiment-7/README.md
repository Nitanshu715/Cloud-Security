# Experiment 7: Remediating an Incident by Using AWS Config and Lambda

## Overview

This experiment demonstrates how AWS Config can monitor Amazon EC2 security group configurations and work with AWS Lambda to automatically remediate unwanted inbound rule changes.

A security incident is simulated by modifying the inbound rules of the `LabSG1` security group in the `Lab VPC`. AWS Config monitors the security group and uses a custom rule to invoke a pre-created Lambda function. The Lambda function removes unwanted inbound permissions and restores the desired configuration. Amazon CloudWatch Logs are then used to verify the remediation activity.

## Objectives

- Explain how IAM roles grant AWS services access to other AWS services.
- Configure AWS Config to monitor EC2 security group resources.
- Create a custom AWS Config rule that invokes a pre-created Lambda function.
- Simulate an unwanted security group configuration change.
- Verify automatic remediation of the security group.
- Analyze CloudWatch Logs to verify Lambda remediation activity.

## AWS Services Used

- AWS Identity and Access Management (IAM)
- AWS Config
- AWS Lambda
- Amazon EC2
- Amazon VPC
- Amazon CloudWatch Logs

## Scenario

The lab environment contains two pre-provisioned IAM roles, a Lambda function, a default VPC with a default security group, and a custom VPC named `Lab VPC` containing the `LabSG1` security group.

AWS Config is configured to continuously monitor EC2 security group resources. The `LabSG1` security group is intentionally modified by adding inbound rules for HTTP, HTTPS, SMTPS, and IMAPS.

The custom AWS Config rule named `EC2SecurityGroup` invokes the pre-created Lambda function when monitored resources are evaluated. The Lambda function compares the security group's inbound permissions with the required configuration and removes the unwanted SMTPS and IMAPS permissions.

CloudWatch Logs are used to verify that the Lambda function performed the remediation.

## Task 1: Examining and Updating IAM Roles

### AwsConfigLambdaSGRole

The `AwsConfigLambdaSGRole` IAM role contains the `awsconfig_lambda_ec2_sg_role_policy` policy.

The policy provides permissions for:

- `logs:CreateLogGroup`
- `logs:CreateLogStream`
- `logs:PutLogEvents`
- `config:PutEvaluations`
- `ec2:DescribeSecurityGroups`
- `ec2:AuthorizeSecurityGroupIngress`
- `ec2:RevokeSecurityGroupIngress`

These permissions allow the Lambda function to write CloudWatch Logs, submit AWS Config evaluations, inspect EC2 security groups, and add or remove inbound security group permissions.

### AwsConfigRole

The `AwsConfigRole` IAM role is used by AWS Config.

The existing `S3Access` policy remains attached to the role. The `AWS_ConfigRole` managed policy is also attached so that AWS Config has the required permissions to monitor resources.

## Task 2: Setting Up AWS Config

AWS Config is configured with the following settings:

| Setting | Value |
|---|---|
| Recording strategy | Specific resource types |
| Resource type | AWS EC2 SecurityGroup |
| Frequency | Continuous |
| IAM role | AwsConfigRole |
| Delivery channel | Default S3 bucket settings |

The AWS Config Resource Inventory is then examined to verify that EC2 security group resources are being monitored.

AWS Config may also display related resource types because related resources can affect the behavior of the monitored resources.

## Task 3: Modifying the Monitored Security Group

The `LabSG1` security group in `Lab VPC` is modified to simulate a security incident.

The inbound rules are configured as follows:

| Rule | Protocol / Port | Source |
|---|---|---|
| HTTP | TCP / 80 | Anywhere-IPv4 |
| HTTPS | TCP / 443 | Anywhere-IPv4 |
| SMTPS | TCP / 465 | Anywhere-IPv4 |
| IMAPS | TCP / 993 | Anywhere-IPv4 |

The SMTPS and IMAPS rules represent unwanted permissions that will later be removed by the Lambda remediation function.

## Task 4: Creating the AWS Config Rule

A custom AWS Config rule is created to invoke the pre-created Lambda function.

The rule is configured with the following settings:

| Setting | Value |
|---|---|
| Rule name | EC2SecurityGroup |
| Description | Restrict inbound ports to HTTP and HTTPS |
| Trigger type | When configuration changes |
| Scope of changes | Resources |
| Resource type | AWS EC2 SecurityGroup |
| Parameter key | debug |
| Parameter value | true |

The Lambda function ARN is obtained from the AWS Details section of the lab environment.

The `EC2SecurityGroup` rule evaluates the monitored security groups. After the initial evaluation completes, the resources become compliant and the annotation indicates that permissions were modified.

## Task 5: Revisiting the Security Group Configuration

After AWS Config evaluates the security group, `LabSG1` is examined again.

The desired final configuration contains HTTP and HTTPS, while the unwanted SMTPS and IMAPS permissions are removed.

| Rule | Port | Result |
|---|---:|---|
| HTTP | 80 | Present |
| HTTPS | 443 | Present |
| SMTPS | 465 | Removed |
| IMAPS | 993 | Removed |

### Lambda Function Analysis

The pre-created Lambda function is named `awsconfig_lambda_security_group`.

The function:

- Imports `boto3`, the AWS SDK for Python.
- Defines `REQUIRED_PERMISSIONS` for the desired inbound configuration.
- Uses EC2 security group information to inspect the current configuration.
- Checks the `debug` parameter supplied by the AWS Config rule.
- Detects unwanted inbound permissions.
- Revokes unwanted security group ingress permissions.

The Lambda function therefore provides the automated remediation mechanism for the monitored security groups.

## Task 6: CloudWatch Logs Verification

CloudWatch Logs are used to verify the actions performed by the Lambda function.

The Lambda log group is:

```text
/aws/lambda/awsconfig_lambda_security_group
```

The log events are searched using the following filter:

```text
revoking for
```

The resulting log events provide evidence that the Lambda function revoked unwanted inbound permissions from the monitored security groups.

For `LabSG1`, the remediation removes the SMTPS and IMAPS permissions corresponding to TCP ports 465 and 993.

## Remediation Workflow

The complete remediation workflow is:

1. AWS Config continuously records EC2 security group configurations.
2. A monitored security group is evaluated by the `EC2SecurityGroup` custom rule.
3. The AWS Config rule invokes the Lambda function.
4. The Lambda function compares the current configuration with the required inbound permissions.
5. Unwanted permissions are revoked from the security group.
6. AWS Config evaluates the resulting configuration.
7. The security group becomes compliant.
8. CloudWatch Logs provide evidence of the remediation activity.

## Result

The experiment demonstrates automated security remediation using AWS Config and AWS Lambda.

The `LabSG1` security group was intentionally modified with additional inbound permissions. AWS Config detected the configuration and the custom `EC2SecurityGroup` rule invoked the Lambda function. The Lambda function removed the unwanted SMTPS and IMAPS permissions and restored the desired security group configuration.

CloudWatch Logs provided audit evidence showing the revocation activity performed by the Lambda function.

## Conclusion

This experiment demonstrates how AWS Config, AWS Lambda, IAM, EC2 security groups, and CloudWatch Logs can be combined to implement automated security monitoring and remediation.

AWS Config continuously monitors resource configurations, the custom rule detects configuration changes, Lambda performs the required remediation, IAM provides the necessary permissions, and CloudWatch Logs provide evidence for auditing the remediation process.
