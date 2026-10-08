# Security and Governance Testing — CloudNova

## 1. Purpose

This document records the validation performed against the CloudNova Secure Azure Landing Zone.

The purpose of testing was to confirm that the implemented governance and security controls behave as designed rather than relying only on configuration screenshots.

The validation focused on:

- Governance-tag inheritance.
- Azure region restrictions.
- Network segmentation.
- Allowed and denied network communication.
- Scoped Azure RBAC.
- Read-only audit access.
- Resource deletion protection.
- Azure Policy compliance.
- Administrative auditability.

---

## 2. Test Environment

### Azure Region

```text
South Africa North
```

### Resource Groups

```text
rg-cloudnova-platform-lab
rg-cloudnova-network-lab
rg-cloudnova-workload-lab
```

### Virtual Network

```text
vnet-cloudnova-lab-san-01
10.20.0.0/16
```

### Web Subnet

```text
snet-cloudnova-web-lab-01
10.20.10.0/24
```

### Application Subnet

```text
snet-cloudnova-app-lab-01
10.20.20.0/24
```

### Validation Workloads

```text
Web VM: 10.20.10.4
App VM: 10.20.20.4
```

---

## 3. Test A — Governance Tag Inheritance

### Objective

Verify that Azure Policy automatically applies required CloudNova governance tags to resources when those tags are missing.

### Test

The Network Security Group:

```text
nsg-cloudnova-web-lab-01
```

was created inside:

```text
rg-cloudnova-network-lab
```

without manually entering any resource tags.

### Parent Resource Group Tags

```text
Environment = Lab
Owner = CloudNova-Network
Workload = NetworkFoundation
CostCenter = CloudNova-IT
```

### Expected Result

Azure Policy should automatically inherit the four governance tags from the resource group.

### Actual Result

After deployment, the Network Security Group contained:

```text
Environment = Lab
Owner = CloudNova-Network
Workload = NetworkFoundation
CostCenter = CloudNova-IT
```

without the tags being manually entered during creation.

### Result

**PASS**

### Security and Governance Significance

This demonstrates that CloudNova governance metadata is applied consistently through Azure Policy rather than depending solely on users remembering to add tags manually.

---

## 4. Test B — Allowed Locations Enforcement

### Objective

Verify that CloudNova resources cannot be deployed outside the approved Azure region.

### Policy

Approved region:

```text
South Africa North
```

### Test

An attempt was made to create:

```text
nsg-cloudnova-region-deny-test
```

inside:

```text
rg-cloudnova-network-lab
```

using:

```text
East US
```

as the deployment region.

### Expected Result

Azure Policy should deny the deployment because East US is not an approved CloudNova location.

### Actual Result

The Azure portal displayed:

```text
Deny (Policy details)
```

against the selected region and prevented the deployment from passing validation.

### Result

**PASS**

### Security and Governance Significance

The test confirms that CloudNova region governance operates as an enforceable deployment guardrail rather than only as documentation.

---

## 5. Test C — Approved Region Deployment

### Objective

Confirm that valid CloudNova resources can still be deployed in the approved region.

### Test

CloudNova networking and workload resources were deployed in:

```text
South Africa North
```

### Expected Result

Resources deployed in the approved region should not be blocked by the Allowed Locations policy.

### Actual Result

The CloudNova Virtual Network, Network Security Groups, and workload resources were successfully deployed in South Africa North.

### Result

**PASS**

---

## 6. Test D — Network Segmentation

### Objective

Verify that the CloudNova web and application tiers are deployed into separate network segments.

### Test

The deployed resources were reviewed.

### Actual Configuration

```text
Web VM
10.20.10.4
        ↓
snet-cloudnova-web-lab-01
10.20.10.0/24
```

and:

```text
App VM
10.20.20.4
        ↓
snet-cloudnova-app-lab-01
10.20.20.0/24
```

### Expected Result

The web and application workloads should reside in separate subnets.

### Actual Result

The workloads were deployed into the intended separate network segments.

### Result

**PASS**

---

## 7. Test E — Authorized Web-to-App TCP 8080

### Objective

Verify that the web workload can communicate with the application workload over the explicitly approved application port.

### Test Preparation

The application VM was configured with a temporary service listening on:

```text
TCP 8080
```

### Test

From:

```text
Web VM
10.20.10.4
```

a connection was made to:

```text
App VM
10.20.20.4:8080
```

### Expected Result

The application NSG should permit the connection.

### Actual Result

The validation returned:

```text
PASS: TCP 8080 connection succeeded
HTTP status: 200
```

### Result

**PASS**

### Security Significance

This demonstrates that the network security design permits required application communication.

---

## 8. Test F — Unauthorized Web-to-App TCP 22

### Objective

Verify that Web-to-App traffic not explicitly authorized by the application NSG is blocked.

### Test Preparation

TCP 22 was confirmed to be listening on the application VM.

This ensured that a failed network connection would represent a network-security control rather than simply an unavailable service.

### Test

From:

```text
Web VM
10.20.10.4
```

a connection was attempted to:

```text
App VM
10.20.20.4:22
```

### Expected Result

The application NSG should deny the connection.

### Actual Result

The validation returned:

```text
PASS: TCP 22 connection was blocked
TimeoutError timed out
```

### Result

**PASS**

### Security Significance

Together with the successful TCP 8080 test, this demonstrates that the CloudNova NSG configuration allows required communication while blocking unnecessary cross-subnet traffic.

---

## 9. Network Validation Summary

The network validation demonstrated:

```text
Web → App TCP 8080 → ALLOWED
Web → App TCP 22   → DENIED
```

### Result

**PASS**

The NSG behavior matched the documented network-security design.

---

## 10. Test G — Application-Team RBAC Scope

### Objective

Verify that the CloudNova Application User can access workload resources without receiving visibility or management access to the CloudNova network and platform scopes.

### Identity

```text
CloudNova Application User
```

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

### Test

The user signed in to the Azure portal.

### Expected Result

The user should be able to access:

```text
rg-cloudnova-workload-lab
```

but should not receive access to:

```text
rg-cloudnova-network-lab
rg-cloudnova-platform-lab
```

### Actual Result

The workload resource group was visible and accessible.

The network and platform resource groups were not visible to the user.

### Result

**PASS**

### Security Significance

This validates delegated application administration without granting unnecessary control over shared landing-zone infrastructure.

---

## 11. Test H — Network Administrator RBAC Scope

### Objective

Verify that the CloudNova Network Administrator can access the network foundation without receiving workload or platform administration access.

### Identity

```text
CloudNova Network Administrator
```

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

### Expected Result

The user should be able to access the network resource group but should not receive access to unrelated CloudNova resource groups.

### Actual Result

The Network Administrator could see:

```text
rg-cloudnova-network-lab
```

but could not see:

```text
rg-cloudnova-workload-lab
rg-cloudnova-platform-lab
```

### Result

**PASS**

### Security Significance

This demonstrates separation between network administration and application administration.

---

## 12. Test I — Auditor Read-Only Access

### Objective

Verify that the CloudNova Auditor can inspect CloudNova resources but cannot perform modification operations.

### Identity

```text
CloudNova Auditor
```

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

### Validation Actions

The Auditor attempted several management operations.

#### Platform Resource Group Deletion

Attempt:

```text
Delete rg-cloudnova-platform-lab
```

Result:

```text
DENIED
```

#### NSG Modification

Attempt:

Add an inbound security rule to the CloudNova web NSG.

Result:

```text
DENIED
```

#### VM Deletion

Attempt:

Delete one of the CloudNova validation virtual machines.

Result:

```text
DENIED
```

#### VM Stop Operation

Attempt:

Stop a running CloudNova VM.

Result:

```text
DENIED
```

### Expected Result

The Auditor should have resource visibility but no modification capability.

### Actual Result

All attempted modification operations were denied.

### Result

**PASS**

### Security Significance

This validates the intended read-only audit role across multiple Azure resource types and management operations.

---

## 13. RBAC Validation Summary

The implemented access boundaries behaved as designed.

```text
Application User
        ↓
Workload RG only
        ↓
PASS

Network Administrator
        ↓
Network RG only
        ↓
PASS

Auditor
        ↓
Read-only across CloudNova scopes
        ↓
PASS
```

The test identities were not granted broad subscription-level permissions.

---

## 14. Test J — Network Resource Protection

### Objective

Verify that the CloudNova network foundation is protected from accidental deletion.

### Configuration

A management lock was applied to:

```text
rg-cloudnova-network-lab
```

Lock:

```text
CloudNova-Network-Delete-Protection
```

Type:

```text
CanNotDelete
```

### Test 1 — NSG Deletion

An attempt was made to delete a Network Security Group protected by the inherited resource-group lock.

### Expected Result

Deletion should be blocked.

### Actual Result

Azure prevented the Network Security Group from being deleted.

### Result

**PASS**

---

### Test 2 — Virtual Network Deletion

An attempt was made to delete:

```text
vnet-cloudnova-lab-san-01
```

### Expected Result

Deletion should be blocked.

### Actual Result

Azure prevented the Virtual Network from being deleted.

### Result

**PASS**

### Security Significance

The tests demonstrate that a resource-group-level `CanNotDelete` lock protects multiple components of the shared network foundation.

---

## 15. Test K — Azure Policy Compliance

### Objective

Verify that the implemented governance controls are visible through Azure Policy compliance reporting.

### Workload Resource Group

At the time of validation:

```text
CloudNova - Tag Inheritance - Workload
Compliant
100% (8 out of 8)
```

and:

```text
CloudNova - Allowed Locations - Workload
Compliant
100% (8 out of 8)
```

### Network Resource Group

At the time of validation:

```text
CloudNova - Tag Inheritance - Network
Compliant
100% (3 out of 3)
```

and:

```text
CloudNova - Allowed Locations - Network
Compliant
100% (3 out of 3)
```

### Platform Resource Group

The platform policy assignments were also shown as compliant.

No child resources existed at the time of validation, so the compliance view showed:

```text
100% (0 out of 0)
```

### Result

**PASS**

### Security and Governance Significance

The compliance view provides centralized evidence that CloudNova resources are operating within the implemented governance guardrails.

---

## 16. Test L — Azure Activity Log Auditability

### Objective

Verify that CloudNova management-plane changes are recorded and attributable to the identity that performed them.

### Validated Event

Azure Activity Log recorded creation of the network resource-group management lock.

Observed information included:

```text
Operation name:
Add management locks

Resource:
CloudNova-Network-Delete-Protection

Resource scope:
rg-cloudnova-network-lab

Event initiated by:
Environment administrator

Time:
Oct 7, 2026
```

### Expected Result

The management operation should appear in Azure Activity Log and identify the initiating administrator and affected resource.

### Actual Result

The operation was recorded and attributable to the administrator who performed it.

### Result

**PASS**

### Security Significance

This demonstrates management-plane auditability and administrative accountability for the CloudNova landing zone.

---

## 17. Governance Validation Summary

CloudNova governance testing demonstrated:

- Required governance tags can be inherited automatically.
- Deployment outside the approved region is denied.
- Deployment inside the approved region succeeds.
- Governance compliance can be centrally reviewed.
- Foundational network resources are protected against accidental deletion.
- Administrative changes are recorded in Azure Activity Log.

### Result

**PASS**

---

## 18. Overall Test Results

| Test | Validation | Result |
|---|---|---|
| Test A | Governance tag inheritance | **PASS** |
| Test B | Unapproved region denied | **PASS** |
| Test C | Approved region deployment | **PASS** |
| Test D | Workload subnet segmentation | **PASS** |
| Test E | Web → App TCP 8080 allowed | **PASS** |
| Test F | Web → App TCP 22 denied | **PASS** |
| Test G | Application-team RBAC scope | **PASS** |
| Test H | Network-admin RBAC scope | **PASS** |
| Test I | Auditor read-only authorization | **PASS** |
| Test J | Network resource deletion protection | **PASS** |
| Test K | Azure Policy compliance visibility | **PASS** |
| Test L | Azure Activity Log auditability | **PASS** |

---

## 19. Security and Governance Conclusion

The CloudNova Secure Azure Landing Zone behaved as intended during the defined validation scenarios.

The implementation demonstrated that:

- Azure resources can be organized according to operational responsibility.
- Azure RBAC can separate platform, network, workload, and audit responsibilities.
- Application administrators can manage workload resources without receiving control of shared networking.
- Network administrators can manage the network foundation without automatically receiving workload administration privileges.
- Auditors can inspect resources without modifying them.
- Azure Policy can automatically apply governance metadata.
- Azure Policy can prevent deployment into unauthorized regions.
- Policy compliance can provide centralized governance visibility.
- Network segmentation can permit required application communication while blocking unnecessary cross-subnet traffic.
- Workloads can be validated without assigning them public IP addresses.
- Resource-group-level locks can protect shared network infrastructure from accidental deletion.
- Azure Activity Log can attribute administrative actions to identifiable users.

The project demonstrates not only Azure configuration but also implementation, validation, troubleshooting, and evidence-based verification of landing-zone security and governance controls.

---

## 20. Final Validation Status

**SECURITY AND GOVERNANCE VALIDATION COMPLETE**

All validation scenarios defined for the implemented CloudNova landing-zone scope were completed successfully.

Remaining project work relates to:

- Final repository presentation.
- README completion.
- Evidence organization.
- Cost cleanup of temporary validation compute.
