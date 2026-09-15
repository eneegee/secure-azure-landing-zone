# Governance Design — CloudNova Technologies

## 1. Purpose

This document defines the governance model for the CloudNova Secure Azure Landing Zone.

The goal is to establish a consistent Azure foundation that controls how resources are organized, named, tagged, accessed, protected, monitored, and governed.

The design is intentionally adapted to CloudNova's current project scale.

The implementation uses a single Azure Pay-As-You-Go subscription and does not attempt to reproduce the complete Microsoft enterprise-scale Azure Landing Zone architecture.

Instead, it demonstrates the governance principles that would form the foundation of a larger Azure environment.

---

## 2. Governance Principles

The landing zone will follow the following principles:

### Least Privilege

Users and administrative groups should receive only the permissions required for their responsibilities.

### Scoped Administration

Permissions should be assigned at the narrowest practical Azure scope rather than automatically granting subscription-wide access.

### Policy-Based Governance

Azure Policy should be used to identify or prevent resources that do not meet defined governance requirements.

### Consistent Resource Organization

Resources should follow predictable naming, tagging, and resource-group standards.

### Separation of Responsibilities

Platform, network, workload, and audit responsibilities should be logically separated.

### Secure-by-Default Foundation

Future workloads should inherit or operate within governance controls rather than relying on each application team to implement security independently.

### Auditability

Administrative and governance-related activity should be attributable to identifiable identities.

### Cost Awareness

Resource ownership and workload context should be visible enough to support cost analysis and accountability.

---

## 3. Azure Resource Hierarchy

The project uses the following logical hierarchy:

```text
Microsoft Entra ID
        │
        ▼
Azure Pay-As-You-Go Subscription
        │
        ├── rg-cloudnova-platform-lab
        │
        ├── rg-cloudnova-network-lab
        │
        └── rg-cloudnova-workload-lab
```

### Subscription

The existing Azure Pay-As-You-Go subscription provides the Azure boundary for the project.

The subscription also contains other personal learning resources, so CloudNova governance controls should not unnecessarily affect unrelated projects.

For this reason, project-specific controls will be scoped primarily to CloudNova resource groups and resources rather than being applied broadly across the entire subscription.

---

## 4. Resource Group Design

CloudNova resources will be separated according to operational responsibility.

### `rg-cloudnova-platform-lab`

Purpose:

Provides a logical location for shared or platform-level resources required by the landing-zone implementation.

Examples may include:

- Shared governance-supporting resources.
- Platform services.
- Future shared operational components.

---

### `rg-cloudnova-network-lab`

Purpose:

Contains the networking foundation for the landing zone.

Expected resources include:

- Virtual Network.
- Subnets.
- Network Security Groups.
- Network-related supporting resources.

Keeping networking resources separate allows network permissions and protection controls to be scoped independently from application workloads.

---

### `rg-cloudnova-workload-lab`

Purpose:

Contains demonstration application or workload resources deployed into the governed landing zone.

This allows workload teams to receive permissions over their own resources without automatically receiving control over shared networking or platform resources.

---

## 5. Resource Naming Standard

Azure resources should follow a predictable naming convention.

The general naming pattern is:

```text
<resource-type>-<workload>-<environment>-<region>-<instance>
```

Example:

```text
vnet-cloudnova-lab-<region>-01
```

### Resource Type Abbreviations

The project will use recognizable Azure resource abbreviations where practical.

| Azure Resource | Abbreviation |
|---|---|
| Resource Group | `rg` |
| Virtual Network | `vnet` |
| Subnet | `snet` |
| Network Security Group | `nsg` |
| Virtual Machine | `vm` |
| Network Interface | `nic` |
| Public IP | `pip` |
| Storage Account | `st` |
| Log Analytics Workspace | `law` |

Not every resource will necessarily be deployed during the project.

---

## 6. Naming Examples

Examples of compliant names include:

```text
rg-cloudnova-network-lab
rg-cloudnova-workload-lab

vnet-cloudnova-lab-<region>-01

snet-cloudnova-web-lab-01
snet-cloudnova-app-lab-01

nsg-cloudnova-web-lab-01
nsg-cloudnova-app-lab-01
```

Some Azure services have naming restrictions.

For example, storage-account names cannot contain hyphens and must be globally unique.

A storage account may therefore use a format such as:

```text
stcloudnovalab<unique>
```

The naming standard should remain understandable while respecting individual Azure service naming restrictions.

---

## 7. Environment Classification

The current implementation represents a:

```text
Lab
```

environment.

The naming and tagging structure is designed so that future environments could use values such as:

```text
Dev
Test
Prod
```

without requiring the governance model to be redesigned.

The project must not represent the current lab environment as a production environment.

---

## 8. Tagging Strategy

Tags provide business and operational context that resource names alone cannot provide.

CloudNova will define four primary governance tags.

### Required Tags

| Tag | Purpose | Example |
|---|---|---|
| `Environment` | Identifies deployment environment | `Lab` |
| `Owner` | Identifies responsible team | `CloudNova-Platform` |
| `Workload` | Identifies application or service | `LandingZone` |
| `CostCenter` | Provides cost/accountability context | `CloudNova-IT` |

Example:

```text
Environment = Lab
Owner = CloudNova-Platform
Workload = LandingZone
CostCenter = CloudNova-IT
```

---

## 9. Optional Governance Tags

Additional tags may be used where they provide useful context.

Examples include:

```text
Criticality
DataClassification
ManagedBy
```

These are not mandatory for the initial implementation because unnecessary tagging can create administrative overhead.

The design prioritizes a small set of useful, consistently applied metadata.

---

## 10. Tagging by Resource Group

Resource groups will carry governance metadata appropriate to their purpose.

### Platform

```text
Environment = Lab
Owner = CloudNova-Platform
Workload = SharedPlatform
CostCenter = CloudNova-IT
```

### Network

```text
Environment = Lab
Owner = CloudNova-Network
Workload = NetworkFoundation
CostCenter = CloudNova-IT
```

### Workload

```text
Environment = Lab
Owner = CloudNova-AppTeam
Workload = DemoApplication
CostCenter = CloudNova-Engineering
```

Resource-level tags should either be explicitly applied or inherited through Azure Policy where appropriate.

---

## 11. RBAC Design

Azure RBAC will be used to separate administrative responsibilities.

The design avoids granting all project participants broad subscription-level access.

The proposed role model is:

```text
CloudNova Platform Administration
        │
        ├── Platform resources
        ├── Network resources where required
        └── Workload governance

CloudNova Network Administration
        │
        └── Network resources only

CloudNova Application Team
        │
        └── Workload resources only

CloudNova Audit / Read-Only Access
        │
        └── Visibility without modification
```

---

## 12. Proposed Security Groups

The project may use Microsoft Entra security groups such as:

```text
CloudNova-Platform-Admins
CloudNova-Network-Admins
CloudNova-App-Team
CloudNova-Auditors
```

Permissions should preferably be assigned to groups rather than directly to individual users.

This simplifies access management and allows users to be added or removed without recreating role assignments.

---

## 13. Platform Administrator Access

### Group

```text
CloudNova-Platform-Admins
```

### Intended Role

```text
Contributor
```

### Scope

CloudNova project resource groups only.

### Purpose

Allows designated platform administrators to manage project resources without automatically receiving subscription-wide Owner permissions.

Role-assignment administration remains controlled by the environment owner.

---

## 14. Network Administrator Access

### Group

```text
CloudNova-Network-Admins
```

### Intended Role

```text
Network Contributor
```

### Scope

```text
rg-cloudnova-network-lab
```

### Purpose

Allows network administrators to manage:

- Virtual Networks.
- Subnets.
- Network Security Groups.
- Other supported networking resources.

The group should not automatically receive permissions over application resources.

---

## 15. Application Team Access

### Group

```text
CloudNova-App-Team
```

### Intended Role

```text
Contributor
```

### Scope

```text
rg-cloudnova-workload-lab
```

### Purpose

Allows the application team to deploy and manage workload resources while preventing management of the landing-zone networking foundation.

This demonstrates delegated administration.

---

## 16. Audit Access

### Group

```text
CloudNova-Auditors
```

### Intended Role

```text
Reader
```

### Scope

CloudNova project resources.

### Purpose

Provides visibility into Azure resources and configuration without granting modification permissions.

---

## 17. Environment Owner

The personal Azure account used to manage the learning subscription remains the subscription owner.

This account represents the environment/platform owner responsible for:

- Initial project bootstrap.
- Azure Policy assignments.
- RBAC administration.
- Emergency configuration changes.
- Project cleanup.

The project should not use the subscription Owner role as the normal authorization model for CloudNova workload users.

---

## 18. Azure Policy Strategy

Azure Policy will provide governance controls that operate independently from individual workload teams.

The initial policy baseline will focus on a small number of high-value controls rather than deploying a very large policy catalogue.

The planned controls include:

### Required Governance Tags

Azure Policy will evaluate or enforce the required CloudNova tags.

Priority tags include:

```text
Environment
Owner
CostCenter
```

Where practical, resource-level tags may inherit values from their parent resource group.

---

### Allowed Azure Regions

CloudNova resources should be deployed only into approved Azure regions.

An Allowed Locations policy will be used to prevent accidental deployment into unapproved locations.

The final approved region or regions will be selected during architecture and implementation planning based on service availability, latency, and project requirements.

---

### Storage Security Baseline

Where Azure Storage is used, governance should require secure configuration.

Examples include:

- Secure transfer requirements.
- Review of public network exposure.
- Microsoft Entra-based access where practical.

The exact policy effects will be selected during implementation to avoid introducing unnecessary disruption.

---

## 19. Policy Effects

Different Azure Policy effects will be used according to the risk and maturity of the control.

### Deny

Used when CloudNova has a clear requirement that should not be bypassed during normal resource deployment.

Potential examples:

- Missing required governance metadata.
- Deployment into an unapproved region.

### Audit

Used where visibility should be established before automatic enforcement.

Potential examples:

- Security configurations where immediate denial could unintentionally disrupt workloads.

### Modify

May be used where Azure Policy can safely add or inherit required governance metadata.

An example is inheriting specific tags from a resource group when they are missing from a resource.

---

## 20. Policy Enforcement Philosophy

CloudNova will not use `Deny` for every possible security configuration.

The project follows a progressive governance approach:

```text
Identify requirement
        ↓
Audit configuration
        ↓
Understand impact
        ↓
Remediate
        ↓
Enforce where appropriate
```

Controls that are well understood and low risk to enforce can use `Deny`.

Controls that could affect application availability should initially use `Audit` until their impact is understood.

---

## 21. Resource Protection

Azure resource locks will protect selected foundational resources against accidental deletion.

The preferred lock type for this project is:

```text
CanNotDelete
```

This allows authorized configuration changes while preventing accidental deletion.

A lock may be applied to an important landing-zone resource such as the foundational Virtual Network.

The project will not apply unnecessary locks to every resource because excessive locking can complicate legitimate administration and cleanup.

---

## 22. Cost Governance

Cost governance will use both technical and organizational controls.

### Tagging

The `CostCenter`, `Owner`, `Environment`, and `Workload` tags provide context for cost analysis.

### Resource Separation

Dedicated CloudNova resource groups make it easier to identify project-related resources.

### Cost Analysis

Azure Cost Management can be used to review costs by:

- Resource group.
- Resource type.
- Service.
- Tag.

### Resource Lifecycle

Temporary test resources should be stopped or deleted when no longer required.

Continuously running compute should be avoided unless needed for a specific validation scenario.

---

## 23. Logging and Auditability

The governance design uses Azure-native logging capabilities.

### Azure Activity Log

Azure Activity Log provides visibility into management-plane operations such as:

- Resource creation.
- Resource deletion.
- Configuration changes.
- RBAC operations.
- Policy-related management activity.

The project will validate that controlled management actions can be attributed to the identity that performed them.

### Azure Policy Compliance

Azure Policy compliance results will provide visibility into governed and non-compliant resources.

A full SIEM implementation is outside the scope of this project.

---

## 24. Governance Exceptions

Real environments occasionally require exceptions to governance rules.

An exception should not be created simply because a policy is inconvenient.

A valid exception should identify:

- The policy or control affected.
- The resource or workload requiring the exception.
- Business or technical justification.
- Security impact.
- Compensating controls where required.
- Whether the exception is temporary or permanent.

The current project does not require a formal enterprise exception-management platform.

The principle is documented to demonstrate how governance decisions should be handled.

---

## 25. Governance Validation Scenarios

The governance implementation will be tested through controlled scenarios.

### Test 1 — Missing Required Tag

Attempt to deploy a governed resource without required metadata.

**Expected result:**  
The resource is identified as non-compliant or deployment is denied according to the policy effect.

---

### Test 2 — Compliant Deployment

Deploy a resource using the required governance metadata.

**Expected result:**  
Deployment succeeds and the resource is compliant.

---

### Test 3 — Unapproved Region

Attempt to deploy a governed resource into a location outside the approved region list.

**Expected result:**  
Deployment is denied.

---

### Test 4 — Application-Team Scope

A workload identity or test application-team user attempts to manage resources inside:

```text
rg-cloudnova-workload-lab
```

**Expected result:**  
Authorized operations succeed.

---

### Test 5 — Cross-Scope Access

The same application-team identity attempts to manage networking resources inside:

```text
rg-cloudnova-network-lab
```

**Expected result:**  
The operation is denied.

---

### Test 6 — Network Administrator Scope

A network administrator performs an authorized network-management operation.

**Expected result:**  
The operation succeeds within the network resource scope.

---

### Test 7 — Resource Lock

Attempt to delete a protected foundational resource.

**Expected result:**  
Azure prevents the deletion.

---

### Test 8 — Administrative Audit

Perform a controlled Azure management change.

**Expected result:**  
The operation appears in Azure Activity Log and can be attributed to the initiating identity.

---

### Test 9 — Policy Compliance

Review Azure Policy compliance after governed resources have been deployed.

**Expected result:**  
The environment provides visibility into compliant and non-compliant resources.

---

## 26. Governance Trade-offs

### Security vs Operational Flexibility

Stronger governance reduces configuration drift but can make experimentation and deployment less flexible.

The project therefore applies strict controls only where requirements are clearly understood.

### Central Control vs Team Autonomy

Application teams need enough permissions to manage their workloads without receiving unnecessary control over shared infrastructure.

Resource-group-scoped RBAC provides a practical balance for the current environment.

### Governance Depth vs Project Scale

A production enterprise landing zone may use:

- Multiple subscriptions.
- Management groups.
- Large Azure Policy initiatives.
- Centralized identity governance.
- Dedicated connectivity subscriptions.
- Central security subscriptions.

Those components are not necessary to demonstrate the core concepts in the current single-subscription lab.

---

## 27. Final Governance Model

The CloudNova landing zone governance model is:

```text
Microsoft Entra ID
        │
        ▼
Azure Pay-As-You-Go Subscription
        │
        ├─────────────────────────────────────────┐
        │                                         │
        ▼                                         ▼
Identity Governance                        Azure Policy
        │                                         │
        ├── Platform Admins                       ├── Required Tags
        ├── Network Admins                        ├── Allowed Regions
        ├── Application Team                      └── Security Baselines
        └── Auditors
        │
        ▼
Azure RBAC
        │
        ▼
Resource Groups
        │
        ├── rg-cloudnova-platform-lab
        ├── rg-cloudnova-network-lab
        └── rg-cloudnova-workload-lab
        │
        ▼
Resource Standards
        │
        ├── Naming Convention
        ├── Governance Tags
        ├── Resource Locks
        ├── Network Segmentation
        └── Cost Attribution
        │
        ▼
Monitoring & Auditability
        │
        ├── Azure Activity Log
        └── Azure Policy Compliance
```

---

## 28. Governance Decision

CloudNova will use a **resource-group-based governance model within the existing Azure subscription**.

The implementation will prioritize:

- Microsoft Entra security groups.
- Scoped Azure RBAC.
- Standardized naming.
- Required governance tags.
- Azure Policy.
- Approved deployment regions.
- Resource protection.
- Cost visibility.
- Azure-native auditability.

This provides a practical landing-zone foundation that demonstrates secure Azure governance without introducing unnecessary enterprise-scale complexity.
