# Architecture Decision — CloudNova Secure Azure Landing Zone

## 1. Decision Context

CloudNova Technologies is expanding its use of Microsoft Azure and expects additional application workloads to be deployed over time.

The current challenge is not the absence of Azure services. The challenge is the absence of a consistent foundation governing how those services are deployed and managed.

Without a defined landing-zone architecture, continued growth could result in:

- Inconsistent resource organization.
- Excessive Azure permissions.
- Configuration drift.
- Weak workload ownership.
- Poor cost attribution.
- Unstructured networking.
- Inconsistent security controls.
- Limited auditability.
- Increasing operational complexity.

CloudNova therefore requires an Azure foundation that introduces governance and security before additional workloads are deployed.

The current project is implemented in a personal Azure Pay-As-You-Go subscription and represents a simulated customer environment.

The selected architecture must therefore demonstrate realistic landing-zone principles without pretending that the environment is a large enterprise Azure estate.

---

## 2. Requirements Driving the Decision

### Governance

The architecture should provide a consistent framework for:

- Resource organization.
- Naming.
- Tagging.
- Policy enforcement.
- Resource protection.
- Cost attribution.

### Identity and Access

Azure access should be controlled through Microsoft Entra ID and Azure RBAC.

Permissions should:

- Follow least privilege.
- Be assigned primarily through security groups.
- Be scoped according to operational responsibility.
- Avoid unnecessary subscription-wide access.

### Workload Separation

Application teams should be able to manage their own workload resources without automatically receiving administrative access to shared networking or platform resources.

### Network Foundation

The environment should provide a structured networking model capable of supporting workload segmentation and future expansion.

### Security

The architecture should provide reusable security guardrails rather than requiring each workload team to establish its own baseline independently.

### Auditability

Administrative and governance activity should be visible and attributable to identifiable identities.

### Cost

The implementation should remain proportional to the scale of the project and avoid unnecessary enterprise services or continuously running infrastructure.

### Scalability

The architecture should be capable of evolving toward a larger Azure environment without requiring the governance principles to be redesigned completely.

---

## 3. Architecture Options Considered

### Option A — Basic Single-Resource-Group Environment

In this model, CloudNova would deploy most project resources into one resource group within the existing subscription.

Azure RBAC could be used for basic access control, but governance and operational separation would remain limited.

Example:

```text
Azure Subscription
        │
        ▼
rg-cloudnova-lab
        │
        ├── Networking
        ├── Applications
        ├── Shared Resources
        └── Monitoring
```

#### Advantages

- Simple to implement.
- Low administrative overhead.
- Easy to understand for a very small environment.
- Minimal initial configuration.

#### Limitations

- Weak separation of responsibilities.
- Harder to delegate access cleanly.
- Application administrators may receive unnecessary access to shared resources.
- Network and workload lifecycle become tightly coupled.
- Governance becomes more difficult as the environment grows.
- Less representative of a scalable landing-zone design.

---

### Option B — Governed Single-Subscription Landing Zone

This model retains the existing Azure subscription but separates resources by responsibility using multiple resource groups.

The environment uses:

- Microsoft Entra security groups.
- Scoped Azure RBAC.
- Azure Policy.
- Standardized naming.
- Governance tags.
- Resource locks.
- Network segmentation.
- Azure Activity Log.
- Azure Policy compliance.

Logical structure:

```text
Microsoft Entra ID
        │
        ▼
Azure Subscription
        │
        ├── rg-cloudnova-platform-lab
        │
        ├── rg-cloudnova-network-lab
        │
        └── rg-cloudnova-workload-lab
```

Responsibilities are separated so that:

```text
Platform Admins
        ↓
CloudNova platform resources

Network Admins
        ↓
CloudNova network resources

Application Team
        ↓
CloudNova workload resources

Auditors
        ↓
Read-only visibility
```

#### Advantages

- Demonstrates scoped administration.
- Supports least privilege.
- Separates workload and networking responsibilities.
- Supports policy-based governance.
- Provides a clearer cost and ownership model.
- Remains practical for the current lab environment.
- Can evolve toward a larger landing-zone architecture.
- Avoids unnecessary enterprise-scale infrastructure.

#### Limitations

- All CloudNova resources remain within one subscription.
- Subscription-level quotas and boundaries remain shared.
- Stronger isolation between environments would require additional subscriptions.
- Some enterprise landing-zone patterns cannot be fully represented.
- Governance design requires more configuration than Option A.

---

### Option C — Enterprise-Scale Multi-Subscription Landing Zone

This model would introduce a larger Azure organizational structure using multiple subscriptions and potentially management groups.

A representative structure could include:

```text
Tenant
  │
  ▼
Management Groups
  │
  ├── Platform
  │     ├── Connectivity Subscription
  │     └── Management Subscription
  │
  └── Landing Zones
        ├── Application Subscription
        ├── Development Subscription
        └── Production Subscription
```

This architecture could provide stronger organizational and isolation boundaries for a large Azure estate.

#### Advantages

- Strong workload and environment isolation.
- Greater separation of operational responsibilities.
- Better support for large organizations.
- Easier application of governance across many subscriptions.
- Suitable for larger production environments.

#### Limitations

- Excessive for the current personal lab environment.
- Greater administrative complexity.
- Additional subscription and governance management.
- More difficult to demonstrate meaningfully without an actual enterprise operating model.
- Would add complexity without materially improving the learning objectives of the current project.

---

## 4. Option Comparison

| Requirement | Option A — Basic Environment | Option B — Governed Single Subscription | Option C — Enterprise Scale |
|---|---|---|---|
| Resource organization | Limited | Strong | Strong |
| Least-privilege delegation | Limited | Strong | Strong |
| Workload/network separation | Weak | Strong | Strong |
| Azure Policy governance | Possible but limited in design | Strong | Strong |
| Cost attribution | Moderate | Strong | Strong |
| Network structure | Basic | Strong | Strong |
| Auditability | Moderate | Strong | Strong |
| Operational simplicity | Strong | Good | Lower |
| Appropriate for current lab | Good | Strong | Low |
| Future extensibility | Limited | Strong | Strong |
| Enterprise-scale isolation | Weak | Moderate | Strong |

---

## 5. Selected Architecture

CloudNova will use:

**Option B — Governed Single-Subscription Landing Zone**

The architecture provides a practical balance between:

- Governance.
- Security.
- Delegated administration.
- Cost.
- Operational simplicity.
- Future scalability.

It introduces meaningful landing-zone controls without adding enterprise-scale components that cannot be justified by the current environment.

---

## 6. Resource Organization Decision

The CloudNova implementation will use three primary resource groups.

### Platform Resource Group

```text
rg-cloudnova-platform-lab
```

Purpose:

Shared or platform-level CloudNova resources.

---

### Network Resource Group

```text
rg-cloudnova-network-lab
```

Purpose:

Landing-zone network infrastructure.

Expected resources include:

- Virtual Network.
- Subnets.
- Network Security Groups.
- Related network components.

---

### Workload Resource Group

```text
rg-cloudnova-workload-lab
```

Purpose:

Demonstration application and workload resources deployed into the governed environment.

---

## 7. Identity and RBAC Decision

Microsoft Entra security groups will represent operational responsibilities.

The proposed groups are:

```text
CloudNova-Platform-Admins
CloudNova-Network-Admins
CloudNova-App-Team
CloudNova-Auditors
```

Azure RBAC assignments will be scoped according to responsibility.

### Platform Administration

Role:

```text
Contributor
```

Scope:

CloudNova project resource groups where platform administration is required.

The group will not automatically receive subscription-level Owner access.

---

### Network Administration

Role:

```text
Network Contributor
```

Scope:

```text
rg-cloudnova-network-lab
```

This allows network administrators to manage networking resources without automatically receiving control over application resources.

---

### Application Administration

Role:

```text
Contributor
```

Scope:

```text
rg-cloudnova-workload-lab
```

This allows the application team to manage its workload while preventing management of the network foundation.

---

### Audit Access

Role:

```text
Reader
```

Scope:

CloudNova project resources.

This provides visibility without modification capability.

---

## 8. Governance Decision

Azure Policy will provide reusable governance guardrails.

The initial governance baseline will focus on:

```text
Required Tags
Allowed Azure Regions
Selected Security Configuration Auditing
```

Policy will be applied at the narrowest scope that meets the project requirement without affecting unrelated resources in the personal Azure subscription.

---

## 9. Policy Enforcement Approach

CloudNova will use different policy effects according to the maturity and risk of each control.

### Deny

Used where the requirement is clear and safe to enforce.

Examples may include:

- Deployment outside approved Azure regions.
- Selected missing governance requirements.

### Audit

Used to identify security or configuration issues before strict enforcement.

This is particularly useful where a `Deny` policy could unintentionally disrupt legitimate resource deployment.

### Modify

May be used when Azure Policy can safely apply or inherit governance metadata such as tags.

The implementation follows:

```text
Define requirement
        ↓
Observe / Audit
        ↓
Validate impact
        ↓
Remediate
        ↓
Enforce
```

---

## 10. Network Architecture Direction

CloudNova will use a segmented Azure Virtual Network rather than placing all workloads into one undifferentiated subnet.

The detailed IP addressing and security rules will be defined in:

```text
docs/network-design.md
```

The architecture will support logical separation between different workload functions.

The implementation will demonstrate:

- Virtual Network design.
- Subnet segmentation.
- Network Security Groups.
- Explicit traffic controls.
- Allowed and denied communication testing.

A full enterprise hub-and-spoke implementation is not required for the current project.

However, the network design should remain compatible with future evolution toward more centralized connectivity if CloudNova grows.

---

## 11. Resource Protection Decision

Selected foundational resources will use:

```text
CanNotDelete
```

resource locks.

The lock will be applied only where accidental deletion represents a meaningful operational risk.

The project will avoid placing locks on every resource because excessive locking would complicate normal administration and cleanup.

The network foundation is a suitable candidate for deletion protection.

---

## 12. Monitoring and Audit Decision

The project will use Azure-native governance and management visibility.

### Azure Activity Log

Used to validate administrative actions such as:

- Resource creation.
- Configuration changes.
- Policy operations.
- RBAC-related activity.
- Resource deletion attempts.

### Azure Policy Compliance

Used to identify:

- Compliant resources.
- Non-compliant resources.
- Governance-policy results.

A full SIEM implementation is intentionally excluded.

---

## 13. Cost Governance Decision

CloudNova will use governance metadata to improve cost attribution.

Primary tags include:

```text
Environment
Owner
Workload
CostCenter
```

CloudNova resource groups also provide a clear project boundary for cost review.

The environment will avoid unnecessary continuously running compute and expensive platform services unless they are required for a validation scenario.

---

## 14. Security Architecture Principles

The selected design follows several security principles.

### Least Privilege

Permissions are granted according to required responsibilities.

### Separation of Duties

Network administration and application administration are treated as separate responsibilities.

### Defense in Depth

Identity controls, RBAC, Azure Policy, network controls, resource protection, and auditability provide multiple layers of governance.

### Secure by Default

Future workloads should be deployed into an environment where governance controls already exist.

### Explicit Authorization

Access should be intentionally assigned rather than assumed because a user already has access elsewhere in the environment.

### Auditability

Administrative activity should be attributable to an identifiable user.

---

## 15. Architecture Trade-offs

### Single Subscription vs Multiple Subscriptions

A multi-subscription design would provide stronger administrative and workload boundaries.

However, the current project does not require enterprise-scale subscription isolation.

The single-subscription design reduces complexity while still allowing meaningful separation through:

- Resource groups.
- Azure RBAC.
- Azure Policy.
- Network segmentation.

A production CloudNova environment could introduce additional subscriptions as organizational scale, workload criticality, regulatory requirements, or environment separation increases.

---

### Built-In Roles vs Custom Roles

The project will initially use Azure built-in roles.

Examples include:

```text
Contributor
Network Contributor
Reader
```

Built-in roles reduce unnecessary custom-role complexity.

If testing demonstrates that a built-in role provides permissions significantly broader than the business requirement, a custom role could be evaluated as a future improvement.

---

### Policy Enforcement vs Developer Flexibility

Strict policy enforcement reduces configuration drift but can prevent legitimate deployment when requirements are poorly understood.

CloudNova will therefore use strict enforcement only for controls that have been validated.

Other controls may begin in `Audit` mode.

---

### Network Security vs Cost

A production enterprise landing zone may include dedicated firewall appliances, centralized inspection, advanced routing, and private connectivity services.

Deploying these solely for a personal demonstration could create unnecessary cost.

The current implementation will demonstrate network segmentation and traffic control using cost-conscious Azure-native capabilities.

---

### Governance Depth vs Operational Complexity

More policies, roles, tags, locks, and administrative boundaries do not automatically result in better governance.

The selected model prioritizes controls that address identified customer risks.

Governance components should exist because they serve a requirement, not simply because Azure provides the feature.

---

## 16. Production Evolution Path

If CloudNova's Azure environment expands significantly, the architecture could evolve toward:

```text
Microsoft Entra ID
        ↓
Management Group Hierarchy
        ↓
Multiple Azure Subscriptions
        │
        ├── Platform / Connectivity
        ├── Management / Security
        ├── Development Workloads
        └── Production Workloads
        ↓
Central Policy Governance
        ↓
Centralized Networking
        ↓
Workload Landing Zones
```

Possible future capabilities include:

- Management groups.
- Multiple subscriptions.
- Dedicated platform subscriptions.
- Centralized connectivity.
- Hub-and-spoke networking.
- Azure Firewall.
- Private DNS architecture.
- Microsoft Entra Privileged Identity Management.
- Conditional Access.
- Centralized security monitoring.
- Microsoft Sentinel.
- Infrastructure as Code.
- Automated policy deployment.
- Enterprise FinOps processes.

These are future architecture considerations and are not part of the completed scope unless explicitly implemented later.

---

## 17. Limitations

The architecture is being implemented in a personal learning environment.

It does not demonstrate:

- Enterprise-scale Azure administration.
- Production landing-zone operations.
- Large management-group hierarchies.
- Multi-subscription production governance.
- Enterprise network operations.
- Production regulatory compliance ownership.
- Production workload migration.
- Enterprise incident-response operations.

The project demonstrates hands-on implementation and validation of representative Azure landing-zone principles.

---

## 18. Architecture Decision

**Decision: Proceed with the Governed Single-Subscription Landing Zone.**

The selected architecture is:

```text
Microsoft Entra ID
        ↓
Operational Security Groups
        ↓
Scoped Azure RBAC
        ↓
Azure Pay-As-You-Go Subscription
        ↓
┌─────────────────────────────────────────┐
│ CloudNova Landing Zone                  │
│                                         │
│  rg-cloudnova-platform-lab              │
│                                         │
│  rg-cloudnova-network-lab               │
│          ↓                              │
│      VNet + Subnets                     │
│      NSGs                               │
│                                         │
│  rg-cloudnova-workload-lab              │
│          ↓                              │
│      Governed Workloads                 │
│                                         │
└─────────────────────────────────────────┘
        ↓
Governance Guardrails
        │
        ├── Naming Standards
        ├── Required Tags
        ├── Azure Policy
        ├── Allowed Regions
        ├── Resource Locks
        └── Cost Attribution
        ↓
Monitoring & Auditability
        │
        ├── Azure Activity Log
        └── Azure Policy Compliance
```

This architecture provides CloudNova with a governed Azure foundation while remaining appropriate for the scale, cost constraints, and objectives of the current project.
