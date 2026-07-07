# AWS Security Guide 0: AWS CLI Fundamentals
## Getting Started with the Command Line Interface

---

## Table of Contents
1. [What is AWS CLI?](#what-is-aws-cli)
2. [Installation & Setup](#installation--setup)
3. [Configuration & Credentials](#configuration--credentials)
4. [Basic Commands](#basic-commands)
5. [Common Patterns](#common-patterns)
6. [Troubleshooting](#troubleshooting)
7. [Next Steps](#next-steps)

---

## What is AWS CLI?

### Overview
The **AWS Command Line Interface (AWS CLI)** is a unified tool for managing AWS services from your terminal or shell. Instead of using the AWS Management Console (web interface), you can control most AWS services programmatically through commands.

**Key Benefits:**
- Automate repetitive tasks with scripts
- Infrastructure-as-Code (IaC) friendly
- Faster than clicking through the console
- Version control your infrastructure changes
- CI/CD pipeline integration
- Batch operations on multiple resources

### AWS CLI Versions
- **AWS CLI v2** (Current, Recommended) - Python-based, faster, better features
- **AWS CLI v1** (Legacy) - Still supported but not recommended for new projects

### Official Documentation
**AWS CLI Reference**: https://aws.amazon.com/cli/

---

## Installation & Setup

### Recipe 1: Install AWS CLI v2 on Linux/Mac

**Option A: Using curl (Recommended)**

```bash
# Download the installer
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Unzip the archive
unzip awscliv2.zip

# Run the installer
sudo ./aws/install

# Verify installation
aws --version
# Output: aws-cli/2.13.0 Python/3.11.0 Linux/5.10.0-1-generic

# Get help
aws help
```

**Option B: Using package manager (Ubuntu/Debian)**

```bash
# Update package list
sudo apt-get update

# Install AWS CLI v2
sudo apt-get install -y awscli

# Verify
aws --version
```

**Option C: Using Homebrew (macOS)**

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install AWS CLI
brew install awscli

# Verify
aws --version
```

### Recipe 2: Install AWS CLI v2 on Windows

**Option A: MSI Installer (Easiest)**
1. Download: https://awscli.amazonaws.com/AWSCLIV2.msi
2. Run the installer
3. Open PowerShell and verify: `aws --version`

**Option B: Using Windows Package Manager**
```powershell
winget install Amazon.AWSCLI
```

**Option C: Using Chocolatey**
```powershell
choco install awscli
```

### Recipe 3: Update AWS CLI

```bash
# Check current version
aws --version

# Update to latest (Linux/Mac)
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update

# Update on macOS with Homebrew
brew upgrade awscli

# Update on Windows with MSI
# Download latest MSI and run installer again
```

---

## Configuration & Credentials

### Recipe 1: Generate AWS Access Keys

**In AWS Console:**
1. Sign in to AWS Management Console
2. Go to: **IAM** → **Users** → Select your username
3. Click **Security credentials** tab
4. Under "Access keys" → Click **Create access key**
5. Choose **Command Line Interface (CLI)**
6. Accept the recommendation
7. Copy and save your **Access Key ID** and **Secret Access Key**
   - **NEVER share these keys!**
   - Store securely (password manager, not git repo)

### Recipe 2: Configure AWS CLI Profile (Easiest Method)

```bash
# Interactive configuration (creates default profile)
aws configure

# You'll be prompted for:
# AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
# AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
# Default region name [None]: us-east-1
# Default output format [None]: json
```

**Configuration files created:**
- `~/.aws/credentials` — Stores access keys
- `~/.aws/config` — Stores region and output format

**Example credentials file (~/.aws/credentials):**
```ini
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

[dev-account]
aws_access_key_id = AKIAIOSFODNN7DEV
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYDEVEXAMPLE

[prod-account]
aws_access_key_id = AKIAIOSFODNN7PROD
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYPRODEXAMPLE
```

**Example config file (~/.aws/config):**
```ini
[default]
region = us-east-1
output = json

[profile dev-account]
region = us-west-2
output = table

[profile prod-account]
region = eu-west-1
output = json
```

### Recipe 3: Use Multiple AWS Accounts (Profiles)

```bash
# Configure a new profile
aws configure --profile dev-account

# Use specific profile in commands
aws s3 ls --profile dev-account

# Set default profile for session
export AWS_PROFILE=dev-account

# List all configured profiles
aws configure list-profiles

# View profile details
aws configure list --profile prod-account
```

### Recipe 4: Use Environment Variables (Alternative)

```bash
# Set credentials via environment variables
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_DEFAULT_REGION="us-east-1"
export AWS_DEFAULT_OUTPUT="json"

# Test connection
aws sts get-caller-identity
```

**Output:**
```json
{
    "UserId": "AIDACKCEVSQ6C2EXAMPLE",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/username"
}
```

### Recipe 5: Use IAM Roles (EC2 Instances)

```bash
# No manual configuration needed!
# AWS CLI automatically uses EC2 instance IAM role

# On EC2 instance, just run:
aws s3 ls

# Verify which role/credentials are being used
aws sts get-caller-identity
```

### Recipe 6: Use Temporary Credentials (STS)

```bash
# Get temporary credentials for cross-account access
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/CrossAccountRole \
  --role-session-name temporary-session

# Output:
# {
#     "Credentials": {
#         "AccessKeyId": "ASIATEMP...",
#         "SecretAccessKey": "...",
#         "SessionToken": "...",
#         "Expiration": "2024-01-15T10:30:00Z"
#     }
# }

# Export credentials for use
export AWS_ACCESS_KEY_ID="ASIATEMP..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_SESSION_TOKEN="..."
```

---

## Basic Commands

### Command Structure
```
aws [service] [command] [subcommand] [options]
```

### Recipe 1: Essential Service Commands

```bash
# EC2 - Virtual Servers
aws ec2 describe-instances                    # List running instances
aws ec2 run-instances --image-id ami-12345    # Launch instance
aws ec2 stop-instances --instance-ids i-1234  # Stop instance
aws ec2 terminate-instances --instance-ids i-1234  # Delete instance

# S3 - Object Storage
aws s3 ls                                      # List buckets
aws s3 ls s3://bucket-name                     # List bucket contents
aws s3 cp file.txt s3://bucket-name/           # Upload file
aws s3 sync . s3://bucket-name/                # Sync directory

# IAM - Identity & Access Management
aws iam list-users                             # List IAM users
aws iam create-user --user-name john           # Create user
aws iam attach-user-policy --user-name john \  # Attach permissions
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# RDS - Managed Databases
aws rds describe-db-instances                  # List databases
aws rds create-db-instance --db-instance-identifier mydb \
  --db-instance-class db.t3.micro --engine mysql

# CloudFormation - Infrastructure as Code
aws cloudformation create-stack --stack-name myapp \
  --template-body file://template.json
```

### Recipe 2: Query Output Formats

```bash
# JSON (default, most flexible)
aws ec2 describe-instances --output json

# Table (human-readable)
aws ec2 describe-instances --output table

# Text (for parsing with grep/awk)
aws ec2 describe-instances --output text

# YAML (rare, but available)
aws ec2 describe-instances --output yaml
```

### Recipe 3: Filter and Query Results (JQ)

```bash
# Install jq (JSON query tool)
sudo apt-get install jq    # Ubuntu/Debian
brew install jq            # macOS

# Get only instance IDs
aws ec2 describe-instances --output json | jq '.Reservations[].Instances[].InstanceId'

# Get running instances with tags
aws ec2 describe-instances \
  --query 'Reservations[].Instances[?State.Name==`running`].[InstanceId,Tags[?Key==`Name`].Value|[0]]' \
  --output table

# Count running instances
aws ec2 describe-instances \
  --query 'length(Reservations[].Instances[])' \
  --output text

# Filter with complex conditions
aws s3api list-buckets --output json | \
  jq '.Buckets[] | select(.CreationDate > "2024-01-01") | .Name'
```

### Recipe 4: Help & Documentation

```bash
# General help
aws help

# Service-specific help
aws s3 help
aws ec2 help

# Command-specific help
aws s3 cp help
aws ec2 run-instances help

# List all available services
aws <tab><tab>    # (in bash with autocomplete)

# Search documentation online
aws iam list-users help    # Opens browser with docs
```

---

## Common Patterns

### Pattern 1: Script to List All Resources

```bash
#!/bin/bash
# List all AWS resources across regions

echo "AWS Account Summary"
echo "==================="

# Account info
echo ""
echo "Account ID:"
aws sts get-caller-identity --query 'Account' --output text

# EC2 instances
echo ""
echo "EC2 Instances:"
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].[InstanceId,State.Name,InstanceType]' \
  --output table

# S3 buckets
echo ""
echo "S3 Buckets:"
aws s3api list-buckets --query 'Buckets[].Name' --output text

# IAM users
echo ""
echo "IAM Users:"
aws iam list-users --query 'Users[].UserName' --output text
```

**Run it:**
```bash
chmod +x list-resources.sh
./list-resources.sh
```

### Pattern 2: Batch Operations

```bash
#!/bin/bash
# Stop all running instances with tag Environment=dev

INSTANCES=$(aws ec2 describe-instances \
  --filters "Name=tag:Environment,Values=dev" \
           "Name=instance-state-name,Values=running" \
  --query 'Reservations[].Instances[].InstanceId' \
  --output text)

for INSTANCE in $INSTANCES; do
  echo "Stopping instance: $INSTANCE"
  aws ec2 stop-instances --instance-ids $INSTANCE
done
```

### Pattern 3: Error Handling

```bash
#!/bin/bash

# Capture error output
set -e    # Exit on error

BUCKET="my-bucket-$RANDOM"

# Create bucket with error handling
if aws s3api create-bucket --bucket $BUCKET --region us-east-1; then
  echo "✓ Bucket created: $BUCKET"
else
  echo "✗ Failed to create bucket"
  exit 1
fi

# Verify bucket exists
if aws s3api head-bucket --bucket $BUCKET; then
  echo "✓ Bucket verified"
else
  echo "✗ Bucket not accessible"
  exit 1
fi
```

### Pattern 4: Parallel Operations

```bash
#!/bin/bash
# Delete multiple S3 buckets in parallel

BUCKETS=("bucket-1" "bucket-2" "bucket-3" "bucket-4" "bucket-5")

# Run deletions in background (4 parallel jobs)
for BUCKET in "${BUCKETS[@]}"; do
  (
    echo "Deleting: $BUCKET"
    aws s3 rm s3://$BUCKET --recursive
    aws s3api delete-bucket --bucket $BUCKET
  ) &
  
  # Limit to 4 parallel jobs
  if [ $(jobs -r -p | wc -l) -ge 4 ]; then
    wait -n
  fi
done

# Wait for all background jobs
wait
echo "All buckets deleted"
```

### Pattern 5: Dry-Run Mode (Read-Only Testing)

```bash
# Test command without making changes
aws ec2 terminate-instances \
  --instance-ids i-1234567890abcdef0 \
  --dry-run

# Output: An error will show if you have permission
# DryRunOperation. Request would have succeeded, but DryRun flag is set.

# Once verified, run without --dry-run
aws ec2 terminate-instances \
  --instance-ids i-1234567890abcdef0
```

---

## Troubleshooting

### Issue 1: "Unable to locate credentials"

**Cause:** AWS CLI can't find credentials.

**Solution:**
```bash
# Check if credentials are configured
aws sts get-caller-identity

# If fails, reconfigure
aws configure

# Or set environment variables
export AWS_ACCESS_KEY_ID="your-key"
export AWS_SECRET_ACCESS_KEY="your-secret"
export AWS_DEFAULT_REGION="us-east-1"

# Test again
aws sts get-caller-identity
```

### Issue 2: "Access Denied" / "UnauthorizedOperation"

**Cause:** IAM permissions are insufficient.

**Solution:**
```bash
# Check current user/role
aws sts get-caller-identity

# Verify permissions
aws iam get-user-policy --user-name username --policy-name PolicyName

# View attached policies
aws iam list-attached-user-policies --user-name username

# Attach required policy (if you have admin access)
aws iam attach-user-policy \
  --user-name username \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

### Issue 3: "The security token included in the request is invalid"

**Cause:** Temporary credentials expired or are incorrect.

**Solution:**
```bash
# Check if using temporary credentials
aws sts get-caller-identity

# If using STS assume-role, refresh
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/MyRole \
  --role-session-name my-session

# Update environment variables with new credentials
export AWS_ACCESS_KEY_ID="new-key"
export AWS_SECRET_ACCESS_KEY="new-secret"
export AWS_SESSION_TOKEN="new-token"
```

### Issue 4: "InvalidAction" or "Unknown service"

**Cause:** Typo in service or command name.

**Solution:**
```bash
# List available services
aws help | grep -i "available services"

# Check command syntax
aws s3 help      # Not "aws s3api" for basic commands

# Use autocomplete
aws s3<tab><tab>
```

### Issue 5: Command Times Out

**Cause:** Network issue or command is hanging.

**Solution:**
```bash
# Add explicit timeout
timeout 30 aws s3 ls

# Use debug mode to see what's happening
aws s3 ls --debug

# Check AWS service health
# Visit: https://health.aws.amazon.com
```

### Issue 6: "JQ: command not found"

**Cause:** jq is not installed.

**Solution:**
```bash
# Install jq
sudo apt-get install jq      # Ubuntu/Debian
brew install jq              # macOS
choco install jq             # Windows/Chocolatey

# Or use Python for JSON parsing
aws s3api list-buckets --output json | python3 -m json.tool
```

---

## Security Best Practices

### DO:
```bash
# 1. Use profiles for different accounts
aws s3 ls --profile prod-account

# 2. Use IAM roles on EC2 (no stored credentials)
# 3. Store credentials in ~/.aws/credentials (not in repo)
# 4. Use environment variables for temporary credentials
# 5. Rotate access keys regularly (every 90 days)
# 6. Use MFA with CLI

# 6. MFA Example
aws sts get-session-token \
  --serial-number arn:aws:iam::123456789012:mfa/username \
  --token-code 123456

# 7. Use CloudTrail to audit AWS CLI commands
aws cloudtrail describe-trails
```

### DON'T:
```bash
# DON'T store credentials in scripts
aws s3 ls --access-key AKIA... --secret-key ...

# DON'T commit credentials to git
# git add ~/.aws/credentials

# DON'T share access keys
# export AWS_ACCESS_KEY_ID="key-in-email"

# DON'T use root account credentials
# aws configure (as root)

# DON'T store credentials in environment variables permanently
# echo "export AWS_ACCESS_KEY_ID=..." >> ~/.bashrc
```

---

## Next Steps

### Progression Path

```
You Are Here (Guide 0: AWS CLI Fundamentals)
    ↓
Guide 1: Logging & Visibility (CloudTrail, AWS Config)
    ↓
Guide 2: Threat Detection (GuardDuty, Security Hub)
    ↓
Guide 3: DDoS & Web Protection (WAF, Shield)
    ↓
Guide 4: S3 Security (Auditing, Encryption, Access Control)
```

### Quick Start Exercises

**Exercise 1: List Your Resources**
```bash
aws sts get-caller-identity
aws ec2 describe-instances --output table
aws s3 ls
```

**Exercise 2: Create and Delete an S3 Bucket**
```bash
# Create bucket
aws s3api create-bucket --bucket test-bucket-$RANDOM --region us-east-1

# List buckets
aws s3 ls

# Delete bucket
aws s3api delete-bucket --bucket test-bucket-XXXXX
```

**Exercise 3: Query Data with JQ**
```bash
# List all running instances with their IDs
aws ec2 describe-instances --output json | \
  jq '.Reservations[].Instances[] | select(.State.Name=="running") | .InstanceId'
```

### Useful Commands Reference

| Task | Command |
|------|---------|
| Get account ID | `aws sts get-caller-identity` |
| Get account alias | `aws iam list-account-aliases` |
| List all S3 buckets | `aws s3 ls` |
| List EC2 instances | `aws ec2 describe-instances --output table` |
| Get service health | `aws health describe-events` |
| View CloudTrail logs | `aws cloudtrail lookup-events` |
| Check IAM permissions | `aws iam list-attached-user-policies --user-name NAME` |
| Get AWS CLI version | `aws --version` |
| Update AWS CLI | `sudo ./aws/install --bin-dir /usr/local/bin --update` |

### Resources

- **AWS CLI Official Docs**: https://aws.amazon.com/cli/
- **AWS CLI GitHub**: https://github.com/aws/aws-cli
- **AWS CLI Configuration**: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html
- **IAM Best Practices**: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
- **AWS API Reference**: https://docs.aws.amazon.com/cli/latest/reference/

---

## Summary

You now have:
- AWS CLI installed
- Credentials configured
- Basic command knowledge
- Common patterns and scripts
- Troubleshooting guide

**Ready for the next guide?** 
→ Proceed to **AWS_Security_Guide_1_Logging_and_Visibility.md** to start monitoring your AWS account with CloudTrail and AWS Config!

