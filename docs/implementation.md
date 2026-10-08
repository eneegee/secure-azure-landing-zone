# Implementation — CloudNova Secure Azure Landing Zone

## 1. Purpose

This document records the Azure implementation completed for the CloudNova Secure Azure Landing Zone.

The implementation translates the approved customer requirements, governance design, architecture decision, and network design into a working Azure environment.

The project focuses on:

- Resource organization.
- Microsoft Entra security groups.
- Scoped Azure RBAC.
- Azure Policy governance.
- Governance-tag inheritance.
- Regional deployment restrictions.
- Network segmentation.
- Network Security Groups.
- Private workload deployment.
- Resource protection.
- Policy compliance.
- Azure management-plane auditability.

The environment is a personal Azure Pay-As-You-Go lab representing a simulated customer scenario and is not a production deployment.

---

## 2. Azure Environment

### Subscription

Azure Pay-As-You-Go subscription.

### Primary Region

```text
South Africa North
```

### Landing-Zone Model

The project implements a governed single-subscription landing zone.

CloudNova-specific controls are scoped to the CloudNova resource groups so that unrelated personal Azure projects are not affected.

---

## 3. Resource Group Structure

Three resource groups were created to separate operational responsibilities.

### Platform Resource Group

```text
rg-cloudnova-platform-lab
```

Purpose:

Provides a logical boundary for CloudNova shared and platform-level resources.

Tags:

```text
Environment = Lab
Owner = CloudNova-Platform
Workload = SharedPlatform
CostCenter = CloudNova-IT
```

---

### Network Resource Group

```text
rg-cloudnova-network-lab
```

Purpose:

Contains the CloudNova landing-zone network foundation.

Tags:

```text
Environment = Lab
Owner = CloudNova-Network
Workload = NetworkFoundation
CostCenter = CloudNova-IT
```

---

### Workload Resource Group

```text
rg-cloudnova-workload-lab
```

Purpose:

Contains demonstration workloads deployed into the governed landing zone.

Tags:

```text
Environment = Lab
Owner = CloudNova-AppTeam
Workload = DemoApplication
CostCenter = CloudNova-Engineering
```

---

## 4. Microsoft Entra Security Groups

The following Microsoft Entra security groups were created.

```text
CloudNova-Platform-Admins
CloudNova-Network-Admins
CloudNova-App-Team
CloudNova-Auditors
```

Azure permissions were assigned primarily to groups rather than directly to individual test users.

This allows access to be managed through group membership while keeping Azure RBAC assignments consistent.

---

## 5. Test Identities

The following cloud-only test identities were created.

### Platform Administrator

```text
CloudNova Platform Administrator
```

Membership:

```text
CloudNova-Platform-Admins
```

---

### Network Administrator

```text
CloudNova Network Administrator
```

Membership:

```text
CloudNova-Network-Admins
```

---

### Application User

```text
CloudNova Application User
```

Membership:

```text
CloudNova-App-Team
```

---

### Auditor

```text
CloudNova Auditor
```

Membership:

```text
CloudNova-Auditors
```

Each test identity was placed in the group corresponding to its intended operational responsibility.

---

## 6. Azure RBAC Implementation

Azure RBAC was used to separate platform, networking, application, and audit responsibilities.

No CloudNova test group was granted subscription-level Owner access.

### Platform Administrators

Group:

```text
CloudNova-Platform-Admins
```

Role:

```text
Contributor
```

Scopes:

```text
rg-cloudnova-platform-lab
rg-cloudnova-network-lab
rg-cloudnova-workload-lab
```

Purpose:

Provides CloudNova platform administrators with resource-management capability across the project while keeping role-assignment administration controlled by the subscription owner.

---

### Network Administrators

Group:

```text
CloudNova-Network-Admins
```

Role:

```text
Network Contributor
```

Scope:

```text
rg-cloudnova-network-lab
```

Purpose:

Allows management of CloudNova network resources without granting access to workload administration.

---

### Application Team

Group:

```text
CloudNova-App-Team
```

Role:

```text
Contributor
```

Scope:

```text
rg-cloudnova-workload-lab
```

Purpose:

Allows application-team members to manage CloudNova workload resources without granting control of the shared network foundation.

---

### Auditors

Group:

```text
CloudNova-Auditors
```

Role:

```text
Reader
```

Scopes:

```text
rg-cloudnova-platform-lab
rg-cloudnova-network-lab
rg-cloudnova-workload-lab
```

Purpose:

Provides visibility into the CloudNova environment without modification permissions.

---

## 7. Environment Owner

The personal Azure account used to bootstrap the project remains the subscription owner.

This identity is used for tasks such as:

- Initial environment creation.
- Azure Policy administration.
- RBAC administration.
- Resource-lock administration.
- Project troubleshooting.
- Cleanup.

The subscription Owner role is not used as the normal authorization model for CloudNova test identities.

---

## 8. Allowed Locations Policy

The built-in Azure Policy:

```text
Allowed locations
```

was assigned independently to each CloudNova resource group.

Assignments include:

```text
CloudNova - Allowed Locations - Platform
CloudNova - Allowed Locations - Network
CloudNova - Allowed Locations - Workload
```

The allowed Azure region is:

```text
South Africa North
```

Policy enforcement is enabled.

This prevents governed CloudNova resources from being deployed into unauthorized Azure regions.

---

## 9. Governance Tag Inheritance

A custom Azure Policy initiative was created:

```text
CloudNova - Governance Tag Inheritance
```

The initiative uses the built-in:

```text
Inherit a tag from the resource group if missing
```

policy for the following tags:

```text
Environment
Owner
Workload
CostCenter
```

The initiative was assigned to:

```text
rg-cloudnova-platform-lab
rg-cloudnova-network-lab
rg-cloudnova-workload-lab
```

The policy uses the `Modify` effect so that resources missing the required governance metadata can inherit the corresponding values from their parent resource group.

This reduces reliance on users manually entering governance tags for every deployment.

---

## 10. Governance Policy Model

The implemented governance model is:

```text
CloudNova Resource Group
        │
        ├── Allowed Locations
        │       └── South Africa North
        │
        └── Governance Tag Inheritance
                ├── Environment
                ├── Owner
                ├── Workload
                └── CostCenter
```

This provides both deployment guardrails and consistent resource metadata.

---

## 11. Virtual Network

The CloudNova network foundation was deployed into:

```text
rg-cloudnova-network-lab
```

### Virtual Network

Name:

```text
vnet-cloudnova-lab-san-01
```

Address space:

```text
10.20.0.0/16
```

Region:

```text
South Africa North
```

The address space provides room for segmented workload subnets and future expansion.

---

## 12. Subnet Design

Two workload subnets were deployed.

### Web Subnet

```text
snet-cloudnova-web-lab-01
```

Address range:

```text
10.20.10.0/24
```

Purpose:

Represents the web-facing application tier used as the source of network-validation traffic.

---

### Application Subnet

```text
snet-cloudnova-app-lab-01
```

Address range:

```text
10.20.20.0/24
```

Purpose:

Represents a more restricted internal application tier.

---

### Reserved Address Range

The following range remains reserved for possible future shared services:

```text
10.20.30.0/24
```

No subnet was created for this range because there is currently no workload requirement for it.

---

## 13. Network Security Groups

Two Network Security Groups were created.

### Web NSG

```text
nsg-cloudnova-web-lab-01
```

Associated subnet:

```text
snet-cloudnova-web-lab-01
```

No unnecessary Internet-facing SSH rule was added.

---

### Application NSG

```text
nsg-cloudnova-app-lab-01
```

Associated subnet:

```text
snet-cloudnova-app-lab-01
```

The application NSG contains explicit Web-to-App traffic controls.

---

## 14. Application NSG Rules

### Allow Required Application Traffic

```text
Name: Allow-Web-To-App-8080
Priority: 100
Source: 10.20.10.0/24
Destination: 10.20.20.0/24
Protocol: TCP
Destination port: 8080
Action: Allow
```

### Deny Other Web-to-App Traffic

```text
Name: Deny-Web-To-App-Other
Priority: 110
Source: 10.20.10.0/24
Destination: 10.20.20.0/24
Protocol: Any
Destination port: *
Action: Deny
```

Because lower NSG priority numbers are evaluated first, TCP 8080 is explicitly allowed before the broader Web-to-App deny rule is evaluated.

---

## 15. Network Security Model

The implemented traffic model is:

```text
Web subnet
10.20.10.0/24
        │
        ├── TCP 8080
        │      ↓
        │    ALLOW
        │
        └── Other Web → App traffic
               ↓
             DENY
        │
        ▼
Application subnet
10.20.20.0/24
```

This provides explicit application communication rather than relying on broad default Virtual Network connectivity.

---

## 16. Temporary Validation Workloads

Two Linux virtual machines were deployed into:

```text
rg-cloudnova-workload-lab
```

The VMs were created for network-security validation.

### Web Test VM

```text
vm-cloudnova-web-lab-01
```

Subnet:

```text
snet-cloudnova-web-lab-01
```

Private IP during validation:

```text
10.20.10.4
```

---

### Application Test VM

```text
vm-cloudnova-app-lab-01
```

Subnet:

```text
snet-cloudnova-app-lab-01
```

Private IP during validation:

```text
10.20.20.4
```

---

## 17. Private Workload Design

The test VMs were deployed without public IP addresses.

The intended design is:

```text
Internet
   │
   X
Direct VM administrative exposure
```

Administrative test commands were executed through Azure management capabilities instead of opening SSH directly to the Internet.

This allowed network segmentation to be tested without introducing unnecessary public exposure.

---

## 18. Application Test Service

The application VM was configured with a temporary Python HTTP service listening on:

```text
TCP 8080
```

SSH on:

```text
TCP 22
```

was also confirmed to be listening on the application VM.

This was important for validation because it allowed the project to distinguish between:

```text
service unavailable
```

and:

```text
traffic explicitly blocked by the NSG
```

during the denied TCP 22 test.

---

## 19. Resource Protection

A management lock was applied at the network resource-group scope.

### Scope

```text
rg-cloudnova-network-lab
```

### Lock

```text
CloudNova-Network-Delete-Protection
```

### Lock Type

```text
CanNotDelete
```

Applying the lock at the resource-group level causes network resources within that scope to inherit the deletion protection.

This protects the shared network foundation while still allowing authorized configuration changes.

---

## 20. Resource-Lock Scope

The implemented protection model is:

```text
rg-cloudnova-network-lab
        │
        └── CanNotDelete
                │
                ├── vnet-cloudnova-lab-san-01
                ├── nsg-cloudnova-web-lab-01
                └── nsg-cloudnova-app-lab-01
```

The workload resource group was intentionally not given the same protection because the temporary validation VMs need to remain easy to remove after testing to control cost.

---

## 21. Azure Policy Compliance

Azure Policy compliance was reviewed after implementation.

At the time of validation:

### Workload Scope

```text
Tag Inheritance:
Compliant — 100% (8 out of 8)

Allowed Locations:
Compliant — 100% (8 out of 8)
```

### Network Scope

```text
Tag Inheritance:
Compliant — 100% (3 out of 3)

Allowed Locations:
Compliant — 100% (3 out of 3)
```

### Platform Scope

The policy assignments were present and compliant, but no child resources existed at the time of validation.

---

## 22. Monitoring and Auditability

Azure Activity Log was used to verify that administrative operations are recorded and attributable.

A validated example included:

```text
Operation name: Add management locks
Resource: CloudNova-Network-Delete-Protection
Event initiated by: Environment administrator
Status: Recorded in Azure Activity Log
```

The Activity Log provides management-plane visibility including:

- Operation name.
- Initiating identity.
- Resource.
- Timestamp.
- Operation status.

This supports administrative accountability and investigation.

---

## 23. Cost-Conscious Implementation

The project intentionally avoids deploying services that are not required to demonstrate the landing-zone requirements.

Examples include:

- No Azure Firewall deployed solely for demonstration.
- No ExpressRoute.
- No VPN Gateway.
- No unnecessary hub-and-spoke environment.
- No dedicated SIEM deployment.
- No public IP addresses for the test VMs.
- Small temporary Linux VMs used only for validation.

Temporary compute resources can be deleted after evidence is captured to avoid unnecessary ongoing charges.

---

## 24. Implemented Architecture

```text
Microsoft Entra ID
        │
        ├── CloudNova-Platform-Admins
        ├── CloudNova-Network-Admins
        ├── CloudNova-App-Team
        └── CloudNova-Auditors
        │
        ▼
Scoped Azure RBAC
        │
        ▼
Azure Pay-As-You-Go Subscription
        │
        ├── rg-cloudnova-platform-lab
        │
        ├── rg-cloudnova-network-lab
        │       │
        │       ├── vnet-cloudnova-lab-san-01
        │       │       ├── Web subnet
        │       │       └── App subnet
        │       │
        │       ├── Web NSG
        │       ├── App NSG
        │       └── CanNotDelete lock
        │
        └── rg-cloudnova-workload-lab
                ├── Web test VM
                └── App test VM

Governance Guardrails
        │
        ├── Allowed Locations
        └── Tag Inheritance
                ├── Environment
                ├── Owner
                ├── Workload
                └── CostCenter

Monitoring
        │
        ├── Azure Policy Compliance
        └── Azure Activity Log
```

---

## 25. Implementation Status

The defined CloudNova landing-zone controls have been implemented.

Completed areas include:

- Resource-group separation.
- Entra security groups.
- Scoped RBAC.
- Governance tags.
- Azure Policy.
- Region restrictions.
- Tag inheritance.
- Virtual Network deployment.
- Subnet segmentation.
- NSG-based traffic control.
- Private validation workloads.
- Resource deletion protection.
- Policy compliance visibility.
- Management-plane auditability.

Security and governance behavior is documented separately in:

```text
docs/testing.md
```
