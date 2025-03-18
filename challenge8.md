# Cloudfoxable 3 - It's a Secret

## Challenge Statement

In this challenge, we were tasked with discovering and retrieving a hidden flag stored in AWS Secrets Manager or AWS Systems Manager (SSM) Parameter Store. The goal was to enumerate available secrets and extract their values using AWS CLI tools. This challenge tested our ability to use IAM permissions, understand AWS secret management, and leverage enumeration techniques to find sensitive information.

## Analysis

The environment was configured with a user profile `cloudfoxable-admin` with full `AdministratorAccess` permissions. This allowed us to list and retrieve secrets without any restrictions.

### **Step 1: Enumerating Secrets with CloudFox**

We started by using **CloudFox**, a security tool designed to enumerate AWS resources. Running the following command helped us identify all stored secrets:

```bash
cloudfox aws -p cloudfoxable secrets -v2
```

#### **Findings:**

CloudFox retrieved a list of secrets stored in **AWS Secrets Manager** and **AWS Systems Manager (SSM) Parameter Store**:

```
╭────────────────┬───────────┬───────────────────────────────────────┬──────────────────────────────────╮
│    Service     │  Region   │                 Name                  │           Description            │
├────────────────┼───────────┼───────────────────────────────────────┼──────────────────────────────────┤
│ SecretsManager │ us-east-1 │ DomainAdministrator-Credentials       │                                  │
│ SecretsManager │ us-east-1 │ SegueFlag                             │                                  │
│ SecretsManager │ us-east-1 │ database-secret                       │                                  │
│ SecretsManager │ us-east-1 │ my-app-secret                         │ Secure secret for sensitive data │
│ SSM            │ us-east-1 │ /cloudfoxable/flag/its-a-secret       │                                  │
╰────────────────┴───────────┴───────────────────────────────────────┴──────────────────────────────────╯
```

We noted that the **SSM parameter** `/cloudfoxable/flag/its-a-secret` was a likely candidate for the flag.

### **Step 2: Extracting Secrets from AWS Systems Manager (SSM)**

Since we identified a potential flag stored as an **SSM Parameter**, we attempted to retrieve it using the AWS CLI:

```bash
aws --profile cloudfoxable ssm get-parameter --name "/cloudfoxable/flag/its-a-secret" --with-decryption
```

#### **Output:**

```json
{
  "Parameter": {
    "Name": "/cloudfoxable/flag/its-a-secret",
    "Type": "SecureString",
    "Value": "FLAG{ItsASecret::IsASecretASecretIfTooManyPeopleHaveAccessToIt?}",
    "Version": 1,
    "LastModifiedDate": "2025-02-28T17:12:27.136000+00:00",
    "ARN": "arn:aws:ssm:us-east-1:418295694570:parameter/cloudfoxable/flag/its-a-secret",
    "DataType": "text"
  }
}
```

The flag was successfully retrieved!

### **Step 3: Verifying IAM Permissions**

To ensure we had the required permissions, we enumerated IAM policies attached to our `cloudfoxable-admin` user:

```bash
cloudfox aws -p cloudfoxable permissions --principal cloudfoxable-admin -v2
```

#### **Findings:**

```
╭──────┬────────────────────┬─────────┬─────────────────────┬────────┬────────┬──────────┬───────────╮
│ Type │        Name        │ Policy  │     Policy Name     │ Effect │ Action │ Resource │ Condition │
├──────┼────────────────────┼─────────┼─────────────────────┼────────┼────────┼──────────┼───────────┤
│ User │ cloudfoxable-admin │ Managed │ AdministratorAccess │ Allow  │ *      │ *        │ No        │
╰──────┴────────────────────┴─────────┴─────────────────────┴────────┴────────┴──────────┴───────────╯
```

The `AdministratorAccess` policy granted full access, confirming why we could retrieve all secrets without restrictions.

## Solution

1. Used **CloudFox** to enumerate AWS Secrets Manager and SSM Parameter Store.
2. Identified `/cloudfoxable/flag/its-a-secret` as a potential flag.
3. Used `aws ssm get-parameter` to retrieve the **decrypted value** of the secret.
4. Confirmed we had full `AdministratorAccess` permissions, enabling unrestricted access.
5. Successfully retrieved the flag:

```
FLAG{ItsASecret::IsASecretASecretIfTooManyPeopleHaveAccessToIt?}
```

## Reflection

### **Approach Taken**

- Enumerated secrets using CloudFox.
- Retrieved secure parameters using AWS CLI.
- Verified IAM policies to understand access levels.

### **Challenges Faced**

- Understanding AWS secret management (Secrets Manager vs. SSM Parameter Store).
- Ensuring the correct AWS profile was used.

### **Security Takeaways**

- **IAM permissions should follow the principle of least privilege.**
  - `AdministratorAccess` allowed unrestricted access, making secret retrieval trivial.
- **Secrets should have strict access controls.**

  - SSM parameters containing sensitive information should be restricted to necessary users only.

- **Use AWS CloudTrail for monitoring.**
  - Logging and alerts should be set up for sensitive secret access attempts.

This challenge highlighted the importance of **IAM security**, **secret management**, and **AWS enumeration techniques** for security professionals. 🚀
