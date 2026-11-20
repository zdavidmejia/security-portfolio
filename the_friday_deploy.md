# The Friday Deploy: Pre-Production Security Review

## Executive Summary

A pre-production review was performed against a newly deployed Azure Function App before promotion into production. The review was conducted using Reader-level access and compared the deployment against an established production baseline.

Four significant findings were identified involving region placement, workload identity, data exposure, and network access controls. The most critical issue was a publicly accessible storage container that was verified to be readable from an unauthenticated browser session. Because this represented active exposure rather than a potential future risk, it was prioritized above all other findings.

---

# Context and Scope

A new notification service, **func-intern-notify**, was deployed into the **rg-madhat-lab-chap4** resource group late on a Friday afternoon and had not undergone a deployment review before consideration for production promotion. 

Access during this review was limited to the Azure Reader role. No written deployment standard was provided. Instead, a known-good production workload, **func-madhat-shared**, located within **rg-madhatlabs-prod-cus**, was used as the baseline for comparison. 

---

# Method

The deployment was reviewed using comparative baseline analysis.

Since no documented standard existed, configuration decisions were evaluated against the existing production implementation across five review areas:

1. Region placement
2. Workload identity
3. Data exposure
4. Inbound network access
5. Remediation priority

Where possible, risks were validated through direct testing rather than assumptions.

---

# Finding 1: The Where

## Observation

The Function App **func-intern-notify** was deployed in **Australia East**, while the production baseline **func-madhat-shared** operates in **Central US**. 

## Verification

The Azure Overview page for both Function Apps was reviewed and the Location field was compared directly. 

## Why It Matters

Cross-region communication introduces unnecessary latency and additional operational cost.

Azure generally does not charge for data ingress, but outbound cross-region traffic (egress) is billable. As a result, every interaction between compute and supporting services may incur additional cost while increasing response time.

## Recommendation

Redeploy the Function App into the organization's standard region to align compute resources with dependent services.

**Severity:** Medium

---

# Finding 2: The Who

## Observation

The Function App had no managed identity configured. System-assigned identity was disabled and no user-assigned identity was attached. 

## Verification

The Identity blade was reviewed and compared against the production workload, which utilizes a user-assigned managed identity. 

## Why It Matters

Managed identities eliminate the need for stored credentials.

Without a managed identity, applications typically rely on connection strings, storage keys, or other embedded secrets. These credentials increase operational risk because they must be stored, protected, rotated, and monitored.

## Recommendation

Attach a user-assigned managed identity and grant the minimum permissions required through Azure RBAC. Remove stored credentials where possible.

**Severity:** Medium

---

# Finding 3: The Leak

## Observation

A storage container was configured with anonymous access enabled. The container allowed blob data to be read without authentication. 

## Verification

The blob URL was copied from the storage account and opened in a private browser session without authentication. The file loaded successfully despite the user not being signed in. 

## Why It Matters

This was not merely a misconfiguration. It was an active exposure.

Data stored within the container was accessible from the public internet without credentials, demonstrating a real and immediate security risk rather than a theoretical one.

## Recommendation

Disable anonymous access immediately and review all publicly accessible content for potential exposure.

**Severity:** High

---

# Finding 4: The Door

## Observation

The Function App had no meaningful inbound access restrictions configured. Access rules allowed traffic from any source and unmatched traffic was configured with an Allow action. 

## Verification

The Networking configuration was reviewed and compared to the production baseline, which uses a deny-oriented access restriction model. 

## Why It Matters

The application was designed to execute scheduled tasks and does not require unrestricted inbound internet access.

Allowing inbound traffic unnecessarily increases the application's attack surface and creates opportunities for future abuse.

## Recommendation

Implement access restrictions consistent with the production baseline. A deny rule should be added so unmatched traffic is implicitly denied.

**Severity:** Medium

---

# Finding 5: The Call

## Priority Assessment

All four findings require remediation; however, one finding represents active exposure while the others represent architectural or operational weaknesses.

### Priority Ranking

| Priority | Finding | Rationale |
|-----------|-----------|-----------|
| 1 | Public Storage Container | Active data exposure confirmed through unauthenticated access. |
| 2 | Open Inbound Access | Unnecessary public attack surface. |
| 3 | Missing Managed Identity | Increased credential-management risk. |
| 4 | Wrong Region | Cost and performance concern rather than direct security exposure. |

## Decision

The publicly accessible storage container should be remediated immediately.

The region mismatch introduces latency and cost concerns. The missing managed identity increases reliance on stored credentials. The open network configuration expands the attack surface.

However, only the public storage container represents an active event already occurring at the time of review. Data was confirmed to be accessible without authentication from an unauthenticated browser session.

The governing principle for prioritization was:

> Active exposure beats potential exposure. Exposure beats hygiene.

---

# Recommendations

## Immediate Actions

1. Disable anonymous access on the storage container.
2. Review exposed content for sensitive information.
3. Validate storage permissions against production standards.

## Short-Term Actions

4. Implement inbound access restrictions.
5. Attach a user-assigned managed identity.
6. Remove any stored credentials no longer required.

## Future Improvements

7. Standardize regional deployment requirements.
8. Create a pre-production deployment checklist.
9. Require deployment reviews before production promotion.

---

# Lessons Learned

This review demonstrated the value of baseline comparison when no written standard exists. By evaluating a new deployment against a known-good production implementation, configuration weaknesses were quickly identified and validated.

More importantly, the exercise reinforced that identifying issues is only part of security work. The ability to prioritize risk, distinguish active exposure from theoretical weaknesses, and communicate remediation priorities is often more valuable than the findings themselves.

---

## Resume Bullet

> Performed a pre-production security review of an Azure Function App deployment against an established production baseline. Evaluated region placement, workload identity, storage exposure, and network access controls. Verified anonymous public access to cloud storage through unauthenticated testing and delivered a risk-prioritized remediation plan emphasizing active exposure over configuration hygiene.
