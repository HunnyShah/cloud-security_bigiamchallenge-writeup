# Challenge 1

## Challenge Statement

In this challenge, we were presented with an S3 bucket policy and tasked with finding a flag. The challenge tests understanding of AWS IAM policies, S3 bucket permissions, and common security misconfigurations.

### IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::thebigiamchallenge-storage-9979f4b/*"
    },
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::thebigiamchallenge-storage-9979f4b",
      "Condition": {
        "StringLike": {
          "s3:prefix": "files/*"
        }
      }
    }
  ]
}
```

### Analysis

**GetObject Permission**:

- Allows ALL principals (`"Principal": "*"`)
- Permits `s3:GetObject` action
- Applies to ALL objects in the bucket (`/*`)
- No conditions or restrictions

**ListBucket Permission**:

- Allows ALL principals
- Permits `s3:ListBucket` action
- Restricted to prefix "files/\*"
- This means you can only list objects in the "files/" directory

The key vulnerability lies in the mismatch between listing and access permissions:

- Users can only LIST objects under the "files/" prefix
- BUT can GET any object in the bucket if they know its path
- This is a common security misconfiguration where object access isn't properly scoped

## Solution

1. Tried to list the contents under the "files/" directory since that's what we had permission to do:

aws s3 ls s3://thebigiamchallenge-storage-9979f4b/files/

2. Found there was a file named `flag1.txt` in the files directory.

3. Initial attempts to download the file to disk failed due to a read-only filesystem:

aws s3 cp s3://thebigiamchallenge-storage-9979f4b/files/flag1.txt ./flag1.txt

Error received: `download failed: s3://thebigiamchallenge-storage-9979f4b/files/flag1.txt to ./flag1.txt [Errno 30] Read-only file system: '/var/task/flag1.txt.D486faBD'`

4. Solution: Stream the file content directly to stdout using the dash (-) parameter:

aws s3 cp s3://thebigiamchallenge-storage-9979f4b/files/flag1.txt -

5. This revealed the flag: `{wiz:exposed-storage-risky-as-usual}`

## Reflection

**Approach:**

- Familarize with AWS CLI
- Analyzed S3 bucket policy
- Found we could list "files/" directory and get any object
- Used AWS CLI to explore
- Found flag1.txt in files directory
- Used aws s3 cp s3://thebigiamchallenge-storage-9979f4b/files/flag1.txt - to read it

**Main Challenge:**

- Didn't knew about AWS CLI Learned through docs online
- Read-only filesystem preventing file downloads
- Solved by streaming to stdout instead of saving file

**Defense Tips:**

- Match GetObject and List permissions properly
- Avoid using "\*" for Principal
- Follow least privilege principle
- Enable S3 Block Public Access settings
