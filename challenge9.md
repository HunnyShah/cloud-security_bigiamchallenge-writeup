# Cloudfoxable 4 - It's Another Secret

## Challenge Statement

In this challenge, we were tasked with retrieving a hidden flag stored in AWS Systems Manager (SSM) Parameter Store. The challenge provided access to an IAM role named `Ertz`, which we needed to assume in order to enumerate permissions and locate the flag. This exercise tested our ability to assume roles, enumerate IAM permissions, and interact with AWS services to extract sensitive data.

## Analysis

The challenge required assuming the `Ertz` role from a starting user, `ctf-starting-user`. After assuming the role, we needed to analyze its permissions and find the flag stored in AWS SSM.

### **Step 1: Assuming the Ertz Role**

The first step was to assume the `Ertz` role. We did this by editing our AWS credentials configuration file (`~/.aws/config`) to include a new profile for `ertz`:

```ini
[profile ertz]
region = us-west-2
role_arn = arn:aws:iam::418295694570:role/Ertz
source_profile = cloudfoxable
```

After adding the profile, we verified that we could successfully assume the role by running:

```bash
aws --profile ertz sts get-caller-identity
```

#### **Output:**

```json
{
  "UserId": "AROAWCZC5YTVN777FU3K4:botocore-session-1742744272",
  "Account": "418295694570",
  "Arn": "arn:aws:sts::418295694570:assumed-role/Ertz/botocore-session-1742744272"
}
```

This confirmed that we successfully assumed the `Ertz` role.

### **Step 2: Enumerating IAM Permissions**

Once we assumed the `Ertz` role, the next step was to enumerate its permissions. We attempted to retrieve user policies:

```bash
aws --profile ertz iam get-user-policy --user-name Ertz
```

However, this resulted in an error because the `ertz` role did not have permission to list user policies. Similarly, trying to list attached policies failed:

```bash
aws --profile ertz iam list-attached-user-policies --user-name Ertz
```

#### **Error Output:**

```plaintext
An error occurred (AccessDenied) when calling the ListAttachedUserPolicies operation: User is not authorized.
```

Since we could not enumerate policies directly, we had to rely on trial and error to test available permissions.

### **Step 3: Searching for the Flag**

Based on prior challenges, we suspected that the flag might be stored in **AWS SSM Parameter Store**. We attempted to retrieve parameters using:

```bash
aws --profile ertz ssm get-parameter --name /cloudfoxable/flag/its-another-secret --with-decryption
```

#### **Successful Output:**

```json
{
  "Parameter": {
    "Name": "/cloudfoxable/flag/its-another-secret",
    "Type": "SecureString",
    "Value": "FLAG{ItsAnotherSecret::ThereWillBeALotOfAssumingRolesInThisCTF}",
    "Version": 1,
    "LastModifiedDate": "2025-02-28T17:12:27.274000+00:00",
    "ARN": "arn:aws:ssm:us-east-1:418295694570:parameter/cloudfoxable/flag/its-another-secret",
    "DataType": "text"
  }
}
```

The flag was successfully retrieved!

## Solution Summary

1. **Assumed the `Ertz` role** using AWS CLI by configuring the profile in `~/.aws/config`.
2. **Verified the role assumption** using `sts get-caller-identity`.
3. **Tested available IAM permissions** by attempting to list policies but encountered access denial.
4. **Retrieved the flag** from AWS SSM Parameter Store using `ssm get-parameter --with-decryption`.
5. **Successfully extracted the flag:**

```
FLAG{ItsAnotherSecret::ThereWillBeALotOfAssumingRolesInThisCTF}
```

## Reflection

### **Approach Taken**

- Configured AWS CLI profiles to assume the target role.
- Verified assumed role identity.
- Used trial and error to determine available permissions.
- Retrieved the secret from AWS SSM using valid permissions.

### **Challenges Faced**

- Limited ability to enumerate permissions due to `AccessDenied` errors.
- Uncertainty about which AWS services contained the flag.
- Ensuring the correct AWS profile was used for role assumption.

### **Security Takeaways**

- **IAM roles should follow the principle of least privilege.**
  - The `Ertz` role had permissions to retrieve SSM parameters but not to enumerate its own policies.
- **SSM SecureStrings should have strict access controls.**
  - Anyone with access to the `ertz` role could retrieve sensitive secrets.
- **Role assumption is a powerful technique in AWS pentesting.**
  - Understanding how to assume roles and test permissions is key to cloud security assessments.

This challenge emphasized the importance of **role-based access control**, **IAM privilege enumeration**, and **secure storage of secrets** within AWS. 🚀
