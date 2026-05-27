# ☁️ AWS Cloud IAM Security Assessment & Remediation Lab

![AWS](https://img.shields.io/badge/AWS-Cloud%20Security-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Prowler](https://img.shields.io/badge/Prowler-Security%20Scanner-blue?style=for-the-badge)
![NIST](https://img.shields.io/badge/NIST-CSF%20Aligned-green?style=for-the-badge)
![CIS](https://img.shields.io/badge/CIS-AWS%20Foundations-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

---

## 📌 Overview

This project simulates real-world AWS IAM and cloud security misconfigurations found in enterprise environments. It demonstrates the **full security lifecycle** — from building a vulnerable environment and exploiting it, to detecting, remediating, and validating the fixes using industry-standard tools and frameworks.

**Why this project matters:**
Cloud IAM misconfigurations are consistently among the top causes of cloud breaches. This lab was built to develop hands-on skills in identifying, exploiting, detecting, and hardening these vulnerabilities — mirroring the workflow of a real cloud security engineer or SOC analyst.

**Full Security Lifecycle Covered:**
```
Vulnerability Creation → Exploitation → Detection & Monitoring → Remediation → Validation
```

---

## 🎯 Objectives

- Simulate insecure AWS IAM configurations in a controlled lab environment
- Demonstrate privilege escalation via `AssumeRole` abuse
- Exploit publicly exposed S3 storage containing sensitive credentials
- Detect misconfigurations using **Prowler** security scanner
- Implement centralized logging with **AWS CloudTrail**
- Configure **CloudWatch** detection rules and **SNS** alerting
- Harden the environment using least privilege principles
- Validate remediation through rescanning and before/after comparison

---

## 🏗️ Architecture Diagram

> See `/architecture/` folder for full diagram.

The lab environment consists of:
- **IAM Users & Roles** — intentionally misconfigured with over-permissive policies
- **S3 Bucket** — publicly exposed with simulated sensitive credentials
- **CloudTrail** — multi-region trail capturing all API activity
- **CloudWatch** — metric filters and alarms for suspicious event detection
- **SNS** — automated email alerting on triggered alarms
- **Prowler** — external security scanner validating posture before and after remediation

---

## 🚨 Vulnerabilities Identified

| # | Vulnerability | Severity | Status |
|---|---|---|---|
| 1 | Over-Permissive IAM Policy (`*:*`) | 🔴 Critical | ✅ Remediated |
| 2 | Privilege Escalation via AssumeRole | 🔴 Critical | ✅ Remediated |
| 3 | Public S3 Bucket Exposure | 🟠 High | ✅ Remediated |
| 4 | Exposed Sensitive Credentials in S3 | 🟠 High | ✅ Remediated |
| 5 | Missing CloudTrail Logging | 🟠 High | ✅ Remediated |
| 6 | MFA Disabled for IAM User | 🟡 Medium | ✅ Remediated |

---

## ⚔️ Exploitation Phase

### 1. Public S3 Bucket Exposure

An S3 bucket was intentionally configured with public access enabled. A file containing simulated credentials was uploaded and made publicly accessible.

**Attack Path:**
```
Attacker discovers public bucket → Downloads credentials file → Uses credentials for unauthorized AWS access
```

**Impact:**
- Access to sensitive files and data
- Enumeration of cloud resources
- Use of exposed credentials for further account compromise

**Evidence:** Public bucket access confirmed via browser; credentials file accessible externally without authentication.

---

### 2. IAM Privilege Escalation via AssumeRole

An overly permissive IAM policy allowed `security-lab-user` to assume privileged IAM roles, including an admin role.

**Exploit Command:**
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::<ACCOUNT_ID>:role/vulnerable-admin-role \
  --role-session-name privilege-escalation-test
```

**Result:** Temporary administrator credentials successfully generated — full privilege escalation achieved.

**Attack Path:**
```
Low-privilege user → AssumeRole abuse → Temporary admin credentials → Full environment control
```

**Impact:**
- Full administrative access to the AWS environment
- Ability to create/delete resources, modify policies, exfiltrate data
- Lateral movement across the AWS environment

---

## 🔍 Detection & Monitoring

### Prowler Security Assessment

Prowler was used to scan the AWS environment for IAM misconfigurations, public storage exposure, missing MFA, logging gaps, and security best practice violations.

**Initial Scan Results:**

| Metric | Value |
|---|---|
| Total Findings | 342 |
| ❌ Failed Findings | 216 |
| ✅ Passed Findings | 126 |

> See `/findings/prowler-initial-report/` for full output.

---

### AWS CloudTrail Logging

CloudTrail was configured as a **multi-region trail** to capture:

| Event Type | Purpose |
|---|---|
| IAM Activity | Track policy changes and user modifications |
| Console Logins | Detect unauthorized access attempts |
| API Calls | Full audit trail of AWS actions |
| Role Assumptions | Detect `AssumeRole` abuse |
| Security-Relevant Events | Identify suspicious patterns |

**Features Enabled:** Multi-region logging · Log file validation · S3 log storage · CloudWatch integration

---

### CloudWatch Detection Engineering

CloudWatch metric filters and alarms were created to detect suspicious activity in real time.

| Detection Rule | Purpose |
|---|---|
| `Root-Account-Login-Alert` | Detect root account console logins |
| `Console-Login-Without-MFA-Alert` | Detect insecure console access without MFA |

---

### SNS Alerting

AWS SNS was integrated with CloudWatch alarms to deliver **automated email notifications** for all triggered security events — simulating a real SOC alerting pipeline.

---

## 🛡️ Remediation & Hardening

### 1. Public S3 Exposure — Remediated

**Actions Taken:**
- Removed public bucket policy
- Re-enabled Block Public Access settings
- Restricted all object-level access

**Validation:**
```xml
<Code>AccessDenied</Code>
<Message>Access Denied</Message>
```
✅ Credentials file no longer publicly accessible.

---

### 2. IAM Hardening — Remediated

**Actions Taken:**
- Removed `AdministratorAccess` policy from user
- Deleted `AssumeAnyRolePolicy`
- Enforced least privilege — scoped permissions to required actions only

**Validation:**
```
An error occurred (AccessDenied) when calling the AssumeRole operation
```
✅ Privilege escalation path fully blocked.

---

### 3. MFA Enforcement — Remediated

MFA was enabled for the IAM user to reduce the risk of credential compromise and unauthorized console access.

---

## 📉 Before vs. After Comparison

| Security Metric | ❌ Before Remediation | ✅ After Remediation |
|---|---|---|
| Failed Prowler Findings | 216 | 1 |
| Public S3 Access | Enabled | Disabled |
| Privilege Escalation | Successful | Blocked |
| MFA Enforcement | Disabled | Enabled |
| CloudTrail Logging | Disabled | Enabled |
| CloudWatch Monitoring | Not Configured | Configured |
| SNS Alerting | Not Configured | Configured |

**Final Prowler Scan Results:**

| Metric | Value |
|---|---|
| Total Findings | 1 |
| ❌ Failed Findings | 1* |

> *Remaining finding related to AWS Firewall Manager — outside scope of this IAM-focused lab.

---

## 🧩 Framework Mapping (NIST / CIS)

| Security Control | CIS AWS Foundations | NIST CSF Function |
|---|---|---|
| MFA Enforcement | CIS 1.10 | Protect (PR.AC) |
| Least Privilege IAM | CIS 1.16 | Protect (PR.AC) |
| Block Public S3 Access | CIS 2.1 | Protect (PR.DS) |
| CloudTrail Logging | CIS 3.1 | Detect (DE.CM) |
| CloudWatch Monitoring | CIS 3.x | Detect (DE.CM) |
| SNS Alerting | — | Respond (RS.CO) |
| IAM Hardening | CIS 1.x | Protect (PR.AC) |
| Detection Engineering | — | Detect (DE.AE) |

---

## 📚 Key Learnings

- **Small IAM misconfigurations can lead to full cloud compromise** — a single overly permissive policy enabled complete privilege escalation
- **Least privilege is non-negotiable** — wildcards (`*:*`) in IAM policies create critical attack surfaces
- **Public cloud storage remains a top risk** — S3 misconfigurations are consistently among the most common real-world cloud breaches
- **Centralized logging dramatically improves detection** — CloudTrail + CloudWatch enabled full visibility into attacker activity
- **CloudWatch metric filters are powerful detection tools** — custom rules can detect specific attack patterns in near real-time
- **Validation is essential** — rescanning with Prowler after remediation confirmed the fixes and showed measurable improvement
- **Cloud security requires both offensive and defensive thinking** — understanding how attackers exploit misconfigs is essential for effective defense

---

## 📁 Repository Structure

```
cloud-iam-security-assessment-lab/
│
├── README.md
├── architecture/
│   └── architecture-diagram.png
├── screenshots/
│   ├── public-s3-exposure/
│   ├── privilege-escalation/
│   ├── prowler-findings/
│   ├── cloudtrail-config/
│   ├── cloudwatch-alarms/
│   ├── sns-alerts/
│   └── remediation-validation/
├── findings/
│   ├── prowler-initial-report/
│   └── prowler-final-report/
├── policies/
│   ├── vulnerable-policies/
│   └── hardened-policies/
├── logs/
│   └── cloudtrail-samples/
```

---

## 🛠️ Technologies & Services Used

**AWS Services**

![IAM](https://img.shields.io/badge/AWS-IAM-FF9900?style=flat&logo=amazonaws)
![S3](https://img.shields.io/badge/AWS-S3-FF9900?style=flat&logo=amazonaws)
![CloudTrail](https://img.shields.io/badge/AWS-CloudTrail-FF9900?style=flat&logo=amazonaws)
![CloudWatch](https://img.shields.io/badge/AWS-CloudWatch-FF9900?style=flat&logo=amazonaws)
![SNS](https://img.shields.io/badge/AWS-SNS-FF9900?style=flat&logo=amazonaws)

**Security Tools**

![Prowler](https://img.shields.io/badge/Prowler-Scanner-blue?style=flat)
![AWS CLI](https://img.shields.io/badge/AWS-CLI-232F3E?style=flat&logo=amazonaws)

**Frameworks**

![NIST](https://img.shields.io/badge/NIST-CSF-green?style=flat)
![CIS](https://img.shields.io/badge/CIS-AWS%20Foundations%20Benchmark-red?style=flat)
![SOC2](https://img.shields.io/badge/SOC%202-Aligned-purple?style=flat)

---

## 🔗 References

- [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
- [Prowler Documentation](https://docs.prowler.cloud/)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [AWS Security Best Practices](https://aws.amazon.com/architecture/security-identity-compliance/)

---


---

> 💡 *This lab was built in a controlled AWS environment for educational purposes. All misconfigurations were intentionally created and fully remediated.*
