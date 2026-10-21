# Lab 6 — Monitoring: Log Analytics, KQL & Workbooks

**Date:** October 20, 2026
**Environment:** Microsoft Entra ID tenant with Entra ID P2 trial, Azure subscription
**Accounts used:** personal account (subscription Owner) and lab.admin (Global Administrator)

---

## Objective

Stream Microsoft Entra sign-in and audit logs to a Log Analytics workspace, investigate them with KQL, and visualize Conditional Access results with a workbook.

---

## What I Built

**Part A — Log Analytics workspace**
- Created **law-sc300-lab** in resource group **rg-sc300-lab**
- Granted **lab.admin** the **Log Analytics Contributor** role on the resource group (Azure RBAC is separate from Entra roles, as seen in Lab 1a)

**Part B — Entra diagnostic settings**
- Registered the **Microsoft.Insights** resource provider on the subscription (see Problems)
- Created diagnostic setting **send-to-law** in **Entra ID → Monitoring & health → Diagnostic settings**:
  - Logs: **AuditLogs, SignInLogs, NonInteractiveUserSignInLogs, RiskyUsers, UserRiskEvents**
  - Destination: **law-sc300-lab**

**Part C — Generated activity**
- Signed in to My Apps as Alex, Jordan, and Sam in separate incognito sessions

**Part D — KQL investigation**

Sign-ins per user and result:
```kql
SigninLogs
| where TimeGenerated > ago(1d)
| summarize SignIns = count() by UserPrincipalName, ResultType
```
| User | ResultType | Sign-ins |
|---|---|---|
| lab.admin | 0 (success) | 2 |
| jordan.hr | 50203 | 1 |

Failed sign-ins with the reason:
```kql
SigninLogs
| where ResultType != "0"
| project TimeGenerated, UserPrincipalName, AppDisplayName, ResultType, ResultDescription
```
| Time (local) | User | App | ResultType | ResultDescription |
|---|---|---|---|---|
| 10/20/2026 4:44 PM | jordan.hr | My Apps | 50203 | User has not registered the authenticator app and must register or snooze this notification. |

Audit activity:
```kql
AuditLogs
| where TimeGenerated > ago(1d)
| project TimeGenerated, OperationName, Identity, Result
| order by TimeGenerated desc
```
| Time (local) | Operation | Identity | Result |
|---|---|---|---|
| 4:06 PM | Validate user authentication | lab.admin | success |
| 4:03 PM | Settings_GetSettingsAsync | sam.finance | success |
| 4:02 PM | Settings_GetSettingsAsync | Alex.it | success |
| 3:51 PM | Add service principal | Azure Resource Manager | success |

**Part E — Workbook**
- Opened **Conditional Access Insights and Reporting** against law-sc300-lab (last 24 hours)
- **Total: 2 sign-ins, Success: 2, Failure: 0, Not applied: 0**
- Breakdown: unmanaged device, macOS, browser client, app Azure Portal, approximate IP-based location

---

## Problems & Observations

### 1. Diagnostic setting failed to save: `Microsoft.Insights/register/action`
**Error:** *"The client 'lab.admin@...' does not have authorization to perform action 'Microsoft.Insights/register/action' over scope '/subscriptions/...'"*
**Cause:** The subscription had never used Azure Monitor, so the **Microsoft.Insights resource provider wasn't registered**. Saving the setting tried to register it, and that requires **subscription-scope** permission. lab.admin's role was scoped to the resource group.
**Fix:** As the subscription Owner, registered **Microsoft.Insights** under **Subscription → Resource providers**, then saved the setting again as lab.admin.
**Lesson:** Azure services depend on **resource providers** being registered per subscription. Entra admin roles and Azure RBAC are separate, and some tasks need both.

### 2. Logs arrived gradually
**Observation:** AuditLogs appeared first. SigninLogs took longer, and the workspace only holds data from **after** the diagnostic setting was created (no backfill).
**Evidence:** Alex's and Sam's ~4:02 PM activity appears in **AuditLogs**, but their sign-ins don't appear in **SigninLogs**, which started streaming later.
**Lesson:** Turn on log streaming **before** you need the data. Retention and investigation depend on it.

### 3. ResultType 50203 instead of a wrong-password error
**Observation:** Jordan's failed sign-in was **50203**, not the **50126** (invalid username or password) I expected.
**Explanation:** The **ResultDescription** showed an **Authenticator registration prompt** ("register or snooze") interrupting the sign-in, which the log records as a non-zero result.
**Lesson:** Always check **ResultDescription**, not just the code. Not every non-zero ResultType is an attack or a bad password.

---

## Key Takeaways

- **Entra log retention:** sign-in and audit logs are kept for **7 days (Free)** or **30 days (P1/P2)** in the portal. Send them to **Log Analytics** for longer retention and KQL analysis.
- **Diagnostic settings destinations:**
  - **Log Analytics** for querying, workbooks, and alerts
  - **Storage account** for cheap long-term archive
  - **Event Hub** for streaming to a third-party SIEM (Splunk, etc.)
  - **Microsoft Sentinel** sits on top of a Log Analytics workspace
- **Key log tables:** `SigninLogs`, `AADNonInteractiveUserSignInLogs`, `AuditLogs`, `AADRiskyUsers`, `AADUserRiskEvents`
- **Common ResultType codes:** `0` success, `50126` invalid credentials, `50053` account locked, `50074`/`50076` MFA required, `53003` blocked by Conditional Access, `50105` user not assigned to app (Lab 4)
- **KQL basics:** `where` filters, `project` picks columns, `summarize ... by` aggregates, `order by` sorts, `ago(1d)` sets relative time
- **Workbooks** visualize log data. The Conditional Access insights workbook is the standard way to measure a **Report-only** policy's impact before enforcing it.
- Viewing logs requires roles such as **Reports Reader**, **Security Reader**, or **Global Reader**. Configuring diagnostic settings requires **Security Administrator** or **Global Administrator** plus permissions on the destination.

---

## Screenshots

Click a file name to open the screenshot.

| File | Shows |
|---|---|
| [lab6-01-workspace-created.png](../screenshots/lab6-monitoring/lab6-01-workspace-created.png) | Log Analytics workspace law-sc300-lab |
| [lab6-02-labadmin-rbac.png](../screenshots/lab6-monitoring/lab6-02-labadmin-rbac.png) | Log Analytics Contributor assigned to lab.admin |
| [lab6-03a-insights-provider-registered.png](../screenshots/lab6-monitoring/lab6-03a-insights-provider-registered.png) | Microsoft.Insights resource provider registered |
| [lab6-03b-diagnostic-settings.png](../screenshots/lab6-monitoring/lab6-03b-diagnostic-settings.png) | Diagnostic setting sending Entra logs to the workspace |
| [lab6-04-kql-signins-by-user.png](../screenshots/lab6-monitoring/lab6-04-kql-signins-by-user.png) | KQL: sign-ins by user and result |
| [lab6-05-kql-failed-signins.png](../screenshots/lab6-monitoring/lab6-05-kql-failed-signins.png) | KQL: failed sign-in with ResultDescription |
| [lab6-06-kql-audit-logs.png](../screenshots/lab6-monitoring/lab6-06-kql-audit-logs.png) | KQL: audit log activity |
| [lab6-07-workbook-ca-insights.png](../screenshots/lab6-monitoring/lab6-07-workbook-ca-insights.png) | Conditional Access Insights and Reporting workbook |
