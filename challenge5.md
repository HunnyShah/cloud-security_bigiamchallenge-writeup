# CloudFoxable - Challenge 1: Do This First!

## Challenge Statement

This challenge involves deploying **CloudFoxable**, an AWS-based CTF environment, using **Terraform**. The goal is to configure AWS credentials, deploy the infrastructure, and retrieve the first flag from the Terraform output.

## Steps to Solve the Challenge

### **Step 1: Create an AWS IAM User**

1. Log in to the **AWS Management Console**.
2. Navigate to **IAM (Identity and Access Management)**.
3. Click **Users** > **Create User**.
4. Enter the username: `cloudfoxable-admin`.
5. Select **Programmatic access** (needed for Terraform).
6. Attach the **AdministratorAccess** policy.
7. Download and save the **Access Key ID** and **Secret Access Key** securely.

### **Step 2: Configure AWS CLI**

1. Install AWS CLI if not already installed.
2. Configure the AWS CLI with the newly created user:

   ```bash
   aws configure
   ```

   - Enter the **Access Key ID**.
   - Enter the **Secret Access Key**.
   - Set the default region (e.g., `us-west-2`).
   - Leave output format as `json` (default).

3. Verify the AWS credentials:
   ```bash
   aws sts get-caller-identity
   ```
   This should return details about the IAM user.

### **Step 3: Install Terraform**

1. Download Terraform for your OS from [Terraform's official site](https://developer.hashicorp.com/terraform/downloads).
2. Extract and move the binary to a directory in your `PATH` (e.g., `/usr/local/bin/` on Linux/MacOS or `C:\Terraform\` on Windows).
3. Verify installation:
   ```bash
   terraform -v
   ```

### **Step 4: Deploy CloudFoxable**

1. Clone the CloudFoxable repository:

   ```bash
   git clone https://github.com/BishopFox/cloudfoxable.git
   cd cloudfoxable/aws
   ```

2. Copy and configure Terraform variables:

   ```bash
   cp terraform.tfvars.example terraform.tfvars
   ```

   Edit `terraform.tfvars` and set the AWS profile:

   ```bash
   aws_local_profile = "default"
   ```

3. Initialize Terraform:

   ```bash
   terraform init
   ```

4. Apply Terraform deployment:
   ```bash
   terraform apply -auto-approve
   ```
   After completion, Terraform will output important deployment details.

### **Step 5: Retrieve the First Flag**

1. Set up credentials for the **CTF Starting User**:

   ```bash
   echo "" >> ~/.aws/credentials && \
   echo "[cloudfoxable]" >> ~/.aws/credentials && \
   echo "aws_access_key_id = `terraform output -raw CTF_Start_User_Access_Key_Id`" >> ~/.aws/credentials && \
   echo "aws_secret_access_key = `terraform output -raw CTF_Start_User_Secret_Access_Key`" >> ~/.aws/credentials && \
   echo "region = us-west-2" >> ~/.aws/credentials
   ```

2. Verify the credentials:

   ```bash
   aws sts get-caller-identity --profile cloudfoxable
   ```

3. The Terraform output contains the flag:
   ```bash
   FLAG{congrats_you_are_now_a_terraform_expert_happy_hunting}
   ```

## Reflection

### **What was your approach?**

- We started by reading the CloudFoxable setup instructions.
- Installed Terraform and AWS CLI.
- Configured AWS credentials for both deployment and CTF users.
- Deployed CloudFoxable using Terraform and retrieved credentials from the output.
- Verified the setup and found the flag.

### **What was the biggest challenge?**

- Ensuring the correct IAM permissions were in place.
- Debugging AWS profile issues when Terraform failed to authenticate.

### **How did you overcome the challenges?**

- Double-checked `~/.aws/credentials` for misconfigured profiles.
- Used `aws sts get-caller-identity` to verify active IAM roles.
- Re-ran `terraform apply` after fixing AWS credentials.

### **What led to the breakthrough?**

- Carefully reading Terraform outputs and recognizing the starting user’s credentials.
- Understanding how AWS IAM roles work and ensuring correct permissions.

### **On the blue side, how can the learning be used to properly defend important assets?**

- **Limit IAM policies**: Avoid using overly permissive IAM policies for Terraform deployments.
- **Use temporary credentials**: Instead of long-term IAM user credentials, enforce temporary credentials with IAM roles.
- **Audit and monitor AWS activity**: Enable CloudTrail logs to track Terraform deployments and IAM role assumptions.
- **Restrict access keys**: Rotate and disable unused IAM access keys regularly.

This challenge highlighted the importance of **proper IAM access control** and **Terraform security best practices**. 🚀
