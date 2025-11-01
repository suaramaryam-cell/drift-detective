# drift-detective

> Automatically detects when AWS resources drift from their Terraform-defined state and sends alerts before configuration mismatches cause production issues.

## Problem Statement

In cloud environments, manual changes made through the AWS Console often bypass Infrastructure as Code (IaC) workflows, creating **configuration drift**. This leads to:

-  Failed deployments when Terraform state doesn't match reality
-  Security vulnerabilities from undocumented changes
-  Compliance issues during audits
-  Wasted time debugging "mysterious" configuration differences

**Drift Detective** solves this by automatically scanning AWS resources every 6 hours, comparing them against Terraform state, and alerting teams to any discrepancies.

##  Architecture

```
┌─────────────────┐
│   Terraform     │
│   State (S3)    │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────┐
│    EventBridge (Scheduled Trigger)  │
│         Every 6 Hours               │
└────────────┬────────────────────────┘
             │
             ▼
┌────────────────────────────────────┐
│      Lambda Function (Python)       │
│  ┌──────────────────────────────┐  │
│  │ 1. Read Terraform State      │  │
│  │ 2. Query AWS Resources       │  │
│  │ 3. Compare Attributes        │  │
│  │ 4. Detect Drift              │  │
│  └──────────────────────────────┘  │
└────────┬────────────┬──────────────┘
         │            │
         ▼            ▼
┌─────────────┐  ┌──────────────┐
│     SNS     │  │  CloudWatch  │
│   (Email)   │  │     Logs     │
└─────────────┘  └──────────────┘
```

### Current Capabilities
-  **EC2 Instance Drift Detection**
  - Instance type changes
  - Tag modifications (added/removed/changed)
  - Security group associations
  - Monitoring state changes
  
-  **Automated Scanning**
  - Configurable schedule (default: every 6 hours)
  - Manual trigger support via Lambda invocation
  
-  **Smart Alerting**
  - Detailed email notifications via SNS
  - Highlights exact differences between Terraform and AWS
  - Shows both expected and actual values
  
-  **Audit Trail**
  - All scans logged to CloudWatch
  - Historical tracking capability

### Detected Drift Types
| Drift Type | Description | Example |
|------------|-------------|---------|
| `INSTANCE_TYPE_CHANGED` | EC2 instance size modified | `t2.micro` → `t2.small` |
| `TAG_ADDED` | New tag added via Console | Added `Environment: prod` |
| `TAG_MODIFIED` | Tag value changed | `Owner: alice` → `Owner: bob` |
| `TAG_REMOVED` | Tag deleted | Removed `CostCenter` tag |
| `SECURITY_GROUPS_CHANGED` | SG associations modified | Added/removed security groups |
| `MONITORING_CHANGED` | Detailed monitoring toggled | `false` → `true` |
| `DELETED` | Resource exists in Terraform but not AWS | Instance terminated manually |



##  Usage

### Manual Drift Scan
```bash
aws lambda invoke \
  --function-name drift-detective \
  --region us-east-1 \
  response.json

cat response.json | jq

### Create Test Drift
```bash
# Change instance type via Console
aws ec2 modify-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --instance-type t2.small

# Run detector
aws lambda invoke --function-name drift-detective response.json

### View Logs
```bash
aws logs tail /aws/lambda/drift-detective --follow
```

### Change Scan Schedule
Edit `drift-detector/terraform.tfvars`:
```hcl
# Daily at 9 AM UTC
scan_schedule = "cron(0 9 * * ? *)"

# Every 12 hours
scan_schedule = "cron(0 */12 * * ? *)"

# Monday-Friday at 6 PM UTC
scan_schedule = "cron(0 18 ? * MON-FRI *)"
```

Then apply:
```bash
terraform apply
```

##  Example Alert

**Subject:**  Infrastructure Drift Detected - 2 change(s)

```
Infrastructure Drift Detection Report
============================================================

Detected 2 drift(s) between Terraform state and actual AWS resources.

Instance: i-0abc123def456789
------------------------------------------------------------

    INSTANCE_TYPE_CHANGED
      Instance type changed from t2.micro to t2.small
      Terraform: t2.micro
      Actual:    t2.small

    TAG_ADDED
      Tag 'Environment' added with value 'production'
      Terraform: None
      Actual:    production

============================================================
Scan Time: 2025-11-01T10:30:00Z

Action Required: Review changes and update Terraform or revert manual changes.
```
## Cost Analysis

**Monthly Operating Cost: ~$0.10 - $0.50**

### Test Cases Covered
-  Terraform state parsing
-  EC2 instance comparison logic
-  Drift detection accuracy
-  SNS notification formatting
-  Error handling for missing resources

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch 
3. Commit changes 
4. Push to branch 
5. Open a Pull Request

## Author

**Maryam Suara**
- LinkedIn: www.linkedin.com/in/maryam-suara

- Email: suaramaryam@gmail.conm

** If you find this project useful, please star it!**
