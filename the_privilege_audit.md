# Azure RBAC Privilege Audit

## Scope and Methodology

This audit reviewed Azure RBAC assignments within the Mad Hat Labs environment using multiple auditing methods to identify excessive privilege, orphaned assignments, privilege-lifecycle issues, and hidden access paths. The assessment was conducted using Reader-level access along with a PIM-eligible role that was activated during the final stage of the audit.

The goal was not simply to find individual permission assignments, but to compare how different Azure tools expose RBAC information and understand the blind spots of each auditing method.

Methods used:

1. Access Control (IAM) Role Assignment Export
2. Azure CLI Role Assignment Enumeration
3. Azure Resource Graph using KQL
4. Privileged Identity Management (PIM) Export
5. Synthesized Privilege Hunt

---

## Methodology Comparison

| Method | Sees | Misses |
|----------|----------|----------|
| IAM Blade / Export | Active assignments, inherited permissions, scope relationships | Group-member expansion and buried orphaned principals |
| Azure CLI | Structured RBAC data, null principal names, deleted accounts | Must query individual scopes |
| Resource Graph (KQL) | Tenant-wide access review in a single query | Eligible assignments |
| PIM Export | Eligible vs Active access, assignment lifecycle | Assignments outside PIM |
| Hunt / Synthesis | Hidden privilege paths and over-provisioning patterns | Requires combining findings from multiple tools |

---

## Finding 1 – IAM Role Assignment Export

The audit began with the Access Control (IAM) export at the subscription level. Reviewing the exported role-assignment report revealed a recurring pattern of Owner permissions assigned repeatedly across numerous scopes.

Although the export successfully exposed active role assignments, it also revealed a limitation. Assignments granted to groups could be identified, but the export alone did not expose every user indirectly receiving access through group membership. Determining who actually inherits permissions from group assignments requires additional investigation.

The IAM export served as the baseline dataset for the remainder of the audit and revealed the first signs of privilege sprawl.

images/finding1.png

*Figure 1. Subscription-level role-assignment export used to identify repeated high-privilege assignments.*

---

## Finding 2 – Azure CLI

The role-assignment data was reviewed again through Azure CLI. While the information was similar to the IAM export, the structured output made anomalies easier to identify.

The CLI surfaced an assignment whose principal identifier remained present while the principal name no longer resolved. This indicated an orphaned role assignment associated with a deleted identity.

This demonstrated an important governance issue. Azure tracks role assignments through identifiers rather than display names, allowing permissions to persist even after an account appears to have been removed.

The primary advantage of the CLI was repeatability and automation. The same review can be scheduled, scripted, and reused without manual portal navigation.

images/finding2.png

*Figure 2. Azure CLI output highlighting an orphaned role assignment through missing identity information.*

---

## Finding 3 – Azure Resource Graph

The third phase expanded the audit from a single scope to the entire Azure tenant.

Using Azure Resource Graph and Kusto Query Language (KQL), authorization resources were queried directly through the authorizationresources table. This allowed RBAC assignments to be reviewed across all visible resources simultaneously instead of one scope at a time.

Resource Graph proved significantly more scalable than portal or CLI reviews when searching for assignment patterns. However, it only exposed active role assignments and did not surface eligible assignments managed through Privileged Identity Management.

images/finding3.png

*Figure 3. Azure Resource Graph query used to search role-assignment data across the tenant.*

---

## Finding 4 – Privileged Identity Management

The audit then shifted from current access to potential access.

Unlike the previous methods, PIM exposed whether permissions were Active, Eligible, or Expired. The exported report provided visibility into assignment state, membership type, and assignment duration.

The review demonstrated an important governance distinction: users may possess significant privilege without continuously holding it. Eligibility grants the ability to activate access when required, significantly reducing standing administrative exposure.

PIM provided the strongest view of privilege lifecycle, but only for assignments managed through the PIM framework.

images/finding4.png

*Figure 4. PIM export demonstrating the distinction between Active and Eligible role assignments.*

---

## Finding 5 – The Hunt

The final phase combined all previous methods.

An eligible PIM assignment was activated, exposing additional audit scope that was not visible during the initial review. This ultimately revealed a previously hidden resource group that required direct investigation.

A role-assignment export from the hidden scope confirmed the existence of an additional Owner assignment associated with the same pattern of excessive privilege identified earlier in the assessment.

The most important lesson from the Hunt was that no individual tool revealed the entire problem. Only by correlating IAM exports, CLI findings, Resource Graph results, PIM eligibility, and direct scope investigation could the full access picture be understood.

images/finding5a.png

*Figure 5. PIM activation process used to elevate eligible access during the audit.*

images/finding5b.png

*Figure 6. Hidden resource-group investigation used to validate excessive privilege assignments.*

images/finding5c.png

*Figure 7. Additional evidence collected during the privilege hunt.*

images/finding5d.png

*Figure 8. RBAC review of newly exposed audit scope.*

images/finding5e.png

*Figure 9. Validation of assignment patterns identified earlier in the audit.*

images/finding5f.png

*Figure 10. Final evidence gathered from the hidden resource-group review.*

---

## Findings

### High Severity

- Repeated Owner assignments applied across numerous scopes to a single identity.
- Owner permissions identified within a previously hidden resource group.
- Orphaned role assignments associated with deleted identities.

### Medium Severity

- Direct user assignments rather than group-based role assignment strategy.
- Standing permissions that could be moved into PIM eligibility.

### Low Severity

- Limited visibility into group membership through standard IAM exports.
- Visibility gaps caused by using a single audit method.

---

## Recommendations

1. Revoke orphaned role assignments associated with deleted principals.

2. Replace redundant Owner permissions with the least-privileged role required for the business function.

3. Assign permissions to role-based groups whenever possible rather than directly to individual users.

4. Move standing privileged access into PIM eligibility with:
   - MFA enforcement
   - Justification requirements
   - Time-limited activation

5. Perform quarterly RBAC audits using a combination of:
   - IAM exports
   - Azure CLI
   - Resource Graph
   - PIM reporting

6. Review hidden or low-visibility scopes as part of every privileged-access review.

---

## What Surprised Me

The most surprising lesson was discovering that every audit tool exposed a different piece of the overall privilege picture. None of the tools independently revealed the complete state of access within the environment.

I was also surprised that orphaned assignments can persist after identities are removed. The role assignment remains because Azure tracks permissions through principal identifiers rather than display names.

Finally, I underestimated the importance of eligible privilege. Prior to this exercise, I viewed privilege as either present or absent. The audit demonstrated that potential access can be just as important as active access from a governance perspective.

---

## What I Learned

- Active and Eligible access answer different governance questions.
- Azure CLI is significantly more effective for repeatable RBAC auditing than manual portal review.
- Resource Graph scales access reviews across entire tenants through KQL.
- Orphaned assignments represent a governance risk even after the associated account has been removed.
- Effective privilege audits depend on combining data from multiple Azure tools rather than relying on a single interface.

---

## Resume Line

> Conducted a multi-method Azure RBAC audit using IAM exports, Azure CLI, Resource Graph, and Privileged Identity Management. Identified privilege-sprawl patterns, orphaned role assignments, hidden Owner access, and lifecycle-based governance risks while documenting the strengths and limitations of each auditing methodology.
