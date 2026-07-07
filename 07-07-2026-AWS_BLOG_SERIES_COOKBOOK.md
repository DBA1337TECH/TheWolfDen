# AWS Security Guides - Partial Handbook
## Comprehensive Coverage of AWS Security Tools & Services
## From yours truly - Oblivion Edge Vulnerability Research LLC

---

## Guide Overview

This handbook provides practical, hands-on guides for securing AWS accounts using security services and tools. Each guide is designed as a **cookbook** with real-world recipes, step-by-step instructions, and deployable scripts.

### Your Learning Path

```
START HERE
    ↓
[Guide 0] AWS CLI Fundamentals ─────────┐
    ↓                                    │
[Guide 1] Logging & Visibility           │
    CloudTrail & AWS Config              │
    ↓                                    │
[Guide 2] Threat Detection & Response    │
    GuardDuty & Security Hub             │
    ↓                                    │
[Guide 3] DDoS & Web Protection          │
    AWS WAF & AWS Shield                 │
    ↓                                    │
[Guide 4] S3 Security                    │
    Auditing & Securing Buckets          │
    ↓                                    └──→ MASTERY
[Review & Practice]                         
```

---

## Guide Descriptions

### Guide 0: AWS CLI Fundamentals
**File**: `AWS_Security_Guide_0_AWS_CLI_Fundamentals.md`

**What You'll Learn:**
- What AWS CLI is and why you need it
- Installation on Linux, macOS, and Windows
- Credential configuration (access keys, profiles, IAM roles)
- Basic commands and command structure
- Filtering and querying with JQ
- Common patterns and scripts
- Troubleshooting guide

**Key Takeaway**: Master the tool you'll use for all other guides.

**Time Investment**: 30-45 minutes

**Prerequisites**: None

**Next Step**: → Guide 1

---

### Guide 1: Logging & Visibility
**File**: `AWS_Security_Guide_1_Logging_and_Visibility.md`

**Services Covered:**
- **CloudTrail** - API audit logging
- **AWS Config** - Resource configuration monitoring

**What You'll Learn:**
- Create organization trails for multi-account logging
- Set up data event logging for S3
- Configure CloudWatch Logs integration
- Deploy AWS Config rules and conformance packs
- Create custom compliance rules
- Query CloudTrail logs with Athena
- Set up CloudWatch alarms
- Detect unauthorized API calls and suspicious activity

**Key Recipes:**
- Create an Organization Trail
- Set Up Data Event Logging for S3
- Configure CloudWatch Logs Integration
- Deploy CIS AWS Foundations Conformance Pack
- Create Custom Config Rule for S3 Encryption
- Detect Unauthorized API Calls
- Track Root Account Activity
- Monitor Cross-Account Access

**Recipes Include**: 20+ ready-to-use scripts and configurations

**Time Investment**: 2-3 hours

**Prerequisites**: Guide 0 (AWS CLI)

**Next Step**: → Guide 2

**Why Start Here?**
- Foundation for all security monitoring
- Required for compliance (PCI-DSS, HIPAA, SOC 2)
- Enables incident response and forensics
- Builds understanding of what's happening in your account

---

### Guide 2: Threat Detection & Response
**File**: `AWS_Security_Guide_2_Threat_Detection_and_Response.md`

**Services Covered:**
- **GuardDuty** - Managed threat detection
- **Security Hub** - Centralized security findings

**What You'll Learn:**
- Enable GuardDuty for organization with centralized management
- Export findings to S3 for analysis
- Set up EventBridge integration for real-time alerts
- Enable Security Hub across all accounts
- Deploy compliance standards (CIS, PCI-DSS, NIST)
- Create custom insights for threat tracking
- Integrate with SIEM systems (Splunk, ELK)
- Automated threat response with Lambda
- Incident response playbooks

**Key Recipes:**
- Enable GuardDuty for Organization
- Centralized Member Account Management
- Create Findings Export Configuration
- Set Up EventBridge Integration
- Enable Security Hub
- Enable Compliance Standards
- Create Custom Insights
- Integrate with Third-Party SIEM
- Response to EC2 Compromise
- Response to Credential Exposure
- Response to Overly Permissive IAM Policy

**Recipes Include**: 15+ detection and response scripts

**Time Investment**: 2-3 hours

**Prerequisites**: Guide 0 (AWS CLI), Guide 1 (recommended but not required)

**Next Step**: → Guide 3

**Why This Matters?**
- GuardDuty detects threats CloudTrail doesn't catch
- Security Hub aggregates findings from multiple sources
- Automated response saves hours during incident
- Machine learning identifies patterns humans miss

---

### Guide 3: DDoS & Web Protection
**File**: `AWS_Security_Guide_3_DDoS_and_Web_Protection.md`

**Services Covered:**
- **AWS Shield** - DDoS protection (Standard & Advanced)
- **AWS WAF** - Web Application Firewall

**What You'll Learn:**
- Understand Shield Standard vs. Advanced
- Protect CloudFront and load balancers
- Set up DDoS monitoring and alerts
- Create WAF Web ACLs with managed rules
- Deploy OWASP Top 10 protections
- Implement rate limiting and geo-blocking
- Create custom XSS and SQL injection rules
- Bot management strategies
- DDoS attack response playbooks
- WAF rule testing and validation

**Key Recipes:**
- Enable Shield Advanced
- Protect CloudFront Distribution
- Protect Load Balancer with Health Monitoring
- Configure DDoS Alerts
- Create Basic Web ACL
- Deploy AWS Managed Rules
- Create Custom Rate Limiting Rule
- Geo-Blocking Configuration
- Custom XSS Detection
- Bot Management
- WAF Logging to S3
- DDoS Attack Response Playbook

**Recipes Include**: 12+ protection and testing scripts

**Time Investment**: 2-3 hours

**Prerequisites**: Guide 0 (AWS CLI)

**Can Take Before**: Guide 2 (if protecting web apps is priority)

**Why This Matters?**
- 90% of website attacks are L7 (WAF territory)
- DDoS attacks can take down applications
- Shield Advanced pays for itself if attacked
- Managed rules provide immediate protection

---

### Guide 4: S3 Security
**File**: `AWS_Security_Guide_4_S3_Bucket_Security.md`

**Focus Area**: 
- Secure S3 bucket configuration
- Access control
- Encryption and data protection
- Auditing and monitoring

**What You'll Learn:**
- Create secure S3 buckets from scratch
- Block public access at bucket and account level
- Implement proper versioning and MFA Delete
- Encryption strategies (SSE-S3, KMS)
- Cross-account access patterns
- Role-based access control
- S3 access logging and CloudTrail integration
- CloudWatch alarms for security events
- Compliance assessments
- Security audit scripts

**Key Recipes:**
- Create Secure S3 Bucket from Scratch
- Audit Existing Bucket for Compliance
- Cross-Account S3 Access
- Restrict Access to Specific IAM Roles
- Enforce Specific User Agent
- Enable KMS Encryption for Sensitive Data
- Enable Object Versioning & MFA Delete
- Retain Compliance with Object Lock
- Set Up Comprehensive S3 Monitoring
- Create S3 Access Logs Query
- CloudWatch Alarms for Security Events
- Find Publicly Accessible Buckets
- Compliance Check Against CIS Benchmarks
- Generate S3 Security Report

**Recipes Include**: 14+ automation and auditing scripts

**Time Investment**: 2-3 hours

**Prerequisites**: Guide 0 (AWS CLI)

**Can Take Anytime After**: Guide 0

**Why S3?**
- S3 is most commonly misconfigured AWS service
- Data breaches often trace to S3 misconfiguration
- Affects 40%+ of AWS deployments
- Compliance frameworks focus on S3 security

---

## Quick Reference Table

| Guide | Services | Focus | Duration | Difficulty | Priority |
|-------|----------|-------|----------|------------|----------|
| 0 | CLI | Setup & Basics | 30-45m | ⭐ | MUST FIRST |
| 1 | CloudTrail, Config | Logging & Visibility | 2-3h | ⭐⭐ | HIGH |
| 2 | GuardDuty, Security Hub | Threat Detection | 2-3h | ⭐⭐⭐ | HIGH |
| 3 | WAF, Shield | DDoS & Web App | 2-3h | ⭐⭐ | MEDIUM |
| 4 | S3 | Bucket Security | 2-3h | ⭐⭐ | HIGH |

---

## Getting Started

### Minimum Setup (2-3 hours)
1. **Guide 0**: AWS CLI Fundamentals (30-45 min)
2. **Guide 1**: Logging & Visibility (2-3 hours)
3. Test: Run CloudTrail query, verify logs in S3

### Recommended Full Path (8-12 hours)
1. **Guide 0**: AWS CLI Fundamentals (30-45 min)
2. **Guide 1**: Logging & Visibility (2-3 hours)
3. **Guide 2**: Threat Detection (2-3 hours)
4. **Guide 3**: DDoS Protection (2-3 hours)
5. **Guide 4**: S3 Security (2-3 hours)
6. Test: Run all security scripts against your account

### Fast Track by Role (Pick Your Path)

**Security Engineer / Architect**
- Guide 0 → Guide 1 → Guide 2 → (Optional: 3 & 4)

**DevOps / Site Reliability Engineer**
- Guide 0 → Guide 1 → Guide 4 → (Optional: 2 & 3)

**Cloud Developer**
- Guide 0 → Guide 4 → Guide 3 → (Optional: 1 & 2)

**Compliance / Audit Professional**
- Guide 0 → Guide 1 → Guide 4 → Guide 2

---

## What's Included in Each Guide

### Standard Components (Every Guide)
Fundamentals section (what, why, when)  
Setup and configuration  
10-15 ready-to-use recipes  
Real-world scenarios  
Automated response patterns  
Troubleshooting section  
Best practices checklist  
Security considerations  

### Bonus Features
Bash scripts (copy-paste ready)  
Configuration examples  
Performance optimization tips  
Cost optimization guidance  
Common errors and solutions  
Monitoring and alerting setup  
Security best practices  
Compliance framework mapping  

---

## Learning Outcomes

After completing all guides, you will be able to:

### Knowledge
- [ ] Explain each AWS security service and its purpose
- [ ] Understand attack vectors and mitigation strategies
- [ ] Design multi-account security architectures
- [ ] Map compliance frameworks to AWS services

### Skills
- [ ] Deploy security services across AWS accounts
- [ ] Write automation scripts for security operations
- [ ] Configure advanced monitoring and alerting
- [ ] Respond to security incidents
- [ ] Audit configurations for compliance
- [ ] Query logs and findings for analysis
- [ ] Troubleshoot security-related issues

### Hands-On Competencies
- [ ] Create and manage CloudTrail for auditing
- [ ] Deploy GuardDuty and Security Hub
- [ ] Implement WAF rules and DDoS protection
- [ ] Secure S3 buckets against common attacks
- [ ] Automate security responses with Lambda
- [ ] Integrate with SIEM and monitoring tools
- [ ] Test security configurations

---

## Tools You'll Use

| Tool | Purpose | When Used |
|------|---------|-----------|
| **AWS CLI** | Command-line access to AWS | All guides |
| **AWS Console** | Web UI for manual configuration | Guides 0-4 |
| **Bash/PowerShell** | Script automation | All guides |
| **JQ** | JSON query and filtering | All guides |
| **Athena** | Query CloudTrail logs | Guide 1 |
| **CloudWatch** | Metrics and alarms | Guides 1-3 |
| **EventBridge** | Event routing and automation | Guides 2-3 |
| **Lambda** | Serverless automation | Guides 2-4 |
| **IAM** | Access control | Guides 2-4 |

---

## Related AWS Services (Brief Overview)

These services interact with your security setup:

- **CloudWatch** - Metrics and alarms for all services
- **SNS/SQS** - Notifications and messaging
- **Lambda** - Automated response functions
- **EventBridge** - Event routing and automation
- **Systems Manager** - Patch and command automation
- **Inspector** - Vulnerability scanning
- **Access Analyzer** - Unintended access detection
- **Macie** - Data discovery and protection

---

## Quick Commands Cheat Sheet

```bash
# Verify credentials
aws sts get-caller-identity

# List accounts in organization
aws organizations list-accounts

# List all CloudTrail trails
aws cloudtrail describe-trails

# Check GuardDuty status
aws guardduty list-detectors

# List Security Hub findings
aws securityhub get-findings

# Check WAF Web ACLs
aws wafv2 list-web-acls --scope REGIONAL

# List S3 buckets
aws s3 ls

# Test specific profile
aws sts get-caller-identity --profile prod-account

# Enable debugging
aws s3 ls --debug
```

---

## Security Reminders

**Critical Security Practices:**
- Never commit AWS credentials to git
- Rotate access keys every 90 days
- Use IAM roles, not long-term credentials when possible
- Enable MFA on all user accounts
- Apply principle of least privilege
- Audit access regularly
- Monitor CloudTrail for suspicious activity
- Document all security configurations

---

## Support & Resources

### AWS Documentation
- **AWS CLI Reference**: https://aws.amazon.com/cli/
- **AWS Security Best Practices**: https://aws.amazon.com/security/best-practices/
- **AWS Well-Architected Security Pillar**: https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html
- **AWS IAM Best Practices**: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html

### Community & Learning
- **AWS Security Blog**: https://aws.amazon.com/blogs/security/
- **AWS Training and Certification**: https://aws.amazon.com/training/
- **AWS Free Tier**: Try services for free (limited)

### Troubleshooting
- **AWS Service Health Dashboard**: https://health.aws.amazon.com
- **AWS Support Center**: https://console.aws.amazon.com/support/
- **AWS Forums**: https://forums.aws.amazon.com/

---

## Progression Tracking

Use this checklist to track your progress:

### Guide 0: AWS CLI Fundamentals
- [ ] Installed AWS CLI v2
- [ ] Configured credentials
- [ ] Ran first AWS CLI command
- [ ] Tested multiple profiles
- [ ] Installed and tested JQ

### Guide 1: Logging & Visibility
- [ ] Created organization trail
- [ ] Enabled data event logging
- [ ] Set up CloudWatch Logs integration
- [ ] Created Config rules
- [ ] Deployed conformance pack
- [ ] Queried CloudTrail logs
- [ ] Set up CloudWatch alarms

### Guide 2: Threat Detection & Response
- [ ] Enabled GuardDuty for organization
- [ ] Exported findings to S3
- [ ] Enabled Security Hub
- [ ] Deployed compliance standards
- [ ] Created custom insights
- [ ] Set up EventBridge rules
- [ ] Tested automated response

### Guide 3: DDoS & Web Protection
- [ ] Subscribed to Shield Advanced
- [ ] Protected critical resources
- [ ] Created WAF Web ACL
- [ ] Deployed managed rules
- [ ] Implemented rate limiting
- [ ] Set up WAF logging
- [ ] Tested rules in count mode

### Guide 4: S3 Security
- [ ] Created secure bucket
- [ ] Audited existing buckets
- [ ] Implemented encryption
- [ ] Enabled versioning
- [ ] Set up access logging
- [ ] Created compliance report
- [ ] Ran public bucket scanner

---

## Next Steps After Guides

**Level 2 Topics:**
- Multi-account architecture
- Cross-region security
- Disaster recovery
- Cost optimization
- Performance tuning
- Advanced automation

**Certifications:**
- AWS Certified Security - Specialty
- AWS Certified Solutions Architect - Professional

**Community Contribution:**
- Share your security automation scripts
- Create additional guides for specialized topics
- Contribute to AWS open-source security projects

---

## Final Notes

These guides are **practical first**, theoretical second. Every recipe has been tested and is ready to run. As you work through the guides:

**DO:**
- Type out commands to learn them
- Modify scripts for your environment
- Test in non-production first
- Document your security setup
- Share learnings with your team

**DON'T:**
- Copy-paste blindly
- Run on production without testing
- Ignore error messages
- Skip the troubleshooting sections
- Use overly permissive policies

---

## File List

```
AWS_SECURITY_GUIDES_README.md               ← You are here
AWS_Security_Guide_0_AWS_CLI_Fundamentals.md
AWS_Security_Guide_1_Logging_and_Visibility.md
AWS_Security_Guide_2_Threat_Detection_and_Response.md
AWS_Security_Guide_3_DDoS_and_Web_Protection.md
AWS_Security_Guide_4_S3_Bucket_Security.md
```

---

## Ready to Begin?

Start with **Guide 0: AWS CLI Fundamentals** if you haven't already, then progress through the guides in order.

**Happy Building!**

---

*Last Updated: 2026-07-06*  
*AWS Guides Version: 1.0*  
*Compatible with: AWS CLI v2, Python 3.7+*
