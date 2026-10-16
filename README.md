# StartupCo AWS Identity & Security Hardening🔒

**Status:** Complete  
**Environment:** Deployment validated | Lab environment  
**Focus:** AWS IAM Identity Center · Terraform · Cloud Security · Least Privilege · Compliance Monitoring

Designed and implemented an AWS identity and security-hardening solution for a simulated 10-person startup. The project establishes centralized workforce access, role-based permissions, secure EC2 administration, and continuous security monitoring using AWS-native services and Terraform.

**📖 [Full Engineering Case Study on Medium]([PASTE_YOUR_MEDIUM_ARTICLE_URL_HERE](https://medium.com/@ivorycloudops/hardening-aws-access-for-a-10-person-startup-with-iam-identity-center-and-terraform-%EF%B8%8F-54ef035be24c?sharedUserId=ivorycloudops))** — Architecture decisions, implementation details, permission testing, security incident simulation, and remediation evidence.

---

## 1. Business Problem🏢

StartupCo needed to replace shared administrative access with a more secure, accountable identity management solution while maintaining appropriate access for different job functions.

The solution addresses:

- Excessive permissions and shared credentials.
- Lack of separation between development, operations, finance, and analytics responsibilities.
- Insecure administrative access to EC2 instances.
- Limited visibility into security events and configuration changes.

## 2. Architecture🏗️

The solution combines centralized identity management, AWS security services, and Infrastructure as Code.

**Core components:**

- **IAM Identity Center:** Centralized SSO and group-based permission sets for Developers, Operations, Finance, and Analysts.
- **Amazon EC2 & Systems Manager:** Secure instance administration using Session Manager without inbound SSH.
- **Amazon S3 & AWS KMS:** Protected application storage and customer-managed encryption.
- **CloudTrail & AWS Config:** API activity auditing, configuration monitoring, and compliance evaluation.
- **GuardDuty & Security Hub CSPM:** Threat detection and centralized security posture visibility.
- **EventBridge & SNS:** Event-driven notifications for compliance changes.
- **Terraform:** Repeatable infrastructure provisioning, security configuration, and drift remediation.

**Architecture Diagram**

![StartupCo AWS Identity & Security Hardening Architecture](Doc/Architecture/startupco-security-architecture.png)


## 3. Key Decisions & Why🤔

| Decision | Engineering rationale |
|---|---|
| IAM Identity Center | Centralized workforce authentication and access management instead of shared IAM credentials. |
| Group-based permission sets | Aligned access with job responsibilities while separating infrastructure, financial, and application permissions. |
| Systems Manager Session Manager | Enabled auditable EC2 administration without exposing SSH port 22. |
| AWS Config + EventBridge | Enabled detection and notification of security configuration changes. |
| Terraform-managed infrastructure | Established reproducible configuration and supported recovery from unauthorized configuration drift. |

**Security Validation**

Tested permissions across all four workforce groups using permitted and denied AWS actions.

Also simulated an S3 security misconfiguration and verified the detection-to-recovery workflow:

1. AWS Config identified the noncompliant S3 configuration.
2. EventBridge recorded the compliance event.
3. CloudTrail provided the corresponding API activity record.
4. Terraform identified the configuration drift and proposed corrective changes.
5. AWS Config returned to compliant status, and a final Terraform plan reported no changes.

**[View permission testing, screenshots, and remediation evidence on Medium](PASTE_YOUR_MEDIUM_ARTICLE_URL_HERE)**

## 4. What I'd Do Differently at Production Scale📈

- **Multi-account architecture:** Separate production, development, logging, and security workloads using AWS Organizations.
- **Stronger permission boundaries:** Further restrict broad operational permissions using resource-level conditions, service control policies, and periodic access reviews.
- **Automated remediation:** Integrate controlled remediation workflows with approval gates and rollback safeguards.
- **CI/CD integration:** Manage Terraform changes through version-controlled pipelines with security scanning and approval requirements.
- **Centralized security:** Aggregate organization-wide audit logs, GuardDuty findings, and Security Hub results into dedicated security accounts.

---

**Source Code:** This repository contains the Terraform configurations used to implement the infrastructure and security controls.

**Detailed Case Study:** [StartupCo AWS Identity & Security Hardening — Medium]([PASTE_YOUR_MEDIUM_ARTICLE_URL_HERE](https://medium.com/@ivorycloudops/hardening-aws-access-for-a-10-person-startup-with-iam-identity-center-and-terraform-%EF%B8%8F-54ef035be24c?sharedUserId=ivorycloudops))

























## 🏗️ Architecture


```text
AWS IAM Identity Center
        |
        +-- Developers  (4 users) → DeveloperPermission
        +-- Operations  (2 users) → OperationPermission
        +-- Finance     (1 user)  → FinancePermission
        +-- Analysts    (3 users) → AnalystPermission
                |
                v
     Permission sets assigned to the AWS workload account
```

---

## 🧠 Key Decisions & Why

### Identity Architecture

- **IAM Identity Center instead of 10 traditional IAM users.** Eliminates long-lived console credentials entirely and centralizes authentication and MFA at the identity-store level instead of per-user.
- **Groups + permission sets instead of per-user policies.** Ten individual policies would drift out of sync over time; a group-membership change is a one-line Terraform diff instead of a new policy to write and review.

### Least-Privilege Design, by Group

- **Developers — tag-scoped EC2, not account-wide.** Start/stop/reboot is restricted to instances tagged `Role = app-server`, so a developer credential can't reach infrastructure outside the app tier even where EC2 access is otherwise broad.
- **Operations — full access is the correct call here.** Operations owns EC2/CloudWatch/RDS infrastructure by job function, so AWS-managed full-access policies match the actual business requirement instead of being over-provisioned.
- **Finance — visibility without control.** Cost Explorer/Budgets plus read-only `Describe`/`List`/`Get` verbs only; validated that EC2 start, S3 upload, and Session Manager are all explicitly denied. Financial visibility and infrastructure administration are different jobs and shouldn't share a credential.
- **Analysts — read-only, and a real IAM boundary.** S3 read access is straightforward; RDS is the interesting case. `rds:Describe*` at the IAM level is not the same thing as a SQL `SELECT` at the database level — IAM controls whether you can reach the database resource at all, not what you can query once connected. That boundary had to be documented explicitly, not assumed.

### EC2 Administration

- **Systems Manager Session Manager, no SSH.** No key pair, no port 22, no inbound security-group rule for management at all. Operations gets a real interactive shell through Identity Center → Systems Manager → Session Manager instead.

### Security Monitoring

- **CloudTrail as the audit layer, GuardDuty as the detective layer, kept deliberately separate.** CloudTrail answers "who did what, when"; GuardDuty answers "is this activity suspicious." Config, Security Hub, EventBridge, and SNS then chain those into a single detect → alert → investigate → remediate flow, tested against one controlled, non-destructive misconfiguration.

---

## 🚀 What I'd Do Differently at Production Scale

- **AWS Organizations + SCPs**, with separate accounts per environment or per function — intentionally out of scope here to keep the project focused on identity and security fundamentals rather than multi-account governance.
- **Automated Config remediation** (e.g., Lambda-backed auto-remediation) instead of a human investigating every finding manually.
- **A delegated Security Hub admin account**, rather than findings living in the same account as the workload.
- **CI/CD-driven permission-set changes** with peer review, instead of applying Terraform directly — access changes are exactly the kind of thing that should require a second set of eyes.
- **Macie** for data classification, if StartupCo's S3 buckets ever held real PII/PHI-type fitness data at scale — deliberately excluded here since the test bucket only holds a sample object.
- **A documented break-glass root procedure** with automatic alerting on any root login, since root should be removed from routine use but still needs a defined emergency path.
