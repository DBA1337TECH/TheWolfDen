# AWS Security Guide 2: Threat Detection & Response
## GuardDuty & Security Hub

---

## Table of Contents
1. [GuardDuty Fundamentals](#guardduty-fundamentals)
2. [GuardDuty Setup & Configuration](#guardduty-setup--configuration)
3. [Security Hub Fundamentals](#security-hub-fundamentals)
4. [Security Hub Setup & Management](#security-hub-setup--management)
5. [Threat Response Recipes](#threat-response-recipes)
6. [Automated Response Patterns](#automated-response-patterns)

---

## GuardDuty Fundamentals

### What is GuardDuty?
Amazon GuardDuty is a managed threat detection service that uses machine learning to identify malicious activity, unusual patterns, and unauthorized access attempts.

**Data Sources:**
- VPC Flow Logs (network traffic)
- CloudTrail (API calls)
- DNS logs (DNS queries)
- S3 data event logs
- EBS snapshots (malware detection)

**Key Concepts:**
- **Findings**: Potential security issues with severity (Low, Medium, High)
- **Threat Intelligence**: IP reputation feeds and curated databases
- **Machine Learning Models**: Behavioral analysis and anomaly detection
- **Member Accounts**: Organization-wide management

### Finding Types
- **Recon**: Probe attacks (port scanning, domain enumeration)
- **Resource Compromise**: Cryptomining, DDoS, account compromise
- **Credential Access**: Brute force, unauthorized access
- **Defense Evasion**: Disabling CloudTrail, modifying security groups

---

## GuardDuty Setup & Configuration

### Recipe 1: Enable GuardDuty for Organization

**Use Case**: Centralized threat detection across all AWS accounts.

```bash
# Enable GuardDuty in management account
aws guardduty create-detector \
  --enable \
  --finding-publishing-frequency FIFTEEN_MINUTES \
  --region us-east-1

# Get detector ID (save this!)
DETECTOR_ID=$(aws guardduty list-detectors --region us-east-1 --query 'DetectorIds[0]' --output text)
echo "Detector ID: $DETECTOR_ID"

# Enable for organization (from management account)
aws guardduty create-organization-admin-account \
  --admin-account-id 123456789012 \
  --region us-east-1

# List member accounts to add
aws organizations list-accounts \
  --query 'Accounts[*].[Id,Name]' \
  --output table
```

### Recipe 2: Centralized Member Account Management

**Use Case**: Automatically enable GuardDuty across all org accounts.

```bash
#!/bin/bash
# Script: enable-guardduty-org.sh

MANAGEMENT_ACCOUNT=123456789012
ADMIN_ACCOUNT=123456789012
REGION=us-east-1

# Get all accounts
ACCOUNTS=$(aws organizations list-accounts \
  --query 'Accounts[?Status==`ACTIVE`].Id' \
  --output text)

for ACCOUNT in $ACCOUNTS; do
  if [ "$ACCOUNT" != "$ADMIN_ACCOUNT" ]; then
    echo "Creating GuardDuty detector in account: $ACCOUNT"
    
    # Use STS to assume role in member account
    CREDENTIALS=$(aws sts assume-role \
      --role-arn arn:aws:iam::$ACCOUNT:role/OrganizationAccountAccessRole \
      --role-session-name guardduty-setup \
      --query 'Credentials.[AccessKeyId,SecretAccessKey,SessionToken]' \
      --output text)
    
    export AWS_ACCESS_KEY_ID=$(echo $CREDENTIALS | awk '{print $1}')
    export AWS_SECRET_ACCESS_KEY=$(echo $CREDENTIALS | awk '{print $2}')
    export AWS_SESSION_TOKEN=$(echo $CREDENTIALS | awk '{print $3}')
    
    # Create detector
    aws guardduty create-detector \
      --enable \
      --region $REGION
  fi
done

unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN
```

### Recipe 3: Create Findings Export Configuration

**Use Case**: Export GuardDuty findings to S3 for analysis.

```bash
# Create S3 bucket for exports
aws s3api create-bucket \
  --bucket guardduty-findings-export \
  --region us-east-1

# Create bucket policy for GuardDuty
aws s3api put-bucket-policy \
  --bucket guardduty-findings-export \
  --policy '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "AllowGuardDutyPut",
        "Effect": "Allow",
        "Principal": {
          "Service": "guardduty.amazonaws.com"
        },
        "Action": "s3:PutObject",
        "Resource": "arn:aws:s3:::guardduty-findings-export/*"
      }
    ]
  }'

# Create export configuration
aws guardduty create-publishing-destination \
  --detector-id $DETECTOR_ID \
  --destination-type S3 \
  --destination-properties DestinationArn=arn:aws:s3:::guardduty-findings-export
```

### Recipe 4: Set Up EventBridge Integration

**Use Case**: Real-time response to GuardDuty findings.

```bash
# Create SNS topic for alerts
aws sns create-topic --name guardduty-findings

# Create EventBridge rule
aws events put-rule \
  --name guardduty-high-severity \
  --event-pattern '{
    "source": ["aws.guardduty"],
    "detail-type": ["GuardDuty Finding"],
    "detail": {
      "severity": [7, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7, 7.8, 7.9, 8, 8.0, 8.1, 8.2, 8.3, 8.4, 8.5, 8.6, 8.7, 8.8, 8.9]
    }
  }' \
  --state ENABLED

# Add SNS as target
aws events put-targets \
  --rule guardduty-high-severity \
  --targets "Id"="1","Arn"="arn:aws:sns:us-east-1:123456789012:guardduty-findings"
```

---

## Security Hub Fundamentals

### What is Security Hub?
AWS Security Hub is a cloud security posture management (CSPM) and security information and event management (SIEM) service that aggregates security findings from multiple AWS services.

**Integrated Services:**
- GuardDuty (threat detection)
- IAM Access Analyzer (unintended access)
- AWS Config (resource compliance)
- AWS Firewall Manager (centralized security)
- Amazon Inspector (vulnerability scanning)
- Third-party tools (Prowler, Splunk, etc.)

**Key Features:**
- Compliance standards (CIS, PCI-DSS, NIST)
- Security insights and metrics
- Custom insights and filters
- Automated responses and workflows

---

## Security Hub Setup & Management

### Recipe 1: Enable Security Hub

```bash
# Enable Security Hub in primary account
aws securityhub enable-security-hub \
  --region us-east-1

# Create administrator account
aws securityhub create-admin-account \
  --admin-account-id 123456789012 \
  --region us-east-1

# Enable for organization
aws securityhub create-organization-admin-account \
  --admin-account-id 123456789012 \
  --region us-east-1
```

### Recipe 2: Enable Compliance Standards

**Use Case**: Evaluate against CIS AWS Foundations Benchmark.

```bash
# Get available standards
aws securityhub describe-standards \
  --query 'Standards[*].[Name,StandardsArn]' \
  --output table

# Enable CIS standard
CIS_ARN="arn:aws:securityhub:us-east-1::standards/service-managed/aws-foundational-security-best-practices"

aws securityhub batch-enable-standards \
  --standards-subscription-requests "StandardsArn=$CIS_ARN" \
  --region us-east-1

# List enabled standards
aws securityhub get-enabled-standards \
  --region us-east-1 \
  --query 'StandardsSubscriptions[*].[StandardsArn,ComplianceStatus]' \
  --output table
```

### Recipe 3: Create Custom Insights

**Use Case**: Track critical unresolved findings.

```bash
# High severity findings, not archived
aws securityhub create-insight \
  --name "High Severity Findings" \
  --filters '{
    "SeverityLabel": [
      {
        "Value": "HIGH",
        "Comparison": "EQUALS"
      },
      {
        "Value": "CRITICAL",
        "Comparison": "EQUALS"
      }
    ],
    "RecordState": [
      {
        "Value": "ACTIVE",
        "Comparison": "EQUALS"
      }
    ]
  }' \
  --group-by-attribute RESOURCE_ID \
  --region us-east-1

# Finding by resource type
aws securityhub create-insight \
  --name "IAM Findings" \
  --filters '{
    "ResourceType": [
      {
        "Value": "AwsIam",
        "Comparison": "EQUALS"
      }
    ]
  }' \
  --group-by-attribute SEVERITY \
  --region us-east-1

# Unresolved findings by age
aws securityhub create-insight \
  --name "Findings Over 30 Days Old" \
  --filters '{
    "FirstObservedAt": [
      {
        "Value": 2592000000,
        "Comparison": "LESS_THAN"
      }
    ],
    "RecordState": [
      {
        "Value": "ACTIVE",
        "Comparison": "EQUALS"
      }
    ]
  }' \
  --group-by-attribute RESOURCE_ID
```

### Recipe 4: Integrate with Third-Party SIEM

**Use Case**: Send findings to Splunk.

```bash
# Create EventBridge rule for all findings
aws events put-rule \
  --name security-hub-to-splunk \
  --event-pattern '{
    "source": ["aws.securityhub"],
    "detail-type": ["Security Hub Findings - Imported"]
  }' \
  --state ENABLED

# Send to Lambda for transformation
aws events put-targets \
  --rule security-hub-to-splunk \
  --targets "Id"="1","Arn"="arn:aws:lambda:us-east-1:123456789012:function:send-to-splunk"
```

**Lambda Function (send_to_splunk.py):**
```python
import json
import urllib3
import os

http = urllib3.PoolManager()

def lambda_handler(event, context):
    findings = event['detail']['findings']
    splunk_endpoint = os.environ['SPLUNK_ENDPOINT']
    splunk_token = os.environ['SPLUNK_TOKEN']
    
    for finding in findings:
        # Transform finding to Splunk format
        body = json.dumps({
            'event': {
                'finding_id': finding['Id'],
                'severity': finding['Severity']['Label'],
                'title': finding['Title'],
                'resource_type': finding['Resources'][0]['Type'],
                'created_at': finding['FirstObservedAt']
            },
            'sourcetype': 'security_hub'
        })
        
        r = http.request(
            'POST',
            splunk_endpoint,
            body=body,
            headers={'Authorization': f'Splunk {splunk_token}'}
        )
    
    return {'statusCode': 200}
```

---

## Threat Response Recipes

### Scenario 1: Response to EC2 Compromise Finding

**Finding Type**: "Trojan.EC2/BlackholeTraffic!DNS"

**Manual Response Steps:**
1. Isolate instance (modify security group)
2. Create snapshot for forensics
3. Notify security team
4. Gather logs and network data
5. Terminate instance after analysis

**Automated Response (SSM Document):**
```json
{
  "schemaVersion": "0.3",
  "description": "Isolate potentially compromised EC2 instance",
  "parameters": {
    "InstanceId": {
      "type": "String",
      "description": "EC2 Instance ID"
    }
  },
  "mainSteps": [
    {
      "name": "CreateSnapshot",
      "action": "aws:executeAwsApi",
      "onFailure": "Continue",
      "inputs": {
        "Service": "ec2",
        "Api": "CreateSnapshots",
        "InstanceSpecifications": [
          {
            "InstanceId": "{{ InstanceId }}"
          }
        ]
      }
    },
    {
      "name": "CreateSecurityGroup",
      "action": "aws:executeAwsApi",
      "inputs": {
        "Service": "ec2",
        "Api": "CreateSecurityGroup",
        "GroupName": "isolation-sg",
        "GroupDescription": "Isolation security group for compromised instance",
        "VpcId": "vpc-12345678"
      }
    },
    {
      "name": "ModifyInstanceSecurityGroup",
      "action": "aws:executeAwsApi",
      "inputs": {
        "Service": "ec2",
        "Api": "ModifyInstanceAttribute",
        "InstanceId": "{{ InstanceId }}",
        "Groups": [
          "sg-isolation"
        ]
      }
    }
  ]
}
```

### Scenario 2: Response to Credential Exposure

**Finding Type**: "PrincipalPolicy.IAM/UnauthorizedAPICallsRecon"

**Steps:**
```bash
#!/bin/bash
# Script: respond-to-credential-exposure.sh

ACCESS_KEY=$1
FINDING_ID=$2

# Deactivate exposed access key
aws iam update-access-key-status \
  --access-key-id $ACCESS_KEY \
  --status Inactive

# Get key owner
KEY_USER=$(aws iam get-access-key-last-used \
  --access-key-id $ACCESS_KEY \
  --query 'UserName' \
  --output text)

# Create new key for user
NEW_KEY=$(aws iam create-access-key \
  --user-name $KEY_USER \
  --query 'AccessKey.[AccessKeyId,SecretAccessKey]' \
  --output text)

# Update Finding status
aws securityhub update-findings \
  --finding-identifiers "Id=$FINDING_ID,ProductArn=arn:aws:securityhub:us-east-1:123456789012:product/123456789012/default" \
  --note "Text=Key deactivated, new key issued to $KEY_USER" \
  --workflow-status RESOLVED

# Send notification
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:SecurityTeam \
  --message "Access key $ACCESS_KEY was exposed and deactivated. New key issued to $KEY_USER."
```

### Scenario 3: Response to Overly Permissive IAM Policy

**Finding Type**: "IAMAccess.PolicyExposure"

**Automated Remediation (Lambda):**
```python
import boto3
import json

iam = boto3.client('iam')
security_hub = boto3.client('securityhub')

def lambda_handler(event, context):
    finding = event['detail']['findings'][0]
    
    # Extract policy details
    policy_arn = finding['Resources'][0]['Id']
    
    # Check if policy allows all actions
    policy = iam.get_policy(PolicyArn=policy_arn)
    version = iam.get_policy_version(
        PolicyArn=policy_arn,
        VersionId=policy['Policy']['DefaultVersionId']
    )
    
    doc = version['PolicyVersion']['Document']
    
    # Check for overly permissive statements
    for statement in doc['Statement']:
        if statement.get('Effect') == 'Allow':
            actions = statement.get('Action', [])
            resources = statement.get('Resource', [])
            
            if actions == '*' and resources == '*':
                # This is dangerous, restrict it
                # Remove policy from users/roles
                update_policy(policy_arn)
                
                # Update finding
                security_hub.update_findings(
                    FindingIdentifiers=[{
                        'Id': finding['Id'],
                        'ProductArn': finding['ProductArn']
                    }],
                    Note={'Text': 'Policy auto-remediated: removed wildcard permissions'},
                    Workflow={'Status': 'RESOLVED'}
                )
    
    return {'statusCode': 200}

def update_policy(policy_arn):
    # Create restricted policy version
    restricted_policy = {
        'Version': '2012-10-17',
        'Statement': [
            {
                'Effect': 'Allow',
                'Action': [
                    's3:GetObject',
                    's3:ListBucket'
                ],
                'Resource': 'arn:aws:s3:::my-bucket/*'
            }
        ]
    }
    
    iam.create_policy_version(
        PolicyArn=policy_arn,
        PolicyDocument=json.dumps(restricted_policy),
        SetAsDefault=True
    )
```

---

## Automated Response Patterns

### Pattern 1: EventBridge → Lambda → Remediation

```bash
# Create EventBridge rule for specific findings
aws events put-rule \
  --name guardduty-ec2-compromise \
  --event-pattern '{
    "source": ["aws.guardduty"],
    "detail-type": ["GuardDuty Finding"],
    "detail": {
      "type": ["Trojan.EC2*"]
    }
  }'

# Target Lambda function
aws events put-targets \
  --rule guardduty-ec2-compromise \
  --targets "Id"="1","Arn"="arn:aws:lambda:us-east-1:123456789012:function:isolate-instance"
```

### Pattern 2: Finding Aggregation Dashboard

**CloudWatch Dashboard JSON:**
```json
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "metrics": [
          ["AWS/SecurityHub", "ComplianceScore", {"stat": "Average"}],
          [".", "FindingsCount", {"stat": "Sum"}],
          [".", "HighSeverityFindings", {"stat": "Sum"}]
        ],
        "period": 300,
        "stat": "Average",
        "region": "us-east-1",
        "title": "Security Hub Metrics"
      }
    }
  ]
}
```

### Pattern 3: Notification and Escalation

```bash
# Create SNS topics for different severity levels
aws sns create-topic --name security-alerts-critical
aws sns create-topic --name security-alerts-high
aws sns create-topic --name security-alerts-medium

# EventBridge rules for escalation
for severity in CRITICAL HIGH MEDIUM; do
  topic_name="security-alerts-${severity,,}"
  aws events put-rule \
    --name "guardduty-$severity" \
    --event-pattern "{
      \"source\": [\"aws.guardduty\"],
      \"detail\": {\"severity\": [\"$severity\"]}
    }"
  
  aws events put-targets \
    --rule "guardduty-$severity" \
    --targets "Id"="1","Arn"="arn:aws:sns:us-east-1:123456789012:$topic_name"
done
```

---

## Best Practices Checklist

- [ ] Enable GuardDuty on all accounts and regions
- [ ] Set up centralized management with admin account
- [ ] Export findings to S3 for long-term retention
- [ ] Create EventBridge rules for real-time alerts
- [ ] Enable Security Hub across all accounts
- [ ] Deploy all relevant compliance standards
- [ ] Create custom insights for your organization
- [ ] Integrate with SIEM (Splunk, ELK, etc.)
- [ ] Document response procedures for each finding type
- [ ] Automate remediation where possible
- [ ] Review and triage findings at least daily
- [ ] Archive resolved findings regularly
- [ ] Test incident response playbooks quarterly

---

## Troubleshooting

**GuardDuty Not Detecting Activity?**
- Ensure VPC Flow Logs are enabled
- Verify CloudTrail is logging
- Check detector is enabled: `aws guardduty get-detector`

**Security Hub Not Showing Findings?**
- Verify integrated services are enabled
- Check standards are subscribed
- Ensure findings are being imported: `aws securityhub get-findings`

**High False Positive Rate?**
- Create custom insights to filter noise
- Use Archives to suppress expected findings
- Adjust insight filters to be more specific
