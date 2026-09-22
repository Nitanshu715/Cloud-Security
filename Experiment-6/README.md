# Experiment 6 — Monitoring and Alerting with CloudTrail and CloudWatch

## Cloud Security and Management

**Name:** Nitanshu Tak  
**SAP ID:** 500121943  
**Program:** B.Tech CSE  
**Batch:** 2 CCVT

---

## Experiment Overview

This experiment demonstrates how AWS logging, monitoring, event detection, notification, metric filtering, alarming, and log analysis services can be combined to monitor security-related activity in an AWS environment.

The experiment uses:

- AWS CloudTrail
- Amazon CloudWatch Logs
- Amazon SNS
- Amazon EventBridge
- Amazon CloudWatch
- AWS IAM
- Amazon EC2

---

## Objectives

- Analyze event details in the CloudTrail event history.
- Create and understand a CloudTrail trail with CloudWatch logging enabled.
- Create an SNS topic and an email subscription.
- Configure an EventBridge rule to monitor security-group changes.
- Create a CloudWatch metric filter.
- Create and test a CloudWatch alarm for failed console login attempts.
- Query CloudTrail logs using CloudWatch Logs Insights.

---

## Scenario

The lab environment contained an EC2 instance associated with a security group, a preconfigured CloudTrail trail that writes to CloudWatch Logs, and a preconfigured IAM test user.

The experiment created an SNS topic and email subscription, configured an EventBridge rule to detect security-group changes, and created a CloudWatch metric filter and alarm to detect repeated failed AWS Management Console login attempts.

---

## Task 1 — CloudTrail and CloudWatch Logging

The CloudTrail Event History was inspected to understand AWS audit records.

A CloudFormation `CreateStack` event was examined, including information such as:

- `userIdentity`
- `eventTime`
- `awsRegion`
- API event details

The existing CloudTrail configuration used for the experiment included:

```text
Trail: LabCloudTrail
Log Group: CloudTrailLogGroup
```

CloudTrail provides the audit events that are subsequently consumed by the monitoring components in this experiment.

---

## Task 2 — Amazon SNS Topic and Email Subscription

An Amazon SNS topic was created to provide a notification destination for security-related events.

```text
SNS Topic: MySNSTopic
Subscription: Email
Status: Confirmed
```

The email subscription was confirmed before using the topic as a notification destination.

---

## Task 3 — EventBridge Security Group Monitoring

An Amazon EventBridge rule was created to monitor security-group changes recorded through AWS CloudTrail.

### Rule Configuration

```text
Rule Name: MonitorSecurityGroups
Event Bus: default
Status: Enabled
```

### Event Pattern

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventSource": ["ec2.amazonaws.com"],
    "eventName": [
      "AuthorizeSecurityGroupIngress",
      "ModifyNetworkInterfaceAttribute"
    ]
  }
}
```

### Target

```text
Target Type: SNS topic
Target: MySNSTopic
Input: Input transformer
```

The input transformer was configured to extract information from the CloudTrail event and format it into a notification.

### EventBridge Test

The `LabSecurityGroup` security group was modified by adding an SSH inbound rule.

```text
Protocol: TCP
Port: 22
Source: 0.0.0.0/0
```

The resulting CloudTrail event was verified as:

```text
Event Name: AuthorizeSecurityGroupIngress
Event Source: ec2.amazonaws.com
fromPort: 22
toPort: 22
```

This demonstrated the security-group monitoring workflow:

```text
Security Group Change
        ↓
CloudTrail
        ↓
EventBridge
        ↓
MySNSTopic
        ↓
Email Notification
```

---

## Task 4 — CloudWatch Metric Filter and Alarm

A CloudWatch metric filter was created on `CloudTrailLogGroup` to identify failed AWS Management Console login attempts.

### Metric Filter

```text
Filter Name: ConsoleLoginErrors
Metric Namespace: CloudTrailMetrics
Metric Name: ConsoleLoginFailureCount
Metric Value: 1
```

### Filter Pattern

```text
{ ($.eventName = ConsoleLogin) && ($.errorMessage = "Failed authentication") }
```

### CloudWatch Alarm

```text
Alarm Name: FailedLogins
Metric: ConsoleLoginFailureCount
Period: 5 minutes
Condition: Greater/Equal to 3
Notification Topic: MySNSTopic
```

The alarm was tested using the lab's `test` IAM user with incorrect console credentials. Multiple failed authentication attempts were generated so that the CloudWatch metric and alarm could be evaluated.

The monitoring workflow was:

```text
Failed Console Login
        ↓
CloudTrail
        ↓
CloudTrailLogGroup
        ↓
Metric Filter
        ↓
ConsoleLoginFailureCount
        ↓
FailedLogins Alarm
        ↓
MySNSTopic
        ↓
Email Notification
```

---

## Task 5 — CloudWatch Logs Insights

CloudWatch Logs Insights was used to query CloudTrail logs for failed console login events.

### Query

```text
filter eventSource="signin.amazonaws.com" and eventName="ConsoleLogin" and responseElements.ConsoleLogin="Failure"
| stats count(*) as Total_Count by sourceIPAddress as Source_IP, errorMessage as Reason, awsRegion as AWS_Region, userIdentity.arn as IAM_Arn
```

The query provides a summarized view of failed login activity by:

- Source IP address
- Failure reason
- AWS Region
- IAM ARN
- Total count

---

## Result

The experiment successfully demonstrated an AWS security monitoring and alerting workflow using CloudTrail, CloudWatch Logs, SNS, EventBridge, and CloudWatch.

The completed configuration allowed:

1. AWS API activity to be recorded by CloudTrail.
2. Security-group modifications to be detected by EventBridge.
3. Security events to be delivered through SNS.
4. Failed console login events to be converted into CloudWatch metrics.
5. Repeated failed login attempts to trigger a CloudWatch alarm.
6. CloudTrail logs to be queried and analyzed using CloudWatch Logs Insights.

---

## Conclusion

This experiment demonstrated how multiple AWS security and monitoring services can work together to provide visibility into account activity and generate alerts for selected security events.

The implementation covered audit logging, event-based monitoring, email notification, metric filtering, alarm creation, alarm testing, and log analysis.

---

## Screenshots

Screenshots for the completed experiment are stored in the experiment documentation.

Recommended screenshot sequence:

1. CloudTrail Event History / Trail
2. SNS Topic and Confirmed Subscription
3. EventBridge `MonitorSecurityGroups` Rule
4. CloudTrail `AuthorizeSecurityGroupIngress` Event
5. CloudWatch `ConsoleLoginErrors` Metric Filter
6. CloudWatch `FailedLogins` Alarm
7. Failed Login Test / Alarm State
8. CloudWatch Logs Insights Query and Results

---

## Experiment Status

```text
Experiment 6 — COMPLETED
CloudTrail              ✓
CloudWatch Logs         ✓
SNS                     ✓
Email Subscription      ✓
EventBridge             ✓
Metric Filter           ✓
CloudWatch Alarm        ✓
Logs Insights           ✓
```

---

## Student Details

```text
Name       : Nitanshu Tak
SAP ID     : 500121943
Program    : B.Tech CSE
Batch      : 2 CCVT
Subject    : Cloud Security and Management
Experiment : 6
```


