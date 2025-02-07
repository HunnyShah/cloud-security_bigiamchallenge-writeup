# Challenge 4

## Challenge Statement

In this challenge, we were provided with an AWS S3 bucket policy that appeared to restrict access to a specific admin user. However, upon closer inspection, there was a misconfiguration that allowed us to bypass the restriction and retrieve the flag. This challenge tests knowledge of AWS S3 bucket policies, IAM conditions, and security misconfigurations.

### S3 Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321/*"
    },
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321",
      "Condition": {
        "StringLike": {
          "s3:prefix": "files/*"
        },
        "ForAllValues:StringLike": {
          "aws:PrincipalArn": "arn:aws:iam::133713371337:user/admin"
        }
      }
    }
  ]
}
```

## Analysis

- The **GetObject** permission allows anyone (`"Principal": "*"`) to retrieve objects from the bucket, meaning if we knew the file path, we could still access it.
- The **ListBucket** permission is restricted using `aws:PrincipalArn`, meaning only the specified admin user can list the files in the bucket.
- This means while we **could not list files**, we **could still retrieve files directly** if we guessed or knew the object names.

## Solution

### **Step 1: List the files in the S3 bucket**

We attempted to list the available files:

```bash
aws s3 ls s3://thebigiamchallenge-admin-storage-abf1321/files/
```

However, we received an **Access Denied** error due to the `ListBucket` restriction.

We then attempted listing without authentication:

```bash
aws s3 ls s3://thebigiamchallenge-admin-storage-abf1321/files/ --no-sign-request
```

This again resulted in an access error. Since listing was blocked, we had to guess the file name.

### **Step 2: Retrieve the Flag**

We knew that previous challenges stored flags under predictable names, so we tried:

```bash
aws s3 cp s3://thebigiamchallenge-admin-storage-abf1321/files/flag-as-admin.txt - --no-sign-request
```

This successfully printed the flag to the terminal.

## Reflection

### **Approach Taken**

- Examined the bucket policy and identified the misconfiguration.
- Realized `GetObject` was still open to the public.
- Bypassed the restricted `ListBucket` action by guessing the file name.
- Successfully retrieved the flag.

### **Challenges Faced**

- Initial attempts to list files were blocked.
- Required understanding of policy misconfigurations to exploit the flaw.

### **Security Takeaways**

- **Granting `s3:GetObject` permissions to `"Principal": "*"` allows unauthorized access.**
- **Even if `ListBucket` is restricted, files remain accessible if paths are predictable.**
- **Use IAM conditions properly to enforce restricted access on all actions.**
- **Enable logging and monitoring to detect unauthorized access attempts.**

This challenge demonstrated how subtle misconfigurations in AWS S3 policies can lead to unintended public access. 🚀
