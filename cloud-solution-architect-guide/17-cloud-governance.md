# Cloud Governance — Azure, AWS, Alibaba Cloud

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
Think of a big company that uses cloud services (like renting computers and storage from Microsoft or Amazon). "Cloud Governance" is basically the rulebook for how everyone in the company is allowed to use those cloud services — who can do what, how much they can spend, and how to make sure nothing gets hacked or goes wrong.

Imagine a school: the principal sets rules (governance), teachers enforce them in classrooms (policies), and students have different levels of access (some can use the computer lab, some can't). Cloud governance works the same way.

---

### Abbreviation Decoder — Every Confusing Term Explained

| Abbreviation | Full Name | Plain English |
|---|---|---|
| RBAC | Role-Based Access Control | A system where your job title decides what you're allowed to do. A "Reader" can look but not touch. An "Owner" can do everything. |
| PIM | Privileged Identity Management | A "just-in-time" key system. You don't permanently have the master key — you request it for 8 hours, use it, and it expires automatically. Like borrowing a key from a lockbox. |
| MFA | Multi-Factor Authentication | Logging in requires two proofs of identity — like your password AND a code from your phone. Even if someone steals your password, they can't get in. |
| CMK | Customer-Managed Key | Normally the cloud provider holds the encryption key to your data. With CMK, YOU hold the key. If you delete it, nobody (not even Microsoft/AWS) can read your data. |
| SCP | Service Control Policy | A hard limit set by the company's IT bosses. Even if your manager gives you permission to do something, the SCP can still block it. Like a company-wide rule that overrides your manager. |
| MG | Management Group | A folder that holds multiple Azure subscriptions. Rules applied to the folder automatically apply to everything inside it. |
| RG | Resource Group | A logical container in Azure. Think of it as a project folder — when you delete the folder, everything inside gets deleted too. |
| CSPM | Cloud Security Posture Management | A tool that constantly checks your cloud setup and gives you a "security score" — like a health checkup for your cloud. |
| CWP | Cloud Workload Protection | Security that watches your running applications for threats in real time. |
| OU | Organizational Unit | In AWS, a folder that groups multiple accounts together so you can apply the same rules to all of them at once. |
| IAM | Identity and Access Management | The system that controls who can log in and what they're allowed to do. |
| SSO | Single Sign-On | One login for everything. You log in once and can access all your company's tools without logging in again. |
| GDPR | General Data Protection Regulation | EU law that says companies must protect personal data of EU citizens. Breaking it = huge fines. |
| PCI DSS | Payment Card Industry Data Security Standard | Rules that any company handling credit card payments must follow. |
| NIST | National Institute of Standards and Technology | A US government body that publishes security best practices. |
| CIS | Center for Internet Security | An organization that publishes security benchmarks (checklists) for cloud environments. |
| DEK | Data Encryption Key | The key that actually encrypts your data. |
| KEK | Key Encryption Key | The key that encrypts the DEK. It's "keys all the way down" — this is called envelope encryption. |
| KMS | Key Management Service | AWS's service for managing encryption keys. |
| RAM | Resource Access Management | Alibaba Cloud's version of IAM — controls who can do what. |
| VNet | Virtual Network | Azure's private network in the cloud. Like your company's internal network, but in the cloud. |
| DNS | Domain Name System | The internet's phone book. Translates "google.com" into an IP address computers can use. |
| FinOps | Financial Operations | The practice of managing cloud costs — making sure you're not wasting money. |
| RBAC | Role-Based Access Control | Already explained above — your role determines your permissions. |
| AD DS | Active Directory Domain Services | Microsoft's system for managing users and computers in a company network. |
| MFA | Multi-Factor Authentication | Already explained above. |
| TDE | Transparent Data Encryption | Automatically encrypts data stored in a database without you having to do anything special. |

---

### The Big Picture — How Azure Governance Works (Like a Company Org Chart)

Imagine a company with departments, teams, and individual employees. Azure governance mirrors this:

```
Company HQ (Azure AD Tenant — the whole organization)
  └── Board of Directors (Root Management Group — top-level rules)
      ├── IT Department (Platform Management Group)
      │   ├── Security Team (Identity Subscription)
      │   ├── Operations Team (Management Subscription)
      │   └── Network Team (Connectivity Subscription)
      ├── Business Units (Landing Zones Management Group)
      │   ├── Internal Apps Team (Corp Management Group)
      │   └── Customer-Facing Apps Team (Online Management Group)
      └── Sandbox (Developers can experiment here safely)
```

Rules set at the top automatically flow down to everything below. This means you set a rule once and it applies everywhere — you don't have to repeat it for every team.

---

### How AWS Governance Works (Same Idea, Different Names)

AWS uses "Organizations" and "Accounts" instead of Management Groups and Subscriptions:

```
Company (AWS Management Account — the boss account)
  └── Security Accounts (for logs and security tools)
  └── Infrastructure Accounts (for networking)
  └── Production Accounts (real customer-facing apps)
  └── Development Accounts (for testing and building)
  └── Sandbox Accounts (for experimenting)
```

SCPs (Service Control Policies) are the rules. Even if someone in a child account has "admin" permissions, an SCP can still block them from doing certain things.

---

### How Alibaba Cloud Governance Works

Alibaba Cloud uses "Resource Directory" (like AWS Organizations) and "RAM" (like IAM). The concepts are the same — just different names.

---

### Why CMK Matters (The Key Question You Had)

Normal encryption: Microsoft/AWS holds the key → they COULD theoretically read your data (though they won't).

CMK (Customer-Managed Key): YOU hold the key in Azure Key Vault → Microsoft literally cannot read your data even if they wanted to. If you delete the key, the data is permanently unreadable.

For a financial company like Swiss Re, CMK is critical for regulatory compliance — they need to prove that only they control access to sensitive data.

---

### PIM in Plain English

Without PIM: Bob is permanently an "Owner" of the production environment. If Bob's account gets hacked, the attacker has permanent owner access.

With PIM: Bob has no standing access. When Bob needs to make a change, he:
1. Requests "Owner" access through PIM
2. Provides a reason ("deploying hotfix for incident #1234")
3. His manager approves (or it auto-approves for low-risk roles)
4. Bob has Owner access for 8 hours
5. Access automatically expires
6. Everything is logged

This is called "just-in-time" access — you get the key only when you need it.

---

> This is your gap area per Swiss Re feedback. Master this section completely.

---

## What is Cloud Governance?

Cloud governance is the set of policies, processes, and controls that ensure cloud resources are used securely, cost-effectively, and in compliance with organizational and regulatory requirements.

### The 5 Pillars of Cloud Governance
1. **Identity & Access Management** — who can do what
2. **Cost Management** — how much are we spending and why
3. **Security & Compliance** — are we meeting standards
4. **Resource Organization** — how resources are structured
5. **Operations** — how changes are managed and audited

---

# AZURE GOVERNANCE (Deep Dive)

## 1. Azure Management Hierarchy

```
Azure AD Tenant (identity boundary — one per organization)
└── Root Management Group
    ├── Platform Management Group
    │   ├── Identity Subscription
    │   │   └── Azure AD DS, AD Connect, MFA
    │   ├── Management Subscription
    │   │   └── Log Analytics, Automation, Update Mgmt
    │   └── Connectivity Subscription
    │       └── Hub VNet, ExpressRoute, DNS, Firewall
    ├── Landing Zones Management Group
    │   ├── Corp Management Group (internal apps)
    │   │   ├── Production Subscription
    │   │   ├── Non-Production Subscription
    │   │   └── Shared Services Subscription
    │   └── Online Management Group (internet-facing)
    │       └── Production Subscription
    ├── Sandbox Management Group
    │   └── Developer Sandbox Subscriptions
    └── Decommissioned Management Group
```

### Why This Structure?
- **Management Groups**: Apply policies and RBAC at scale — one policy covers all child subscriptions
- **Subscriptions**: Billing boundary, service limits, access boundary
- **Resource Groups**: Lifecycle management — delete RG = delete all resources in it
- **Resources**: Individual services

### Key Interview Point
"We use Management Groups to apply governance at scale. A policy applied at the root Management Group automatically applies to all subscriptions. This means we can enforce things like 'all resources must have a cost-center tag' or 'no public IP addresses' across the entire organization without touching each subscription individually."

---

## 2. Azure Policy

### What it does
- Enforces organizational standards
- Evaluates compliance of resources
- Can automatically remediate non-compliant resources

### Policy Effects (in order of severity)
```
Disabled → policy not evaluated
Audit → log non-compliance, don't block
AuditIfNotExists → audit if related resource missing
Append → add fields to resource
Modify → add/update tags on resources
DeployIfNotExists → deploy related resource if missing
Deny → block non-compliant resource creation
```

### Common Policy Examples
```
1. Require tags on all resources
   Effect: Deny
   "All resources must have 'environment' and 'cost-center' tags"

2. Allowed locations (data residency)
   Effect: Deny
   "Resources can only be created in West Europe and North Europe"

3. Allowed VM SKUs
   Effect: Deny
   "Only Standard_D and Standard_E series VMs allowed"

4. Require HTTPS on storage accounts
   Effect: Deny
   "Storage accounts must have 'supportsHttpsTrafficOnly: true'"

5. Deploy Log Analytics agent
   Effect: DeployIfNotExists
   "If VM exists without monitoring agent, deploy it automatically"

6. No public IP on VMs
   Effect: Deny
   "VMs cannot have public IP addresses"

7. Require private endpoints for PaaS services
   Effect: Deny
   "Azure SQL, Storage, Key Vault must use private endpoints"
```

### Policy Initiatives (Policy Sets)
Group multiple policies into one assignment:
- **Azure Security Benchmark**: 200+ policies for security best practices
- **CIS Microsoft Azure Foundations**: CIS benchmark policies
- **NIST SP 800-53**: US government compliance
- **ISO 27001**: Information security standard
- **PCI DSS**: Payment card industry

### Policy Assignment Scope
```
Root MG → applies to ALL subscriptions (use for org-wide standards)
MG → applies to child subscriptions
Subscription → applies to all RGs in subscription
Resource Group → applies to resources in RG
```

### Exemptions
- Exclude specific resources from policy evaluation
- Time-limited exemptions for migration periods
- Waiver vs Mitigated (different exemption types)

---

## 3. Azure RBAC (Role-Based Access Control)

### RBAC Model
```
Security Principal (who)
  + Role Definition (what permissions)
  + Scope (where)
= Role Assignment
```

### Built-in Roles (most important)
```
Owner: full access + manage access
Contributor: full access, cannot manage access
Reader: read-only
User Access Administrator: manage access only

Service-specific:
  AKS Cluster Admin: full cluster access
  AKS RBAC Admin: manage K8s RBAC
  Storage Blob Data Contributor: read/write blobs
  Key Vault Secrets Officer: manage secrets
  Network Contributor: manage networking
```

### Custom Roles
```json
{
  "Name": "Database Operator",
  "Description": "Can start/stop databases but not delete",
  "Actions": [
    "Microsoft.Sql/servers/databases/read",
    "Microsoft.Sql/servers/databases/write",
    "Microsoft.Sql/servers/read"
  ],
  "NotActions": [
    "Microsoft.Sql/servers/databases/delete"
  ],
  "DataActions": [],
  "AssignableScopes": [
    "/subscriptions/{subscriptionId}"
  ]
}
```

### RBAC Best Practices
1. Assign roles to groups, not individual users
2. Use built-in roles where possible
3. Least privilege: start with Reader, add only what's needed
4. Use PIM for privileged roles (just-in-time)
5. Regular access reviews (Azure AD Access Reviews)
6. Service principals for automation, Managed Identities for Azure resources

---

## 4. Azure PIM (Privileged Identity Management)

### What it does
- Just-in-time privileged access
- Time-bound access (e.g., 8 hours max)
- Approval workflow for sensitive roles
- MFA required for activation
- Audit log of all privileged access

### PIM Workflow
```
User requests activation of Owner role
  ↓
Justification required (why do you need this?)
  ↓
Approver notified (email/Teams)
  ↓
Approver approves/denies
  ↓
Role activated for X hours (max configured)
  ↓
Role automatically expires
  ↓
Audit log entry created
```

### PIM for Azure Resources vs Azure AD Roles
- Azure Resources: subscription, RG, resource-level roles
- Azure AD Roles: Global Admin, Security Admin, etc.

### Interview Answer
"We use PIM to eliminate standing privileged access. No one has permanent Owner or Contributor access. When someone needs to make a change, they activate their role through PIM, provide justification, get approval if required, and the access expires automatically. This dramatically reduces the blast radius of a compromised account."

---

## 5. Azure Conditional Access

### What it does
Controls access based on conditions:
- Who is signing in (user, group, role)
- What app they're accessing
- What device they're using
- Where they're signing in from (location, IP)
- What risk level is detected

### Common Policies
```
Policy 1: Require MFA for all users
  Users: All users
  Apps: All cloud apps
  Grant: Require MFA

Policy 2: Block legacy authentication
  Users: All users
  Client apps: Exchange ActiveSync, Other clients
  Grant: Block

Policy 3: Require compliant device for sensitive apps
  Users: All users
  Apps: Azure portal, Azure management
  Conditions: Device must be compliant (Intune-managed)
  Grant: Require compliant device

Policy 4: Block access from risky locations
  Users: All users
  Locations: Exclude trusted locations
  Risk: High sign-in risk
  Grant: Block

Policy 5: Require MFA for Azure management
  Users: All users
  Apps: Microsoft Azure Management
  Grant: Require MFA
```

---

## 6. Azure Cost Governance

### Cost Management Tools
- **Azure Cost Management + Billing**: view, analyze, optimize costs
- **Azure Advisor**: recommendations for cost, security, reliability
- **Azure Budgets**: set spending limits with alerts
- **Cost Allocation**: distribute costs to teams/projects

### Tagging Strategy
```
Required tags (enforced by Azure Policy):
  environment: production | staging | development
  cost-center: CC-1234
  team: platform | data | frontend
  project: project-name
  owner: email@company.com
  created-date: 2024-01-15

Optional tags:
  application: app-name
  criticality: high | medium | low
  data-classification: public | internal | confidential
```

### Cost Optimization Governance
```
1. Reserved Instances governance:
   - Central team purchases reservations
   - Shared scope: applies to all subscriptions
   - Review utilization monthly

2. Budget alerts:
   - 80% of budget → email alert
   - 100% of budget → email + Teams alert
   - 120% of budget → escalation

3. Idle resource cleanup:
   - Azure Advisor identifies idle VMs, unattached disks
   - Automated cleanup for dev/test environments
   - Weekly report for production

4. Right-sizing:
   - Azure Advisor recommendations
   - Review monthly
   - Approval required for production changes
```

### FinOps Maturity Model
```
Crawl: visibility — can you see your costs?
Walk: optimization — are you acting on insights?
Run: operations — is cost optimization part of your culture?
```

---

## 7. Azure Blueprints

### What it does
Package of:
- Policy assignments
- Role assignments
- ARM templates / Bicep
- Resource groups

### Use Case
"When we create a new subscription for a team, we apply a Blueprint that automatically:
- Creates required resource groups
- Assigns required policies (tagging, location, security)
- Assigns RBAC roles to the team
- Deploys baseline infrastructure (Log Analytics workspace, Key Vault)
This ensures every new environment is compliant from day one."

### Blueprint vs ARM Template vs Policy
```
ARM Template: deploys resources
Policy: enforces rules on resources
Blueprint: combines both + RBAC, locked to subscription
```

---

## 8. Azure Landing Zone

### What it is
A pre-configured, compliant Azure environment that follows best practices.

### Components
```
Management:
  - Log Analytics workspace (centralized logging)
  - Azure Monitor (alerting)
  - Update Management
  - Change Tracking

Identity:
  - Azure AD (already exists)
  - Azure AD DS (if needed for legacy apps)
  - PIM configured

Connectivity:
  - Hub VNet
  - Azure Firewall
  - ExpressRoute / VPN Gateway
  - Azure DNS Private Resolver
  - Azure Bastion

Security:
  - Microsoft Defender for Cloud (all plans)
  - Microsoft Sentinel
  - Azure Policy initiatives assigned
  - Key Vault (per subscription)

Governance:
  - Management Group hierarchy
  - Policy assignments
  - RBAC assignments
  - Tagging policy
  - Budget alerts
```

---

## 9. Azure Defender for Cloud (CSPM)

### Two Functions
1. **CSPM (Cloud Security Posture Management)**: assess and improve security posture
2. **CWP (Cloud Workload Protection)**: detect and respond to threats

### Secure Score
- Percentage of security recommendations implemented
- Higher = better security posture
- Recommendations grouped by control (e.g., "Enable MFA", "Encrypt data at rest")

### Regulatory Compliance Dashboard
- Shows compliance against: Azure Security Benchmark, CIS, PCI DSS, ISO 27001, NIST
- Drill down to specific failing controls
- Export compliance reports

---

# AWS GOVERNANCE

## 10. AWS Organizations & Control Tower

### AWS Organizations Structure
```
Management Account (billing, SCPs, no workloads)
└── Root
    ├── Security OU
    │   ├── Log Archive Account (CloudTrail, Config logs)
    │   └── Security Tooling Account (GuardDuty, Security Hub)
    ├── Infrastructure OU
    │   ├── Network Account (Transit Gateway, DNS)
    │   └── Shared Services Account (CI/CD, ECR)
    ├── Workloads OU
    │   ├── Production OU
    │   └── Non-Production OU
    └── Sandbox OU
```

### Service Control Policies (SCPs)
- Applied at OU or account level
- Restrict what IAM can do — even root user
- Deny list vs allow list approach

```json
// Deny leaving AWS Organizations
{
  "Effect": "Deny",
  "Action": "organizations:LeaveOrganization",
  "Resource": "*"
}

// Require MFA for sensitive actions
{
  "Effect": "Deny",
  "Action": ["iam:DeletePolicy", "iam:DeleteRole"],
  "Resource": "*",
  "Condition": {
    "BoolIfExists": {"aws:MultiFactorAuthPresent": "false"}
  }
}

// Restrict to specific regions
{
  "Effect": "Deny",
  "Action": "*",
  "Resource": "*",
  "Condition": {
    "StringNotEquals": {
      "aws:RequestedRegion": ["eu-west-1", "eu-central-1"]
    }
  }
}
```

### AWS Control Tower
- Automated landing zone setup
- Pre-built guardrails (preventive = SCPs, detective = Config rules)
- Account Factory: automated account provisioning via Service Catalog
- Dashboard: compliance status across all accounts

## 11. AWS Config

### What it does
- Continuous recording of resource configurations
- Compliance rules (managed + custom)
- Configuration history and timeline
- Remediation actions (manual or automatic)

### Common Config Rules
```
required-tags: all resources must have specified tags
restricted-ssh: no security groups allow SSH from 0.0.0.0/0
s3-bucket-public-read-prohibited: no public S3 buckets
encrypted-volumes: all EBS volumes must be encrypted
iam-password-policy: enforce password complexity
cloudtrail-enabled: CloudTrail must be enabled
mfa-enabled-for-iam-console-access: MFA required for console
```

## 12. AWS IAM Governance

### Permission Boundaries
- Limit maximum permissions a role can have
- Used for delegated administration
- "You can create roles, but they can't exceed these permissions"

### AWS IAM Identity Center (SSO)
- Centralized access management for multiple accounts
- Integrate with Azure AD / Okta / Active Directory
- Permission sets: define what access users get in each account
- Single sign-on across all AWS accounts

### Service Control Policies vs IAM Policies
```
SCP: organization-level guardrails (what's allowed in the account)
IAM Policy: user/role-level permissions (what this identity can do)
Effective permissions = intersection of SCP + IAM Policy
```

---

# ALIBABA CLOUD GOVERNANCE

## 13. Alibaba Cloud Governance Overview

### Resource Hierarchy
```
Alibaba Cloud Account (root)
└── Resource Directory (like AWS Organizations)
    ├── Root Folder
    │   ├── Production Folder
    │   │   └── Production Account
    │   ├── Development Folder
    │   │   └── Dev Account
    │   └── Shared Services Folder
    │       └── Shared Services Account
```

### Key Governance Services
```
Resource Management: organize accounts into folders
RAM (Resource Access Management): IAM equivalent
Control Policy: SCP equivalent (restrict account permissions)
Cloud Config: AWS Config equivalent (compliance rules)
ActionTrail: CloudTrail equivalent (API audit log)
Cost Management: billing and cost analysis
Tag Policy: enforce tagging standards
```

## 14. Alibaba RAM (Resource Access Management)

### Concepts
- **RAM Users**: individual identities (like IAM users)
- **RAM Roles**: assumable identities (like IAM roles)
- **RAM Policies**: permission documents
- **RAM Groups**: collection of users

### Key Difference from AWS/Azure
- Alibaba uses "Alibaba Cloud Account" as root (like AWS root)
- RAM users are sub-accounts under the main account
- Resource Directory enables multi-account management

### RAM Policy Example
```json
{
  "Version": "1",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["ecs:Describe*", "ecs:List*"],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "acs:RequestedRegion": "cn-hangzhou"
        }
      }
    }
  ]
}
```

## 15. Alibaba Cloud Config (Compliance)

### What it does
- Continuous compliance monitoring
- Pre-built rules for common standards
- Custom rules via Function Compute (like Lambda)
- Remediation templates

### Common Rules
```
ecs-instance-no-public-ip: ECS instances should not have public IPs
oss-bucket-public-read-prohibited: OSS buckets should not be public
rds-instance-ssl-enabled: RDS must use SSL
actiontrail-enabled: ActionTrail must be enabled
ram-user-mfa-enabled: RAM users must have MFA
```

## 16. Alibaba Cloud Security Center

### Functions
- Threat detection (like GuardDuty)
- Vulnerability scanning
- Baseline check (security configuration)
- Anti-ransomware
- Container security

### Alibaba vs AWS vs Azure Governance Comparison

| Capability | AWS | Azure | Alibaba |
|------------|-----|-------|---------|
| Multi-account | Organizations | Management Groups | Resource Directory |
| Policy enforcement | SCPs + Config | Azure Policy | Control Policy + Cloud Config |
| Identity | IAM + Identity Center | Azure AD + PIM | RAM |
| Audit log | CloudTrail | Activity Log | ActionTrail |
| Cost management | Cost Explorer | Cost Management | Cost Management |
| Security posture | Security Hub | Defender for Cloud | Security Center |
| Compliance | Config Rules | Policy + Defender | Cloud Config |
| Tagging | Tag Policies | Azure Policy | Tag Policy |

---

## 17. Governance Interview Questions & Answers

**Q: How do you enforce that all Azure resources have required tags?**
A: "We use Azure Policy with a Deny effect at the Management Group level. The policy checks for required tags (environment, cost-center, team) on resource creation. If tags are missing, the deployment fails. We also use a Modify policy to automatically add default tags where possible. This is assigned at the root Management Group so it applies to all subscriptions."

**Q: How do you prevent developers from creating public-facing resources in production?**
A: "Multiple layers: First, Azure Policy with Deny effect prevents creation of public IPs and public storage accounts in production subscriptions. Second, RBAC ensures developers only have Contributor access in dev/test, not production. Third, all production changes go through a CI/CD pipeline with approval gates. Fourth, Defender for Cloud alerts on any public-facing resources that slip through."

**Q: How do you manage costs across multiple teams?**
A: "We use a combination of: mandatory cost-center tags enforced by Policy, Azure Budgets with alerts at 80% and 100%, subscription-per-team for clear billing boundaries, Azure Cost Management for showback reports to each team, and monthly FinOps reviews. We also use Azure Advisor recommendations for right-sizing and idle resource cleanup."

**Q: How do you handle privileged access in Azure?**
A: "No standing privileged access. We use PIM for all Owner and Contributor roles. When someone needs elevated access, they activate through PIM with justification, get approval from their manager, and the access expires after 8 hours maximum. All activations are logged. We also have Conditional Access requiring MFA for all Azure management operations."

**Q: What is a Landing Zone and why does it matter?**
A: "A Landing Zone is a pre-configured, compliant Azure environment. Before any team gets a subscription, we apply our Landing Zone blueprint which sets up: Management Group placement, required Policy assignments, RBAC roles, baseline infrastructure (Log Analytics, Key Vault, networking), and budget alerts. This means every new environment is compliant from day one, not retrofitted later."

**Q: How do you ensure compliance with GDPR/financial regulations in Azure?**
A: "We use Azure Policy to enforce data residency (resources only in EU regions), Defender for Cloud with regulatory compliance dashboard showing GDPR/ISO 27001 status, Azure Purview for data classification and lineage, encryption at rest with customer-managed keys in Key Vault, and regular compliance reports exported from Defender for Cloud. We also use Private Endpoints to ensure PII data never traverses the public internet."
