# Implementing Cloud Security using AWS IAM, KMS, and Security Controls

---

**Objectives**

  - Implement Identity and Access Management (IAM) with least privilege
  - Create IAM users, groups, roles, and policies
  - Configure S3 bucket encryption using AWS KMS
  - Implement network security using Security Groups and NACLs
  - Enable AWS CloudTrail for security auditing
  - Understand multi tenancy security isolation

---

**Theory**

Cloud security operates under the Shared Responsibility Model, which divides security obligations between AWS and the customer.

  - **AWS responsibility** ("Security of the cloud"): Physical infrastructure, hypervisor, and networking hardware.
  - **Customer responsibility** ("Security in the cloud"): Data, IAM, encryption, OS patching, and firewall rules.

**Security Concepts:**

**IAM (Identity and Access Management)** controls who can authenticate and what they are authorized to do on which resources. It works through users, groups, roles, and policies.

**Encryption** protects data at rest (S3, EBS) and in transit (TLS/SSL). AWS KMS manages the underlying encryption keys.

**Network Security** is enforced at two levels; Security Groups act as stateful, instance level firewalls, while NACLs (Network Access Control Lists) are stateless and operate at the subnet level.

**Security Monitoring** is provided by CloudTrail, which logs every API call made in an account, and by CloudWatch, which monitors metrics and can trigger automated alarms.

**Multi tenancy Security** in AWS relies on VPCs, IAM permission boundaries, and hypervisor level isolation through the Nitro system to keep tenants separated. In SaaS environments, key concerns include data segregation, granular access control, and regulatory compliance across shared infrastructure.

---

**Procedure**

**Part A: IAM Security**

**Step 1: Create an IAM Group**

  1. Open the AWS Console and navigate to IAM.
  2. In the left sidebar select Groups, then create a new group named `Developers`.
  3. Attach the managed policy `AmazonS3ReadOnlyAccess` to the group.
  4. Save the group.

**Step 2: Create IAM Users**

  1. In IAM, go to Users and create a new user named `dev-user1`.
  2. Add `dev-user1` to the `Developers` group.
  3. Create a second user named `admin-user`.
  4. Attach the `AdministratorAccess` policy directly to `admin-user`.

**Step 3: Create a Custom IAM Policy (Least Privilege)**

  1. In IAM, go to Policies and create a new policy.
  2. Switch to the JSON editor and paste the following:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::cloud-lab-storage-*",
        "arn:aws:s3:::cloud-lab-storage-*/*"
      ]
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "*"
    }
  ]
}
```

  3. Name the policy `S3-ReadOnly-NoDeletion` and save it.
  4. Attach this policy to the `Developers` group.

**Step 4: Create an IAM Role for EC2**

  1. In IAM, go to Roles and create a new role.
  2. Set the trusted entity to AWS Service and select EC2.
  3. Attach the `AmazonS3ReadOnlyAccess` policy.
  4. Name the role `EC2-S3-ReadOnly-Role` and create it.
  5. Navigate to your EC2 instance, open Instance Settings, select Modify IAM Role, and attach `EC2-S3-ReadOnly-Role`.

**Step 5: Test Permissions as dev-user1**

  1. Sign in to the AWS Console as `dev-user1`.
  2. Navigate to S3 and attempt to read an object. This should succeed.
  3. Attempt to delete an S3 object. This should fail with an "Access Denied" error.


**Part B: Encryption with KMS**

**Step 6: Create a KMS Key**

  1. Navigate to AWS `Key Management Service` and choose to create a new key.
  2. Set the key type to Symmetric.
  3. Enter `cloud-lab-key` as the alias.
  4. Set `admin-user` or `LabRole` as the key administrator.
  5. Grant key usage permissions to the `Developers` or `LabRole` group.
  6. Finish creating the key.

>Screenshot: Page showing details of the key [Mandatory]

>Sample:

<img width="1455" height="719" alt="Screenshot 2026-04-22 at 7 38 38 AM" src="https://github.com/user-attachments/assets/3cf25921-16b6-497a-8186-8719998ee43a" />


**Step 7: Enable S3 Server Side Encryption**

  1. Open your S3 bucket and go to the Properties tab.
  2. Under Default Encryption, select "AWS Key Management Service key (SSE-KMS)".
  3. Choose `cloud-lab-key` from the list.
  4. Save the settings.

**Step 8: Upload a File and Verify Encryption**

  1. Upload a new file to the S3 bucket.
  2. Open the object's properties and confirm the encryption field shows SSE-KMS.


**Part C: Network Security**

**Step 9: Configure Security Groups**

  1. In the EC2 console, navigate to Security Groups and create a new group named `web-server-sg` with description `Allows SSH and HTTP access to developers.`
  2. Add the following inbound rules:
     - Type: `HTTP` (port 80), Source Type: `Anywhere IPv4`
     - Type: `SSH` (port 22), Source Type: `Anywhere IPv4` or `My IP`
  3. Set outbound to allow all traffic.
  4. Create a second security group named `db-server-sg`, with description `Allow MySQL/Aurora (port 3306) from the web-server-sg security group only`
  5. Add one inbound rule type: `MySQL/Aurora` (port 3306) source type:customs source:`web-server-sg` only.
  6. Set outbound to allow all traffic.

> *Screenshot: Take a screenshot of the rules for both security groups [Mandatory].*

> Sample:

<img width="1470" height="413" alt="Screenshot 2026-04-22 at 7 48 29 AM" src="https://github.com/user-attachments/assets/bde4e98a-3d46-4f66-80d3-2c60f57ecf6c" />

<img width="1470" height="395" alt="Screenshot 2026-04-22 at 7 55 59 AM" src="https://github.com/user-attachments/assets/6a29b79b-8244-4e2f-82f4-3e16f9221fa4" />


**Step 10: Configure Network ACLs**

  1. In the VPC console, go to Network ACLs and create a new NACL for your private subnet.
  2. Name:my-ACLs , VPC: select any available VPC and click on create.
  3. Select and click on the `my-ACLs` and click on the Edit inbound rules
  4. Add inbound and outbound rules to deny all traffic except traffic originating from the public subnet CIDR.

>*Screenshot: Take a screenshot of the NACL rules [Mandatory].*

>Sample:

<img width="1470" height="395" alt="Screenshot 2026-04-22 at 8 03 52 AM" src="https://github.com/user-attachments/assets/ae0d5721-f88f-498f-842f-d3a842fa9e32" />


**Part D: Security Monitoring**

**Step 11: Enable AWS CloudTrail**

  1. Navigate to CloudTrail and create a new trail.
  2. Name the trail `security-audit-trail`.
  3. `Trail log bucket and folder` automatically `Logs will be stored in aws-cloudtrail-logs-143724630910-8a92bda6/AWSLogs/143724630910`
  4. Create the trail.

>*Screenshot: Complete the setup and take a screenshot of the trail configuration [Mandatory].*

>Sample:

<img width="1470" height="245" alt="Screenshot 2026-04-22 at 8 11 58 AM" src="https://github.com/user-attachments/assets/00d9406a-0cbc-41b4-8f26-971674208ced" />


**Step 12: View CloudTrail Events**

  1. In CloudTrail, open Event History.

>*Screenshot: Take a screenshot of the event history [Mandatory].*

>Sample:

<img width="1458" height="685" alt="Screenshot 2026-04-22 at 8 17 09 AM" src="https://github.com/user-attachments/assets/7f91641c-c4a3-429e-a68d-a5fccf364ac7" />

---

**Results**

- IAM users, groups, roles, and a custom policy were successfully created.
- The least privilege principle was enforced: developers can read but cannot delete S3 objects.
- KMS encryption was enabled for S3 data at rest.
- Security Groups implement instance level firewall rules with clear tier isolation between web and database layers.
- CloudTrail captures all API activity, supporting security auditing and compliance.

---

**Discussion and Conclusion**

This lab applied multiple layers of cloud security in line with a defense in depth approach. IAM provides fine grained access control through policies built on the principle of least privilege. KMS handles encryption key management for data at rest protection, which is especially important in multi tenant environments where data isolation is critical. Security Groups and NACLs enforce network level boundaries between application tiers. CloudTrail enables continuous monitoring and forensic analysis of all account activity, which is essential for both compliance and incident response.

In a multi tenant SaaS environment, these layered controls work together to ensure that one tenant's data and resources remain isolated from all others. Legal and regulatory requirements around data residency and privacy are supported through encryption, access controls, and audit trails.

---
