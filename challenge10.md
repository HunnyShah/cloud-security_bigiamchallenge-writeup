# Cloudfoxable 5 - Backwards

Challenge Statement

In this challenge, we needed to retrieve a hidden flag stored in AWS Secrets Manager. We were given access to an IAM role named Alexander-Arnold, which had permissions that could help us access the flag. Our task was to enumerate IAM roles, analyze permissions, and extract the flag from AWS Secrets Manager.

Analysis

The challenge required us to investigate the permissions assigned to Alexander-Arnold and determine if they allowed access to AWS Secrets Manager.

Step 1: Enumerating IAM Roles

First, we listed the available IAM roles using AWS CLI:

aws iam list-roles --profile cloudfoxable

Output:

{
"Roles": [
{
"RoleName": "Alexander-Arnold",
"Arn": "arn:aws:iam::418295694570:role/Alexander-Arnold"
}
]
}

This confirmed the presence of the Alexander-Arnold role.

Step 2: Checking Attached Policies

We then checked the policies attached to Alexander-Arnold:

aws iam list-attached-role-policies --role-name Alexander-Arnold --profile cloudfoxable

Output:

{
"AttachedPolicies": [
{
"PolicyName": "corporate-domain-admin-password-policy",
"PolicyArn": "arn:aws:iam::418295694570:policy/corporate-domain-admin-password-policy"
}
]
}

Step 3: Reviewing the Policy

We retrieved the policy document to analyze its permissions:

aws iam get-policy-version --policy-arn arn:aws:iam::418295694570:policy/corporate-domain-admin-password-policy --version-id v1 --profile cloudfoxable

Output:

{
"PolicyVersion": {
"Document": {
"Statement": [
{
"Action": [
"secretsmanager:GetSecretValue"
],
"Effect": "Allow",
"Resource": [
"arn:aws:secretsmanager:us-east-1:418295694570:secret:DomainAdministrator-Credentials-mQa7vU"
]
}
]
}
}
}

This indicated that the Alexander-Arnold role had access to retrieve a specific secret from AWS Secrets Manager.

Step 4: Listing Secrets

We listed available secrets to identify the relevant one:

aws secretsmanager list-secrets --profile cloudfoxable

Output:

{
"SecretList": [
{
"Name": "DomainAdministrator-Credentials",
"ARN": "arn:aws:secretsmanager:us-east-1:418295694570:secret:DomainAdministrator-Credentials-mQa7vU"
}
]
}

Step 5: Retrieving the Secret Value

Since our policy allowed access to DomainAdministrator-Credentials, we extracted the secret using:

aws secretsmanager get-secret-value --secret-id DomainAdministrator-Credentials --profile Alexander-Arnold

Output:

{
"SecretString": "FLAG{backwards::IfYouFindSomethingInterstingFindWhoHasAccessToIt}"
}

The flag was successfully retrieved!

Solution Summary

Enumerated IAM roles using aws iam list-roles.

Checked attached policies for Alexander-Arnold.

Retrieved and analyzed the policy document to identify secretsmanager:GetSecretValue permissions.

Listed secrets in AWS Secrets Manager.

Retrieved the secret value, successfully obtaining the flag.

Flag:

FLAG{backwards::IfYouFindSomethingInterstingFindWhoHasAccessToIt}

Reflection

Approach Taken

Enumerated IAM roles and policies.

Identified secretsmanager:GetSecretValue permissions.

Retrieved secret values using AWS CLI.

Challenges Faced

Understanding AWS IAM role permissions.

Ensuring the correct AWS profile was used for requests.

Security Takeaways

IAM permissions should follow the principle of least privilege.

The policy allowed access to sensitive credentials, which could be exploited if misconfigured.

Secrets should have strict access controls.

Any role with secretsmanager:GetSecretValue permissions should be carefully managed.

AWS CloudTrail should be enabled to monitor access to secrets.

Logging and alerts should be set up for sensitive secret access attempts.

This challenge reinforced the importance of IAM privilege management and the need for secure access controls in AWS Secrets Manager. 🚀
