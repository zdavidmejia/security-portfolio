# Investigating Unauthorized Resource Provisioning in Azure

## Scenario

During a routine review of a cloud subscription, an improperly named resource group was discovered that did not follow established organizational naming standards. The objective of the investigation was to determine what had been deployed, identify evidence of ownership, reconstruct the deployment timeline, and determine why existing governance controls failed to prevent the noncompliant deployment.

## Environment

- Microsoft Azure
- Azure Resource Groups
- Azure Storage Account
- Azure Resource Manager (ARM) Deployments
- Azure Policy
- Azure Cloud Shell
- Azure CLI
- Shared Azure training subscription
- Reader-level investigative access
- Azure Portal and CLI validation workflow

## Investigation

### Step 1: Review Resource Group Inventory

I began by reviewing resource groups within the subscription and comparing them against the organization's naming standards. Most resource groups followed a consistent naming convention that included structured identifiers and organizational context.

During this review, I identified a single resource group whose naming structure did not align with the established pattern used throughout the environment. Because naming standards provide operational context and accountability, the outlier became the focus of the investigation.

**Finding:** A resource group existed that did not follow the organization's naming convention. 

**Conclusion:** The naming anomaly indicated a potentially unauthorized or improperly governed deployment and justified additional investigation. 

**Figure 1.** Resource group inventory review identifying a naming-convention outlier. 

images/stage-one.jpg

---

### Step 2: Inspect Resource Metadata and Tags

After identifying the suspicious resource group, I examined the resources deployed within it. The resource group contained a single resource, which simplified the scope of the investigation.

I reviewed the resource metadata and tags to establish ownership and deployment intent. While the deployment did not follow naming standards, several tags had been applied that provided useful context regarding ownership, environment classification, and testing status.

**Finding:** Metadata and tags established ownership information and indicated that the deployment was intended as a test resource. 

**Conclusion:** Resource tags provided valuable investigative evidence and helped establish deployment intent despite governance deficiencies. 

**Figure 2.** Resource metadata and tag review providing ownership and deployment-context information. 

images/stage-two.png

---

### Step 3: Reconstruct Deployment Activity

To determine how the resource was created, I reviewed the resource group's deployment history. Azure Resource Manager records deployment events and provides timestamps, deployment status, and deployment metadata that can be used to reconstruct an incident timeline.

The deployment history contained a successful deployment associated with the resource. Reviewing deployment records confirmed that the resource was provisioned through a formal deployment process and provided evidence of when the deployment occurred.

**Finding:** A successful deployment record existed for the resource and established a timeline for the provisioning event. 

**Conclusion:** Deployment history served as a reliable audit trail and became a key source of forensic evidence during the investigation. 

**Figure 3.** Review of deployment history showing the successful ARM deployment associated with the resource. 

images/stage-three.png

---

### Step 4: Review Governance Controls

The final phase of the investigation focused on determining why the noncompliant deployment was permitted despite existing governance controls.

I reviewed Azure Policy assignments associated with the environment and examined the naming-convention policy responsible for enforcing resource naming standards. The policy was active and was correctly identifying the resource as noncompliant.

Further examination of the policy parameters revealed that the policy effect was configured as **Audit** rather than **Deny**. Audit-mode policies record violations but allow deployments to proceed, whereas Deny-mode policies prevent noncompliant resources from being created.

**Finding:** The naming-convention policy was configured with an Audit effect rather than a Deny effect. 

**Conclusion:** The root cause was not the absence of governance controls, but rather a governance configuration that monitored violations without preventing them. The policy successfully detected the violation but was not configured to block it. 

**Figure 4.** Policy assignment parameters showing the naming-convention policy operating in Audit mode. 

images/stage-four.png

---

## What Broke / What Surprised Me

One unexpected issue occurred while attempting to validate policy configuration through Azure CLI. After identifying the policy assignment in the Azure Portal, I attempted to enumerate policy assignments through the CLI. The command completed successfully but returned an empty result set (`[]`), despite the fact that the assignment was clearly visible within the Portal experience. 

Initially, I assumed the issue was related to scope selection or command syntax. Further investigation through policy-state queries demonstrated that policy evaluations were occurring and that the resource group had been evaluated for compliance. However, assignment details remained unavailable through the CLI within the constraints of the training environment. Because of this, I relied on the Portal's policy parameter view to validate the configuration. 

This was a good reminder that visibility through management interfaces and APIs does not always align perfectly, and that effective investigations often require validating findings through multiple tools. 

---

## Findings and Recommendations

### Findings

The investigation identified a noncompliant resource group that contained a single test resource. Resource metadata established ownership and deployment intent, while deployment history provided evidence of when the resource was provisioned. Review of governance controls confirmed that an active naming-convention policy existed but was configured with an Audit effect instead of Deny. As a result, the deployment was recorded as noncompliant but was not blocked. 

### Recommendations

1. Change the naming-convention policy effect from **Audit** to **Deny** where enforcement is required. 

2. Require ownership, environment, and cost-center tags during the deployment process to improve accountability and resource tracking. 

3. Review temporary Contributor assignments and establish expiration or approval controls for elevated privileges used during testing activities.

---

## What I Learned

- Azure Resource Manager deployment history can be used to reconstruct the timeline of resource creation events. 
- Resource tags provide valuable evidence when establishing ownership and deployment intent. 
- Azure Policy effects significantly impact governance outcomes; Audit records violations while Deny prevents them. 
- Governance failures are often configuration and process issues, not purely technical failures. 
- In future investigations, I would validate policy behavior earlier rather than assuming that the presence of a policy means enforcement is occurring. 

---

## Resume Line

> Investigated unauthorized resource provisioning in an Azure subscription. Traced deployment activity, analyzed resource metadata, and identified the root governance failure: an Azure Policy naming standard configured with an Audit effect rather than Deny, allowing noncompliant resources to be deployed while recording policy violations. 
