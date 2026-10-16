# StartupCo AWS Identity & Security Hardening🔒

**Status:** Complete  
**Environment:** Deployment validated | Lab environment  
**Focus:** AWS IAM Identity Center · Terraform · Cloud Security · Least Privilege · Compliance Monitoring

Designed and implemented an AWS identity and security-hardening solution for a simulated 10-person startup. The project establishes centralized workforce access, role-based permissions, secure EC2 administration, and continuous security monitoring using AWS-native services and Terraform.

**📖 [Full Engineering Case Study on Medium] Architecture decisions, implementation details, permission testing, security incident simulation, and remediation evidence.

(https://medium.com/@ivorycloudops/hardening-aws-access-for-a-10-person-startup-with-iam-identity-center-and-terraform-%EF%B8%8F-54ef035be24c?sharedUserId=ivorycloudops)**

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

**[View permission testing, screenshots, and remediation evidence on Medium]

(https://medium.com/@ivorycloudops/hardening-aws-access-for-a-10-person-startup-with-iam-identity-center-and-terraform-%EF%B8%8F-54ef035be24c?sharedUserId=ivorycloudops)**

## 4. What I'd Do Differently at Production Scale📈

- **Multi-account architecture:** Separate production, development, logging, and security workloads using AWS Organizations.
- **Stronger permission boundaries:** Further restrict broad operational permissions using resource-level conditions, service control policies, and periodic access reviews.
- **Automated remediation:** Integrate controlled remediation workflows with approval gates and rollback safeguards.
- **CI/CD integration:** Manage Terraform changes through version-controlled pipelines with security scanning and approval requirements.
- **Centralized security:** Aggregate organization-wide audit logs, GuardDuty findings, and Security Hub results into dedicated security accounts.

---

**Source Code:** This repository contains the Terraform configurations used to implement the infrastructure and security controls.

**Detailed Case Study:** [StartupCo AWS Identity & Security Hardening — Medium]

(https://medium.com/@ivorycloudops/hardening-aws-access-for-a-10-person-startup-with-iam-identity-center-and-terraform-%EF%B8%8F-54ef035be24c?sharedUserId=ivorycloudops)
