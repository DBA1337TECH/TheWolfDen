# AWS Security Guide 1: Logging and Visibility
## CloudTrail & AWS Config

---

## Table of Contents
1. [CloudTrail Fundamentals](#cloudtrail-fundamentals)
2. [CloudTrail Setup & Configuration](#cloudtrail-setup--configuration)
3. [AWS Config Fundamentals](#aws-config-fundamentals)
4. [AWS Config Setup & Rules](#aws-config-setup--rules)
5. [Recipes & Common Scenarios](#recipes--common-scenarios)
6. [Monitoring & Analysis](#monitoring--analysis)

---

## CloudTrail Fundamentals

### What is CloudTrail?
CloudTrail logs all API calls made in your AWS account, providing an audit trail for compliance, security monitoring, and troubleshooting.

**Key Concepts:**
- **Management Events**: API calls that modify resources (CreateBucket, ModifyDBInstance, etc.)
- **Data Events**: High-volume API calls like S3 GetObject/PutObject, Lambda Invoke
- **Insight Events**: Unusual patterns detected by CloudTrail (optional, paid)
- **Trail**: A configuration that logs events to S3 and/or CloudWatch Logs

### Why You Need It
- Compliance (PCI-DSS, HIPAA, SOC 2)
- Incident response and forensics
- Detecting unauthorized access
- Auditing account activity

---

## CloudTrail Setup & Configuration

### Recipe 1: Create an Organization Trail (Multi-Account)

**Use Case**: Monitor all accounts in an AWS Organization centrally.

**Steps:**

1. **Enable Organization Trail (Root Account)**
   ```bash
   aws cloudtrail create-organization-trail \
     --name org-trail \
     --s3-bucket-name my-org-cloudtrail-logs \
     --is-multi-region-trail \
     --enable-log-file-validation \
     --include-global-service-events
   ```

2. **Create S3 Bucket with Proper Permissions**
   ```bash
   aws s3api create-bucket \
     --bucket my-org-cloudtrail-logs \
     --region us-east-1
   
   # Attach bucket policy
   aws s3api put-bucket-policy \
     --bucket my-org-cloudtrail-logs \
     --policy '{
       "Version": "2012-10-17",
       "Statement": [
         {
           "Sid": "AWSCloudTrailAclCheck",
           "Effect": "Allow",
           "Principal": {
             "Service": "cloudtrail.amazonaws.com"
           },
           "Action": "s3:GetBucketAcl",
           "Resource": "arn:aws:s3:::my-org-cloudtrail-logs"
         },
         {
           "Sid": "AWSCloudTrailWrite",
           "Effect": "Allow",
           "Principal": {
             "Service": "cloudtrail.amazonaws.com"
           },
           "Action": "s3:PutObject",
           "Resource": "arn:aws:s3:::my-org-cloudtrail-logs/*",
           "Condition": {
             "StringEquals": {
               "s3:x-amz-acl": "bucket-owner-full-control"
             }
           }
         }
       ]
     }'
   ```

3. **Enable CloudTrail (AWS Console)**
   - Go to CloudTrail → Trails
   - Select organization trail
   - Click "Start logging"

### Recipe 2: Set Up Data Event Logging for S3

**Use Case**: Monitor every object access on sensitive S3 buckets.

```bash
# Update trail to include S3 data events
aws cloudtrail put-event-selectors \
  --trail-name org-trail \
  --event-selectors '[
    {
      "ReadWriteType": "All",
      "IncludeManagementEvents": true,
      "DataResources": [
        {
          "Type": "AWS::S3::Object",
          "Values": ["arn:aws:s3:::sensitive-bucket/*"]
        }
      ]
    }
  ]'
```

**Cost Optimization Note**: Data events cost $0.10 per 100,000 events. Consider selective logging on critical buckets only.

### Recipe 3: Configure CloudWatch Logs Integration

**Use Case**: Real-time alerts when suspicious activity occurs.

```bash
# Create IAM role for CloudTrail → CloudWatch Logs
aws iam create-role \
  --role-name CloudTrailCloudWatchLogsRole \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "Service": "cloudtrail.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
      }
    ]
  }'

# Create log group
aws logs create-log-group \
  --log-group-name /aws/cloudtrail/security-monitoring

# Update trail
aws cloudtrail update-trail \
  --name org-trail \
  --cloud-watch-logs-log-group-arn arn:aws:logs:us-east-1:123456789012:log-group:/aws/cloudtrail/security-monitoring:* \
  --cloud-watch-logs-role-arn arn:aws:iam::123456789012:role/CloudTrailCloudWatchLogsRole
```

---

## AWS Config Fundamentals

### What is AWS Config?
AWS Config continuously monitors and records configurations of AWS resources and evaluates them against desired configurations (rules).

**Key Concepts:**
- **Configuration Items**: Snapshots of resource configurations
- **Configuration History**: Timeline of resource changes
- **Conformance Packs**: Collections of Config rules for compliance
- **Config Rules**: Automated checks (AWS-managed or custom)

### Why You Need It
- Compliance auditing (CIS Benchmarks, PCI-DSS)
- Change tracking and troubleshooting
- Security and configuration drift detection
- Automated remediation

---

## AWS Config Setup & Rules

### Recipe 1: Enable AWS Config

**Steps:**

```bash
# Create S3 bucket for Config
aws s3api create-bucket \
  --bucket my-aws-config-bucket \
  --region us-east-1

# Enable AWS Config
aws configservice put-config-service-role \
  --config-service-role-arn arn:aws:iam::123456789012:role/ConfigRole

# Start recorder
aws configservice start-configuration-recorder \
  --configuration-recorder-names default

# Deliver to S3
aws configservice put-delivery-channel \
  --delivery-channel name=default,s3BucketName=my-aws-config-bucket
```

### Recipe 2: Deploy CIS AWS Foundations Conformance Pack

**Use Case**: Enforce CIS Benchmarks automatically.

```bash
# Deploy conformance pack
aws configservice put-conformance-pack \
  --conformance-pack-name cis-aws-foundations \
  --delivery-s3-bucket my-aws-config-bucket \
  --template-s3-uri s3://aws-configservice-us-east-1/conformance-packs/cis-aws-foundations.yaml
```

### Recipe 3: Create Custom Config Rule for S3 Encryption

**Use Case**: Ensure all S3 buckets have encryption enabled.

```bash
# Create Lambda for custom rule
aws lambda create-function \
  --function-name s3-encryption-check \
  --runtime python3.9 \
  --role arn:aws:iam::123456789012:role/ConfigRuleLambda \
  --handler index.lambda_handler \
  --zip-file fileb://lambda_function.zip

# Create Config rule
aws configservice put-config-rule \
  --config-rule '{
    "ConfigRuleName": "s3-bucket-encryption-enabled",
    "Description": "Checks if S3 buckets have encryption enabled",
    "Source": {
      "Owner": "LAMBDA",
      "SourceIdentifier": "arn:aws:lambda:us-east-1:123456789012:function:s3-encryption-check"
    }
  }'
```

**Lambda Function (lambda_function.py):**
```python
import boto3
import json

s3 = boto3.client('s3')
config = boto3.client('config')

def lambda_handler(event, context):
    config_item = json.loads(event['configurationItem'])
    bucket_name = config_item['resourceName']
    
    # Check encryption
    try:
        encryption = s3.get_bucket_encryption(Bucket=bucket_name)
        compliance_status = 'COMPLIANT'
    except s3.exceptions.ServerSideEncryptionConfigurationNotFoundError:
        compliance_status = 'NON_COMPLIANT'
    
    return {
        'compliance': compliance_status,
        'resourceName': bucket_name,
        'resourceType': 'AWS::S3::Bucket'
    }
```

---

## Recipes & Common Scenarios

### Scenario 1: Detect Unauthorized API Calls

**CloudTrail Query (Athena)**
```sql
SELECT eventtime, useragent, sourceipaddress, eventname, errorcode
FROM cloudtrail_logs
WHERE errorcode IS NOT NULL
  AND eventtime > date_format(current_timestamp - interval '24' hour, '%Y-%m-%dT%H:%i:%SZ')
ORDER BY eventtime DESC;
```

### Scenario 2: Track Root Account Activity

**CloudTrail Query**
```sql
SELECT eventtime, eventname, sourceipaddress, requestparameters
FROM cloudtrail_logs
WHERE useridentity.principalid = 'AIDACKCEVSQ6C2EXAMPLE'
  OR useridentity.type = 'Root'
ORDER BY eventtime DESC;
```

### Scenario 3: Monitor Cross-Account Access

**CloudTrail Query**
```sql
SELECT eventtime, useragent, sourceipaddress, eventname, recipientaccountid
FROM cloudtrail_logs
WHERE recipientaccountid != '<your-account-id>'
ORDER BY eventtime DESC;
```

### Scenario 4: Find Failed Authentication Attempts

```sql
SELECT eventtime, sourceipaddress, eventname, errorcode, errormessage
FROM cloudtrail_logs
WHERE errorcode IN ('UnauthorizedOperation', 'AccessDenied', 'InvalidUserID.NotFound')
  AND eventtime > date_format(current_timestamp - interval '7' day, '%Y-%m-%dT%H:%i:%SZ')
ORDER BY sourceipaddress, eventtime DESC;
```

---

## Monitoring & Analysis

### Set Up CloudWatch Alarms

**Alert on Root Account Usage:**
```bash
# Create metric filter
aws logs put-metric-filter \
  --log-group-name /aws/cloudtrail/security-monitoring \
  --filter-name RootAccountUsage \
  --filter-pattern '{ $.userIdentity.type = "Root" }' \
  --metric-transformations \
      metricName=RootAccountUsageCount,metricNamespace=CloudTrailMetrics,metricValue=1

# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name root-account-usage \
  --alarm-description "Alert when root account is used" \
  --metric-name RootAccountUsageCount \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:SecurityAlerts
```

**Alert on Unauthorized API Calls:**
```bash
aws logs put-metric-filter \
  --log-group-name /aws/cloudtrail/security-monitoring \
  --filter-name UnauthorizedAPICallsMetricFilter \
  --filter-pattern '{ ($.errorCode = "*UnauthorizedOperation") || ($.errorCode = "AccessDenied*") }' \
  --metric-transformations \
      metricName=UnauthorizedAPICallsCount,metricNamespace=CloudTrailMetrics,metricValue=1
```

### Best Practices Checklist

- [ ] Enable multi-region trails for all accounts
- [ ] Enable log file validation on all trails
- [ ] Encrypt CloudTrail logs at rest using KMS
- [ ] Set up S3 bucket versioning and MFA delete protection
- [ ] Enable CloudWatch Logs integration for real-time monitoring
- [ ] Query CloudTrail logs regularly with Athena
- [ ] Configure SNS notifications for critical events
- [ ] Archive CloudTrail logs to Glacier after 90 days
- [ ] Enable AWS Config on all accounts and regions
- [ ] Deploy conformance packs for compliance frameworks
- [ ] Set up automated remediation for Config violations
- [ ] Review CloudTrail and Config findings weekly

---

## Troubleshooting

**Trail Not Logging?**
- Check S3 bucket policy allows CloudTrail access
- Verify IAM role has CloudWatchLogs permissions
- Ensure trail is started: `aws cloudtrail start-configuration-recorder`

**Config Rules Not Evaluating?**
- Check Config recorder is running
- Verify Lambda execution role has permissions
- Review Config delivery channel S3 bucket access
