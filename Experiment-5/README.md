# Experiment 5 — Encrypting Data at Rest by Using AWS KMS

## Experiment Overview
 
This experiment demonstrates how AWS Key Management Service (AWS KMS) can be used to encrypt and protect data at rest across AWS services.
 
The experiment covers:
 
1. Creating a customer managed symmetric KMS key.
2. Encrypting an Amazon S3 object using SSE-KMS.
3. Testing public access behavior for an SSE-KMS encrypted object.
4. Accessing the encrypted object through authenticated signed access.
5. Inspecting KMS activity using AWS CloudTrail.
6. Creating an encrypted Amazon EBS volume from an unencrypted root-volume snapshot.
7. Replacing the EC2 root volume with the encrypted volume.
8. Disabling and re-enabling the KMS key to observe its effect on dependent resources.
 
---
 
## AWS Services Used
 
- AWS Key Management Service (KMS)
- Amazon Simple Storage Service (S3)
- Amazon Elastic Compute Cloud (EC2)
- Amazon Elastic Block Store (EBS)
- AWS CloudTrail
 
---
 
## Environment
 
| Parameter | Value |
|---|---|
| AWS Region | `us-east-1` — US East (N. Virginia) |
| KMS Key Type | Symmetric |
| KMS Key Alias | `MyKMSKey` |
| Key Administrator | `voclabs` |
| Key User | `voclabs` |
| S3 Object | `clock.png` |
| S3 Encryption | SSE-KMS |
| KMS Key Used | `MyKMSKey` |
| EBS Root Device | `/dev/xvda` |
| EC2 Instance | `LabInstance` |
 
---
 
## Task 1 — Create the Customer Managed KMS Key
 
A customer managed symmetric KMS key was created through the AWS KMS console.
 
### Configuration
 
- Key type: **Symmetric**
- Key usage: **Encrypt and decrypt**
- Alias: `MyKMSKey`
- Key administrator: `voclabs`
- Key user: `voclabs`
 
### Procedure
 
1. Open the AWS KMS console.
2. Navigate to **Customer managed keys**.
3. Select **Create key**.
4. Select **Symmetric** as the key type.
5. Select **Encrypt and decrypt** as the key usage.
6. Set the alias to `MyKMSKey`.
7. Select `voclabs` as the key administrator.
8. Select `voclabs` as the key user.
9. Review the configuration.
10. Create the KMS key.
 
---
 
## Task 2 — Encrypt the S3 Object Using SSE-KMS
 
The `clock.png` object was uploaded to the lab S3 bucket and protected using the customer managed KMS key.
 
### Procedure
 
1. Open Amazon S3.
2. Open the bucket whose name contains `imagebucket`.
3. Select **Upload**.
4. Add `clock.png`.
5. Open the server-side encryption settings.
6. Select **Specify an encryption key**.
7. Select **SSE-KMS**.
8. Choose the AWS KMS key `MyKMSKey`.
9. Upload the object.
10. Verify that the object uses SSE-KMS encryption.
 
---
 
## Task 3 — Configure Public Access and Test the Encrypted Object
 
The S3 object URL was tested before and after changing the bucket's public access configuration.
 
### Procedure
 
1. Copy the Object URL for `clock.png`.
2. Open the URL in a browser.
3. Observe the initial **Access Denied** response.
4. Open the bucket **Permissions** tab.
5. Edit **Block public access**.
6. Clear **Block all public access**.
7. Confirm the change by entering `confirm`.
8. Edit **Object Ownership**.
9. Enable ACLs.
10. Select **Bucket owner preferred**.
11. Save the configuration.
12. Return to `clock.png`.
13. Select **Actions → Make public using ACL**.
14. Select **Make public**.
15. Refresh the original unsigned Object URL.
16. Observe the resulting **Invalid Argument** response for the SSE-KMS encrypted object.
 
---
 
## Task 4 — Access the Encrypted Object Using Signed Access
 
The encrypted S3 object was accessed through the authenticated AWS console.
 
### Procedure
 
1. Open the S3 bucket.
2. Navigate to **Objects**.
3. Select `clock.png`.
4. Select **Open**.
5. Verify that the image opens successfully.
6. Inspect the URL and observe signed request parameters beginning with `X-Amz-`.
 
This demonstrated authenticated access to the SSE-KMS protected object.
 
---
 
## Task 5 — Inspect KMS Activity Using AWS CloudTrail
 
AWS CloudTrail Event history was used to inspect KMS activity generated while working with the encrypted S3 object.
 
### Procedure
 
1. Open AWS CloudTrail.
2. Navigate to **Event history**.
3. Change the event filter to **Event source**.
4. Search for `kms`.
5. Select `kms.amazonaws.com`.
6. Locate the **GenerateDataKey** event.
7. Open the event record and inspect its details.
8. Return to Event history.
9. Locate the **Decrypt** event.
10. Open the event record and inspect its details.
 
### Events Observed
 
- `GenerateDataKey`
- `Decrypt`
 
---
 
## Task 6 — Encrypt the EC2 Root Volume
 
The EC2 instance root volume was originally unencrypted. A snapshot was created and used to create a new encrypted EBS volume protected by `MyKMSKey`.
 
### Procedure
 
1. Open Amazon EC2.
2. Locate `LabInstance`.
3. Stop the instance.
4. Open the instance **Storage** section.
5. Locate the root EBS volume.
6. Record the root volume's Availability Zone.
7. Open the volume.
8. Select **Actions → Create snapshot**.
9. Add the tag:
   - Key: `Name`
   - Value: `Unencrypted Root Volume`
10. Create the snapshot.
11. Open **Snapshots**.
12. Wait until the snapshot status becomes **Completed**.
13. Select the snapshot.
14. Select **Actions → Create volume from snapshot**.
15. Set the Availability Zone to the same Availability Zone as the original root volume.
16. Enable **Encrypt this volume**.
17. Select `MyKMSKey`.
18. Create the new volume.
 
### Volume Identification
 
The volumes were renamed for clarity:
 
- Original volume: `Old unencrypted root volume`
- New encrypted volume: `New encrypted root volume`
 
### Replace the Root Volume
 
1. Detach the old unencrypted root volume.
2. Wait until the old volume becomes available.
3. Select the new encrypted volume.
4. Select **Actions → Attach volume**.
5. Select `LabInstance`.
6. Use the device name:
 
```text
/dev/xvda
```
 
7. Attach the volume.
8. Return to `LabInstance`.
9. Open the **Storage** section.
10. Verify that the new root volume is encrypted and associated with `MyKMSKey`.
 
> The instance was kept stopped until the KMS disable/re-enable test.
 
---
 
## Task 7 — Disable and Re-enable the KMS Key
 
The effect of disabling the KMS key on dependent encrypted resources was tested.
 
### Disable the Key
 
1. Open AWS KMS.
2. Navigate to **Customer managed keys**.
3. Select `MyKMSKey`.
4. Select **Key actions → Disable**.
5. Confirm the operation.
 
### Test EC2
 
1. Open Amazon EC2.
2. Select `LabInstance`.
3. Select **Instance state → Start instance**.
4. Observe that the instance does not successfully start while the KMS key is disabled.
5. The instance returns to the stopped state.
 
### Test S3
 
1. Return to Amazon S3.
2. Open `clock.png`.
3. Observe the KMS-related failure.
4. The operation reports `KMS.DisabledException` because the KMS key is disabled.
 
### Inspect CloudTrail
 
The following events were inspected in CloudTrail Event history:
 
- `DisableKey`
- `StartInstances`
- `CreateGrant`
 
The `CreateGrant` event showed an error associated with the disabled KMS key.
 
### Re-enable the Key
 
1. Return to AWS KMS.
2. Select `MyKMSKey`.
3. Select **Key actions → Enable**.
4. Re-enable the key.
5. Return to Amazon EC2.
6. Start `LabInstance`.
7. Wait until the instance reaches the **Running** state.
 
---
 
## Results
 
The experiment successfully demonstrated encryption at rest using AWS KMS.
 
### Verified Results
 
- A customer managed symmetric KMS key named `MyKMSKey` was created.
- `clock.png` was encrypted using SSE-KMS.
- Public access behavior of an SSE-KMS encrypted S3 object was tested.
- The encrypted object was successfully opened through authenticated signed access.
- KMS `GenerateDataKey` and `Decrypt` activity was inspected using CloudTrail.
- An encrypted EBS volume was created from an unencrypted root-volume snapshot.
- The encrypted volume was attached as the EC2 root device using `/dev/xvda`.
- Disabling `MyKMSKey` prevented dependent encrypted operations from working normally.
- Re-enabling `MyKMSKey` restored the ability to start `LabInstance`.
 
---
 
## Key Concepts Demonstrated
 
### AWS KMS
 
AWS Key Management Service provides managed cryptographic keys that can be used to protect data across AWS services.
 
### SSE-KMS
 
Server-side encryption with AWS KMS keys allows Amazon S3 objects to be encrypted using a KMS key.
 
### Data at Rest
 
Data stored in services such as Amazon S3 and Amazon EBS can be encrypted to protect it while it is not actively being processed.
 
### Signed Access
 
Authenticated AWS console access can generate signed requests containing parameters such as `X-Amz-...`, allowing authorized access to protected resources.
 
### CloudTrail
 
AWS CloudTrail records API activity and provides an event history that can be used to inspect operations involving AWS KMS and other AWS services.
 
### KMS Key Availability
 
Resources encrypted with a KMS key depend on that key being available for required cryptographic operations. Disabling the key can therefore prevent encrypted resources from functioning normally until the key is enabled again.
 
---
 
## Repository Structure
 
```text
Experiment-5/
│
├── README.md
├── screenshots/
│   ├── 01-kms-key.png
│   ├── 02-s3-sse-kms.png
│   ├── 03-s3-public-access.png
│   ├── 04-signed-access.png
│   ├── 05-cloudtrail-kms-events.png
│   ├── 06-encrypted-ebs-root-volume.png
│   ├── 07-kms-disabled.png
│   └── 08-kms-reenabled-instance-running.png
│
└── report/
    └── Experiment-5-Completion-Report.docx
```
