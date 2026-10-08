# Third-Party Identity Security & Microsoft Sentinel Detection Lab

**Focus:** Security Operations (SOC) · Identity and Access Management (IAM) · Third-Party Risk Management (TPRM) · Governance, Risk & Compliance (GRC)

## Project Overview

This portfolio project combines two related, clearly separated exercises:

1. **Simulated third-party privilege escalation investigation:** Analyze fictional vendor authorization and activity datasets with PowerShell, assess unauthorized administrative activity, and document remediation and validation.
2. **Microsoft Entra ID and Sentinel monitoring lab:** Perform controlled security-group membership changes in an Azure lab tenant, investigate the resulting Entra audit events with Kusto Query Language (KQL), create a Microsoft Sentinel scheduled analytics rule, and investigate generated alerts in Microsoft Defender.

The cloud exercise demonstrates a working detection-and-investigation pipeline. A successful membership change is **not**, by itself, evidence of malicious activity or a real-world compromise.

## Architecture and Investigation Workflow

```text
Controlled vendor group-membership change in Microsoft Entra ID
                      |
                      v
              Entra AuditLogs
                      |
                      v
          Azure Log Analytics (KQL)
                      |
                      v
       Microsoft Sentinel analytics rule
                      |
                      v
       Alert / Microsoft Defender investigation
                      |
                      v
     Access review, risk analysis, and remediation testing
```

## Part 1 — Simulated Vendor Privilege Escalation Investigation

### Scenario

Fictional third-party provider **Apex Support Solutions** was approved to perform standard application support on **Finance-Server**. The simulated vendor account `jcarter_vendor` was not approved for privileged administrative access.

### Investigation

Using PowerShell and CSV-based identity/activity datasets, the investigation:

- Reviewed the vendor's authorized access profile.
- Filtered and correlated activity records.
- Identified a denied Finance-Server access attempt and subsequent privileged activity in the simulated dataset.
- Examined the simulated creation of `temp_admin` and its addition to `Server-Administrators`.
- Compared observed activity with the vendor's approved permissions.

### Risk Assessment

| Field | Assessment |
|---|---|
| Finding | Unauthorized third-party privilege escalation **in the simulated dataset** |
| Affected asset | Finance-Server |
| Third party | Apex Support Solutions (fictional) |
| Impact | 5 / 5 |
| Likelihood | 4 / 5 |
| Risk score | 20 / 25 |
| Risk rating | Critical (lab scoring model) |

### Remediation and Validation

The simulated investigation documented removal of unauthorized `Server-Administrators` access and validation that the excessive privileges no longer appeared in the simulated environment. Recommended preventive controls included MFA, periodic vendor access reviews, change approval, and monitoring of privileged group membership.

### Simulated Investigation Evidence

| Evidence | Screenshot |
|---|---|
| Unauthorized privileged access | [01 — Unauthorized access](Screenshots/01-unauthorized-privileged-access.png) |
| Privileged activity investigation | [02 — Activity investigation](Screenshots/02-privileged-activity-investigation.png) |
| Identity and activity correlation | [03 — Correlation](Screenshots/03-identity-activity-correlation.png) |
| Policy violation | [04 — Policy violation](Screenshots/04-privilege-escalation-policy-violation.png) |
| Risk assessment | [05 — Critical risk](Screenshots/05-critical-risk-assessment.png) |
| Remediation validation | [06 — Validation](Screenshots/06-remediation-validation.png) |
| Executive summary | [07 — Executive summary](Screenshots/07-executive-summary.png) |

## Part 2 — Microsoft Entra ID, Log Analytics & Sentinel Detection

### Lab Objective

Demonstrate how a SOC analyst can detect and investigate a change to a sensitive group that governs third-party access, rather than relying on manual review alone.

### Environment and Tools

- **Microsoft Entra ID:** Lab guest identity and `Finance-Server-Access` security group.
- **Azure Log Analytics:** `AuditLogs` queries for group membership changes.
- **Microsoft Sentinel:** Scheduled analytics rule named **Third-Party Vendor - Sensitive Group Membership Addition**.
- **Microsoft Defender portal:** Review of generated alerts and associated event details.

### Procedure and Findings

**1. Establish the access scenario.** A guest identity representing a third-party vendor was associated with the `Finance-Server-Access` group in the lab. Membership was removed and re-added during controlled testing to produce observable audit events.

**2. Investigate Entra audit logs.** KQL queries in Log Analytics identified successful `Add member to group` and `Remove member from group` operations. Expanded event fields were used to correlate the guest identity, target group, operation, result, and timestamp.

**3. Configure the Sentinel rule.** A scheduled analytics rule monitored relevant membership-addition events. The rule was enabled, validated, and configured to create incidents when matching activity was detected.

**4. Confirm detection and investigate.** The lab generated three alerts visible in the Defender portal. An alert investigation showed a successful group-membership addition and corresponding user/group target resources.

**5. Review access and remediation.** Membership removal was successfully performed and observed in audit logs. The guest was subsequently **re-added for detection testing**, so the final demonstrated group state should not be described as permanently remediated. In a production environment, a security analyst would confirm authorization, escalate unexplained changes, remove unauthorized access, and document the final approved state.

### Detection Logic

The lab used Microsoft Entra `AuditLogs` to identify membership-addition operations, with filtering/correlation for the sensitive `Finance-Server-Access` group. The following is an **illustrative investigation query**, not a verbatim export of the saved Sentinel rule:

```kusto
AuditLogs
| where OperationName == "Add member to group"
| where Result =~ "success"
| where tostring(TargetResources) has "Finance-Server-Access"
| project TimeGenerated, OperationName, Result, TargetResources
| order by TimeGenerated desc
```

**Analyst interpretation:** This detection flags a security-relevant change for review; it does not establish malicious intent. A production rule should incorporate approved change records, identity enrichment, exception handling, and appropriate alert tuning.

### Azure/Sentinel Evidence

| Stage | Published evidence |
|---|---|
| Entra group-addition audit event | [01 — AuditLogs group addition](Azure-Sentinel/Screenshots/01-auditlogs-group-addition.png) |
| KQL sensitive-group investigation | [02 — Sensitive-group KQL filter](Azure-Sentinel/Screenshots/02-sensitive-group-kql-filter.png) |
| Analytics rule validation | [03 — Sentinel rule validation](Azure-Sentinel/Screenshots/03-sentinel-rule-validation.png) |
| Created detection rule | [04 — Detection rule created](Azure-Sentinel/Screenshots/04-detection-rule-created.png) |
| Defender alert investigation | [05 — Defender alert investigation](Azure-Sentinel/Screenshots/05-defender-alert-investigation.png) |

> **Evidence note:** Screenshots are from a controlled lab. Personal identifiers and tenant details should be reviewed before public distribution. This project does not claim a real attacker or an actual production incident.

## Skills Demonstrated

- Third-party access review and least-privilege analysis
- Identity and security-group change auditing
- KQL investigation of Microsoft Entra audit events
- Microsoft Sentinel analytics rule configuration and validation
- Microsoft Defender alert triage and evidence correlation
- Risk assessment, incident documentation, remediation testing, and post-change verification
- Distinguishing suspicious activity requiring investigation from confirmed malicious activity

## Key Takeaway

The simulated investigation demonstrates how unauthorized vendor privileges can be identified and addressed through structured access reviews. The separate cloud lab demonstrates how Microsoft Entra audit telemetry, KQL, Sentinel analytics, and Defender alerts can support continuous monitoring of sensitive third-party access changes.

**Project type:** Educational, controlled lab and simulated incident investigation. No production environment or real-world compromise is represented.
