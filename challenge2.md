# Challenge 1

## Challenge Statement

The challenge presented an SQS (Simple Queue Service) policy with public read/write permissions, challenging us to find a hidden flag through queue message analysis.

### IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["sqs:SendMessage", "sqs:ReceiveMessage"],
      "Resource": "arn:aws:sqs:us-east-1:092297851374:wiz-tbic-analytics-sqs-queue-ca7a1b2"
    }
  ]
}
```

### Analysis

**Policy Permission**:

- Public access ("Principal": "\*")
- Permissions to send and receive messages
- Queue used for "analytics" purposes

## Solution

1. Used AWS CLI to receive messages from the queue

2. Retrieved message containing:

```json
{
  "URL": "https://tbic-wiz-analytics-bucket-b44867f.s3.amazonaws.com/pAXCWLa6ql.html",
  "User-Agent": "Lynx/2.5329.3258dev.35046 libwww-FM/2.14 SSL-MM/1.4.3714",
  "IsAdmin": true
}
```

3. Identified the flag in the URL path: `/pAXCWLa6ql.html`

## Reflection

**Approach:**

- Analyzed SQS policy
- Read messages from public queue
- Found analytics data with embedded flag
- Extracted flag from URL

**Main Challenge:**

- Identifying the flag's location within analytics data
- Navigating AWS CLI to read queue messages

**Defense Tips:**

- Restrict SQS queue access
- Don't embed sensitive data in URLs
- Implement message encryption
- Use least-privilege IAM policies
- Monitor and audit message contents
