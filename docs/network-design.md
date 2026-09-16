# Network Design — CloudNova Secure Azure Landing Zone

## 1. Purpose

This document defines the network architecture for the CloudNova Secure Azure Landing Zone.

The objective is to provide a structured Azure network foundation that:

- Separates workload functions.
- Restricts unnecessary communication.
- Supports least-privilege network access.
- Avoids unnecessary public exposure.
- Provides room for future expansion.
- Can be validated through controlled allowed and denied traffic tests.

The design is intentionally smaller than a production enterprise network but follows principles that can scale into a larger Azure environment.

---

## 2. Network Design Principles

The network architecture follows these principles.

### Segmentation

Different workload functions should not automatically share the same subnet.

### Least-Privilege Connectivity

A workload should communicate only with the network destinations and ports required for its function.

### Private-by-Default Workloads

Demonstration workload virtual machines will not require public IP addresses.

### Explicit Security Boundaries

Network Security Groups will define permitted and denied traffic between workload segments.

### Separation of Network Administration

Networking resources will be maintained separately from application workload resources.

### Future Extensibility

The address space should provide enough capacity for additional subnets without redesigning the Virtual Network.

### Cost Consciousness

The project will use Azure-native networking controls and avoid deploying expensive networking services solely for demonstration purposes.

---

## 3. Azure Region

The primary region for the CloudNova landing-zone lab will be:

```text
South Africa North
```

The region provides a practical deployment location for the simulated environment.

Before implementation, required resource availability and subscription quota will be confirmed.

If a required service or VM SKU is unavailable, another approved Azure region may be selected and documented before deployment.

---

## 4. Virtual Network

### Name

```text
vnet-cloudnova-lab-san-01
```

### Resource Group

```text
rg-cloudnova-network-lab
```

### Address Space

```text
10.20.0.0/16
```

This provides CloudNova with a sufficiently large private address space while allowing individual workload functions to be separated into smaller subnets.

---

## 5. IP Addressing Strategy

The Virtual Network address space is divided according to workload responsibility.

```text
10.20.0.0/16
        │
        ├── 10.20.10.0/24  → Web workload subnet
        │
        ├── 10.20.20.0/24  → Application workload subnet
        │
        └── 10.20.30.0/24  → Reserved for future shared services
```

Only the required subnets will initially be deployed.

The reserved address range documents future capacity without introducing unnecessary Azure resources.

---

## 6. Web Subnet

### Name

```text
snet-cloudnova-web-lab-01
```

### Address Range

```text
10.20.10.0/24
```

### Purpose

Represents the workload tier that would normally receive or process client-facing application traffic.

For the current project, this subnet will contain a temporary demonstration workload used to validate network communication.

### Associated NSG

```text
nsg-cloudnova-web-lab-01
```

---

## 7. Application Subnet

### Name

```text
snet-cloudnova-app-lab-01
```

### Address Range

```text
10.20.20.0/24
```

### Purpose

Represents an internal application tier.

Resources in this subnet should not be treated as directly public-facing.

The subnet will contain a temporary demonstration workload used to validate network segmentation.

### Associated NSG

```text
nsg-cloudnova-app-lab-01
```

---

## 8. Reserved Shared-Services Address Range

The following range will be reserved for future use:

```text
10.20.30.0/24
```

A subnet will not initially be created unless a project requirement justifies it.

Potential future uses could include shared platform or operational services.

Reserving address space now reduces the risk of needing to redesign the Virtual Network later.

---

## 9. High-Level Network Architecture

```text
                         Azure
                           │
                           ▼
              vnet-cloudnova-lab-san-01
                    10.20.0.0/16
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
     Web Workload Subnet        Application Subnet
       10.20.10.0/24              10.20.20.0/24
              │                         │
              ▼                         ▼
       Web Test Workload         App Test Workload
              │                         │
              └──────── TCP 8080 ──────►
                       ALLOWED

              Other unnecessary
              Web → App traffic
                       │
                       ▼
                     DENIED
```

---

## 10. Public Exposure Decision

The temporary workload virtual machines will not require public IP addresses.

The intended model is:

```text
Internet
   │
   X
Direct VM access
```

Both validation workloads remain privately addressed within the CloudNova Virtual Network.

Administrative test commands can be executed through Azure management capabilities rather than exposing SSH directly to the Internet.

This avoids introducing public administrative access solely for lab convenience.

---

## 11. Network Security Groups

Two Network Security Groups will provide subnet-level security boundaries.

```text
nsg-cloudnova-web-lab-01
```

and:

```text
nsg-cloudnova-app-lab-01
```

The NSGs will be associated with their respective subnets.

---

## 12. Application Subnet Security Rules

The application subnet represents the more restricted internal workload tier.

The main requirement is:

> The web tier may communicate with the application tier only through the application port required by the demonstration.

For the project, that port will be:

```text
TCP 8080
```

### Custom Inbound Rules

| Priority | Rule | Source | Destination | Protocol | Port | Action |
|---:|---|---|---|---|---:|---|
| 100 | `Allow-Web-To-App-8080` | `10.20.10.0/24` | `10.20.20.0/24` | TCP | 8080 | Allow |
| 110 | `Deny-Web-To-App-Other` | `10.20.10.0/24` | `10.20.20.0/24` | Any | Any | Deny |

The explicit allow rule is evaluated before the broader deny rule.

This creates the intended boundary:

```text
Web subnet
    │
    ├── TCP 8080 → Application subnet → ALLOW
    │
    └── Other traffic → Application subnet → DENY
```

---

## 13. Web Subnet Security Rules

The web subnet will also use an NSG.

The current implementation does not require direct Internet administration of the workload.

The web NSG therefore does not need to introduce an inbound SSH rule from the Internet.

The web NSG primarily provides a security boundary for the web workload and can be expanded when a justified application requirement exists.

The project avoids creating broad inbound rules simply to demonstrate NSG functionality.

---

## 14. Azure NSG Default Rules

Azure NSGs include default rules for Virtual Network and Internet-related traffic.

The project will not rely solely on the default Virtual Network allow behavior for the application tier.

The custom application-subnet rules intentionally override the broader default Virtual Network allowance for traffic originating from the web subnet.

The custom rules use higher precedence than the Azure default NSG rules.

---

## 15. Stateful Traffic Behavior

Network Security Groups are stateful.

When an allowed connection is successfully established, return traffic for that connection does not require an additional reciprocal NSG rule.

For example:

```text
Web workload
      │
      │ TCP 8080
      ▼
Application workload
```

If the initial connection is allowed, response traffic associated with that session can return normally.

---

## 16. Demonstration Workloads

Two temporary Linux virtual machines will be used only to validate the network design.

### Web Test VM

Proposed name:

```text
vm-cloudnova-web-lab-01
```

Location:

```text
snet-cloudnova-web-lab-01
```

Purpose:

Acts as the source workload for network-security testing.

---

### Application Test VM

Proposed name:

```text
vm-cloudnova-app-lab-01
```

Location:

```text
snet-cloudnova-app-lab-01
```

Purpose:

Represents a private application service.

A temporary application listener will be used on:

```text
TCP 8080
```

to validate approved communication.

---

## 17. VM Cost Strategy

The virtual machines exist only for validation.

The implementation should:

- Use small Linux VM sizes.
- Avoid unnecessary premium disks.
- Avoid public IP addresses.
- Keep the VMs running only while tests are being performed.
- Delete the temporary VMs and their associated compute resources after network validation if they are no longer required.

The permanent portfolio evidence will remain in GitHub even after temporary test compute is removed.

---

## 18. Allowed Communication Test

### Scenario

The web workload connects to the application workload over:

```text
TCP 8080
```

### Expected Flow

```text
vm-cloudnova-web-lab-01
        │
        │ TCP 8080
        ▼
nsg-cloudnova-app-lab-01
        │
        │ Allow-Web-To-App-8080
        ▼
vm-cloudnova-app-lab-01
```

### Expected Result

```text
ALLOWED
```

The connection should succeed.

---

## 19. Denied Communication Test

### Scenario

The web workload attempts to connect to the application workload on a port that has not been authorized.

For example:

```text
TCP 22
```

### Expected Flow

```text
vm-cloudnova-web-lab-01
        │
        │ TCP 22
        ▼
nsg-cloudnova-app-lab-01
        │
        │ Deny-Web-To-App-Other
        ▼
       BLOCKED
```

### Expected Result

```text
DENIED
```

The connection should fail.

---

## 20. Why TCP 8080 Is Used

TCP 8080 is used as a simple demonstration application port.

The purpose is not to claim that every CloudNova application should use TCP 8080.

It provides a controlled way to demonstrate that:

- Required communication can be explicitly allowed.
- Unnecessary communication can be explicitly denied.
- Network segmentation can be validated through actual traffic.

A production application's permitted ports would be determined by its real technical requirements.

---

## 21. Management Access

Direct Internet-facing SSH will not be introduced solely for testing.

Administrative commands required for validation will use Azure management capabilities such as VM Run Command where practical.

This keeps the test workloads privately addressed while still allowing controlled validation.

A production environment could use stronger centralized administrative access patterns depending on operational requirements.

---

## 22. Network Resource Ownership

Networking resources will be maintained inside:

```text
rg-cloudnova-network-lab
```

Examples include:

- Virtual Network.
- Subnets.
- Network Security Groups.

The test workload virtual machines will be placed in:

```text
rg-cloudnova-workload-lab
```

This demonstrates separation between:

```text
Network lifecycle
```

and:

```text
Workload lifecycle
```

The application team can therefore manage workload resources without automatically receiving control over the network foundation.

---

## 23. RBAC Alignment

The network design aligns with the governance RBAC model.

### CloudNova Network Administrators

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

### CloudNova Application Team

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

The design therefore separates permissions for:

```text
Managing the network
```

from:

```text
Managing workloads using the network
```

---

## 24. Resource Protection

The foundational Virtual Network is a candidate for:

```text
CanNotDelete
```

protection.

This will allow configuration changes while protecting the VNet from accidental deletion.

The lock will be tested during the security-validation phase.

---

## 25. Network Monitoring

The project will use available Azure-native capabilities to validate network configuration and management activity.

The main validation areas include:

- Effective NSG configuration.
- Allowed communication.
- Denied communication.
- Azure Activity Log entries for network-management changes.

A production network-monitoring platform is outside the scope of this project.

---

## 26. Private Connectivity

Private connectivity concepts are relevant to a production landing zone.

Future workloads could use capabilities such as:

- Private Endpoints.
- Private DNS.
- Private access to Azure PaaS services.
- Centralized routing.

These capabilities are not required for the initial CloudNova network validation.

They should be introduced only when a workload requirement justifies them.

---

## 27. Hub-and-Spoke Consideration

A larger CloudNova environment could evolve toward:

```text
                    Hub VNet
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
     Workload A   Workload B   Workload C
        Spoke        Spoke        Spoke
```

The hub could later provide centralized:

- Connectivity.
- Firewalling.
- DNS.
- Shared services.
- Hybrid connectivity.

The current project does not have enough independent workloads to justify deploying a hub-and-spoke architecture.

The single segmented VNet therefore provides the appropriate complexity for the current requirement.

---

## 28. Network Security Validation Scenarios

### Test 1 — VNet Deployment

Verify that:

```text
vnet-cloudnova-lab-san-01
```

is deployed using:

```text
10.20.0.0/16
```

**Expected result:**  
The Virtual Network exists with the documented address space.

---

### Test 2 — Subnet Segmentation

Verify the existence of:

```text
snet-cloudnova-web-lab-01
10.20.10.0/24
```

and:

```text
snet-cloudnova-app-lab-01
10.20.20.0/24
```

**Expected result:**  
The workloads are located in separate network segments.

---

### Test 3 — Approved Application Traffic

From the web workload, attempt to reach the application workload over:

```text
TCP 8080
```

**Expected result:**  
The connection succeeds.

---

### Test 4 — Unauthorized Cross-Subnet Traffic

From the web workload, attempt to reach the application workload over an unauthorized port such as:

```text
TCP 22
```

**Expected result:**  
The connection is denied.

---

### Test 5 — No Public VM Exposure

Verify that the temporary workload VMs do not have public IP addresses.

**Expected result:**  
Both workload VMs are privately addressed.

---

### Test 6 — Network Administration Scope

A network administrator performs an approved network-management action within:

```text
rg-cloudnova-network-lab
```

**Expected result:**  
The operation succeeds.

---

### Test 7 — Application-Team Network Modification

The application-team identity attempts to modify a protected network resource.

**Expected result:**  
The operation is denied because its Contributor assignment is scoped to the workload resource group rather than the network resource group.

---

### Test 8 — VNet Deletion Protection

Attempt to delete the protected Virtual Network while the `CanNotDelete` lock is active.

**Expected result:**  
Azure prevents the deletion.

---

### Test 9 — Network Auditability

Review a controlled network configuration change through Azure Activity Log.

**Expected result:**  
The operation can be attributed to the identity that performed it.

---

## 29. Network Security Matrix

| Source | Destination | Protocol/Port | Expected Result |
|---|---|---|---|
| Web subnet | App subnet | TCP 8080 | **Allow** |
| Web subnet | App subnet | TCP 22 | **Deny** |
| Internet | Web test VM management | TCP 22 | **No direct exposure** |
| Internet | App test VM management | TCP 22 | **No direct exposure** |

---

## 30. Security Trade-offs

### Segmentation vs Complexity

Using separate subnets creates additional configuration but provides a clearer security boundary than placing all workloads into one subnet.

### Private Workloads vs Administrative Convenience

Avoiding public IP addresses makes direct administration less convenient but reduces unnecessary exposure.

### NSGs vs Enterprise Firewalling

Network Security Groups provide effective subnet-level traffic controls for the current project.

A production enterprise environment may require centralized inspection or firewall capabilities depending on risk and connectivity requirements.

### Single VNet vs Hub-and-Spoke

Hub-and-spoke provides stronger centralized connectivity patterns for larger environments.

For the current project, deploying multiple VNets and centralized appliances would introduce complexity without a demonstrated requirement.

---

## 31. Final Network Design

The selected network design is:

```text
Azure Subscription
        │
        ▼
rg-cloudnova-network-lab
        │
        ▼
vnet-cloudnova-lab-san-01
10.20.0.0/16
        │
        ├─────────────────────────────┐
        │                             │
        ▼                             ▼
snet-cloudnova-web-lab-01     snet-cloudnova-app-lab-01
10.20.10.0/24                 10.20.20.0/24
        │                             │
        ▼                             ▼
nsg-cloudnova-web-lab-01      nsg-cloudnova-app-lab-01
        │                             │
        ▼                             ▼
Web Test Workload ──8080──► Application Test Workload
        │                             ▲
        └──── Other traffic ──X───────┘
```

The design provides:

- Workload segmentation.
- Explicit traffic authorization.
- Private workload addressing.
- Scoped network administration.
- Cost-conscious security controls.
- Network-policy validation.
- Future address-space capacity.

---

## 32. Network Design Decision

CloudNova will implement a **single segmented Azure Virtual Network** using separate web and application subnets.

Network Security Groups will explicitly permit the required application communication while denying unnecessary cross-subnet traffic.

Temporary private Linux workloads will be used to validate both allowed and denied traffic paths.

The implementation intentionally avoids public administrative exposure, unnecessary firewall appliances, and unjustified enterprise network complexity.

The architecture provides an appropriate secure networking foundation for the current CloudNova landing-zone scope while retaining a clear evolution path for future workloads.
