# drift-detective

> Automatically detects when AWS resources drift from their Terraform-defined state and sends alerts before configuration mismatches cause production issues.

## Summary

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

##  Usage

### Manual Drift Scan

aws lambda invoke \
  --function-name drift-detective \
  --region us-east-1 \
  response.json

cat response.json | jq

### Create Test Drift

# Change instance type via Console
aws ec2 modify-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --instance-type t2.small

# Run detector
aws lambda invoke --function-name drift-detective response.json

![IMG_0331](https://github.com/user-attachments/assets/10ce1239-c450-4d0b-a564-c50a4bd9d2aa)

### View Logs

aws logs tail /aws/lambda/drift-detective --follow


![IMG_0333](https://github.com/user-attachments/assets/01493cd6-c54f-4e4a-a617-9268c9805c1a)

### Change Scan Schedule
Edit `drift-detector/terraform.tfvars`:

# Daily at 9 AM UTC
scan_schedule = "cron(0 9 * * ? *)"

# Every 12 hours
scan_schedule = "cron(0 */12 * * ? *)"

# Monday-Friday at 6 PM UTC
scan_schedule = "cron(0 18 ? * MON-FRI *)"


Then apply:

terraform apply


##  Example Alert

<img width="828" height="1792" alt="IMG_2981" src="https://github.com/user-attachments/assets/25b0f636-7458-4652-a120-7720785a7177" />


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
