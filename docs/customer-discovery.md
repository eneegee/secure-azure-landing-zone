# Customer Discovery — CloudNova Technologies

## 1. Customer Overview

CloudNova Technologies is a growing software company expanding its use of Microsoft Azure.

As cloud adoption has increased, different teams have begun creating Azure resources independently. This has resulted in inconsistent configurations, unclear resource ownership, broad access permissions, limited cost visibility, and inconsistent security controls.

CloudNova expects additional applications and workloads to be migrated to Azure and wants to establish a secure and governed cloud foundation before that growth continues.

The organization therefore requires a standardized Azure landing zone that provides governance, security, operational consistency, and room for future expansion.

---

## 2. Business Problem

CloudNova currently lacks a consistent framework for deploying and managing Azure resources.

The main challenges include:

- Teams creating resources without common deployment standards.
- Inconsistent resource naming.
- Missing or inconsistent ownership information.
- Broad permissions that may exceed job requirements.
- Limited visibility into cloud spending by team or workload.
- Inconsistent security configuration across resources.
- Limited protection against accidental resource deletion.
- No standardized network structure for future workloads.
- Inconsistent monitoring and auditability.
- Increasing difficulty governing Azure as adoption grows.

Without a standardized foundation, these issues may become more difficult and expensive to correct as additional workloads are deployed.

---

## 3. Business Risks

The current environment creates several risks.

### Unauthorized or Excessive Access

Broad Azure permissions may allow users to access or modify resources beyond their responsibilities.

### Configuration Drift

Without centralized governance, resources may gradually diverge from approved security and operational standards.

### Accidental Resource Deletion

Important resources may be deleted or modified without appropriate protection.

### Weak Resource Ownership

Resources without consistent ownership or business-context information may be difficult to manage, support, or investigate.

### Cost Management Risk

Resources created without tagging or cost-governance standards may make it difficult to understand which teams or workloads are responsible for cloud expenditure.

### Network Security Risk

Future workloads deployed without a defined network structure may create unnecessary exposure or make workload segmentation difficult.

### Limited Auditability

Inconsistent logging and monitoring may make administrative activity or security events difficult to investigate.

### Scalability Risk

A cloud environment without governance standards may become increasingly complex as teams, applications, and Azure services increase.

---

## 4. Customer Objectives

CloudNova wants to establish an Azure foundation that:

- Provides consistent governance for new Azure resources.
- Enforces security requirements where appropriate.
- Applies least-privilege access principles.
- Improves resource ownership and accountability.
- Improves visibility into cloud spending.
- Establishes a structured approach to networking.
- Provides baseline monitoring and auditability.
- Protects important resources from accidental changes or deletion.
- Supports future applications without requiring the cloud foundation to be redesigned.
- Remains practical and cost-conscious.

---

## 5. Identity and Access Requirements

The landing zone should use Microsoft Entra ID as the identity foundation.

Access should follow least-privilege principles.

The design should:

- Separate administrative responsibilities from normal workload access.
- Use Azure RBAC for resource authorization.
- Avoid unnecessary subscription-wide permissions.
- Scope permissions as narrowly as practical.
- Support clear ownership of administrative actions.
- Allow future expansion into stronger privileged-access controls where required.

Broad permissions such as `Owner` or `Contributor` should not be assigned to users unless they are justified by an administrative requirement.

---

## 6. Governance Requirements

CloudNova requires a consistent governance model for Azure resources.

The landing zone should include controls for:

### Resource Naming

Azure resources should follow a documented naming convention that makes their purpose and environment easier to identify.

### Resource Tagging

Resources should include relevant metadata such as:

- Environment
- Application or workload
- Owner
- Cost center or business function

### Azure Policy

Azure Policy should be used where practical to evaluate or enforce required configuration standards.

Initial policy use cases may include:

- Required resource tags.
- Allowed deployment regions.
- Security-related configuration requirements.
- Governance compliance visibility.

### Resource Protection

Important shared or foundational resources should be protected against accidental deletion where appropriate.

### Cost Governance

The environment should provide sufficient tagging and cost visibility to identify resource ownership and support cloud-cost management.

---

## 7. Network Requirements

The landing zone should establish a structured network foundation rather than allowing workloads to be deployed into an unplanned network environment.

The design should consider:

- Virtual Network structure.
- Subnet segmentation.
- Network Security Groups.
- Controlled inbound and outbound connectivity.
- Separation between shared/platform resources and application workloads.
- Private connectivity where appropriate.
- Future expansion into more advanced hub-and-spoke or hybrid connectivity.

The implementation should remain proportional to the size and purpose of the current project.

---

## 8. Security Requirements

The landing zone should provide baseline security controls that can be reused by future workloads.

Security requirements include:

- Least-privilege Azure RBAC.
- Resource-level governance.
- MFA protection through the existing Microsoft Entra security configuration.
- Network segmentation.
- Controlled resource deployment.
- Secure configuration standards.
- Protection against accidental deletion.
- Logging of important Azure management activity.
- Visibility into governance compliance.
- Ability to identify configuration violations.

Security controls should be validated after implementation rather than assumed to work correctly.

---

## 9. Monitoring and Audit Requirements

CloudNova requires visibility into Azure administrative and governance activity.

The environment should support:

- Azure Activity Log visibility.
- Identification of administrative changes.
- Attribution of management actions to identities.
- Policy compliance visibility.
- Review of important configuration changes.
- Monitoring foundations that can be expanded for future workloads.

A full enterprise SIEM or SOC implementation is not required for this project.

---

## 10. Cost Requirements

The architecture should remain cost-conscious.

The project should:

- Prefer Azure-native governance capabilities where possible.
- Avoid deploying expensive services solely for demonstration purposes.
- Use small or consumption-based resources where practical.
- Avoid unnecessary continuously running compute.
- Use tags to improve future cost attribution.
- Remove temporary resources that are no longer needed.

Security controls should not be weakened solely to eliminate small legitimate costs, but unnecessary infrastructure should be avoided.

---

## 11. Technical Constraints

The implementation will be completed in a personal Azure Pay-As-You-Go environment.

The environment does not represent a large enterprise Azure estate.

The project therefore has the following constraints:

- A limited number of Azure subscriptions.
- Limited number of test identities.
- Cost-conscious resource deployment.
- No production workloads.
- No enterprise networking environment.
- No on-premises connectivity requirement.
- No requirement to reproduce every component of Microsoft's enterprise-scale Azure Landing Zone architecture.

Enterprise landing-zone principles will be adapted to the scale of the project rather than reproduced unnecessarily.

---

## 12. Security Assumptions

The project assumes that:

- Microsoft Entra ID is the identity provider for Azure access.
- MFA baseline protection is already available through Microsoft Entra Security Defaults.
- Azure RBAC will be used for resource authorization.
- Administrative actions are performed through identifiable user accounts.
- Test resources do not contain sensitive production information.
- The subscription is used only for learning and simulated customer workloads.
- Future production environments may require stronger identity, networking, monitoring, and governance controls.

---

## 13. Success Criteria

The project will be considered successful when the following conditions are demonstrated.

### Governance

- A documented Azure resource naming convention exists.
- Required governance tags are defined.
- Azure Policy can identify or prevent selected non-compliant resource configurations.
- Policy compliance results can be reviewed.

### Identity and Access

- Administrative permissions are scoped according to least-privilege principles.
- Standard workload identities do not receive unnecessary administrative permissions.
- Azure management activity can be attributed to identifiable users.

### Resource Organization

- Project resources follow a documented organizational structure.
- Resource purpose, environment, and ownership can be identified consistently.

### Network Security

- The landing zone contains a documented network structure.
- Workloads are logically segmented where appropriate.
- Network Security Groups restrict traffic according to defined requirements.

### Resource Protection

- Selected foundational resources are protected against accidental deletion where appropriate.

### Cost Governance

- Resources use defined ownership and environment tags.
- Azure cost information can be associated with tagged project resources where supported.

### Monitoring and Auditability

- Important Azure management actions are visible through Azure Activity Log.
- Azure Policy compliance information is available for governance review.

### Documentation

- The final architecture is documented.
- Governance decisions and security trade-offs are documented.
- Implementation steps are recorded.
- Security and governance controls are tested and evidence is captured.

---

## 14. Security Validation Scenarios

The following scenarios will be used to validate the landing zone.

### Scenario 1 — Required Tag Enforcement

A resource is deployed without a required governance tag.

**Expected result:**  
Azure Policy identifies or prevents the non-compliant deployment according to the selected policy effect.

---

### Scenario 2 — Compliant Resource Deployment

A resource is deployed with the required governance metadata.

**Expected result:**  
The resource satisfies the applicable governance policy.

---

### Scenario 3 — Unauthorized Management Access

A test identity without the required management role attempts to perform a privileged Azure resource-management action.

**Expected result:**  
The operation is denied.

---

### Scenario 4 — Authorized Scoped Administration

An authorized administrator performs an approved management operation within the assigned scope.

**Expected result:**  
The operation succeeds.

---

### Scenario 5 — Network Restriction

Traffic that violates the defined Network Security Group rules is attempted.

**Expected result:**  
The unauthorized traffic path is blocked.

---

### Scenario 6 — Allowed Network Communication

Traffic permitted by the defined network-security rules is attempted.

**Expected result:**  
The authorized traffic path succeeds.

---

### Scenario 7 — Resource Protection

An attempt is made to delete a resource protected by an Azure resource lock.

**Expected result:**  
The deletion is prevented.

---

### Scenario 8 — Auditability

A controlled administrative configuration change is performed.

**Expected result:**  
The action appears in Azure Activity Log and can be attributed to the identity that performed it.

---

### Scenario 9 — Policy Compliance Visibility

Azure Policy compliance is reviewed after governed resources are deployed.

**Expected result:**  
Compliant and non-compliant resources can be identified through policy compliance information.

---

## 15. Out of Scope

The following capabilities are intentionally outside the current project scope:

- Full Microsoft enterprise-scale Azure Landing Zone deployment.
- Large multi-subscription enterprise architecture.
- Enterprise management-group hierarchy.
- Production workload migration.
- Production hybrid networking.
- ExpressRoute.
- Production VPN connectivity.
- Enterprise firewall architecture.
- Full production SIEM/SOC implementation.
- Microsoft Sentinel deployment solely for this project.
- Third-party cloud-security platforms.
- Full enterprise FinOps implementation.
- Production disaster recovery.
- Multi-region production architecture.
- Enterprise Privileged Access Management platform.
- Full identity-governance implementation.
- Production regulatory certification.

These capabilities may be appropriate in a larger production environment but are not required to demonstrate the core governance and security objectives of this project.

---

## 16. Expected Project Outcome

At the end of the project, CloudNova should have a documented and tested Azure landing-zone foundation demonstrating:

- Governed Azure resource deployment.
- Least-privilege authorization.
- Resource organization standards.
- Policy-based governance.
- Network segmentation.
- Resource protection.
- Cost-accountability foundations.
- Monitoring and auditability.
- Security validation.

The resulting environment should provide a stronger foundation for future Azure workloads than an unmanaged subscription in which teams deploy resources without common security and governance standards.
