# Lab 5a — Entitlement Management: Catalogs & Access Packages

**Date:** October 20, 2026
**Environment:** Microsoft Entra ID tenant with Entra ID P2 trial
**Admin account used:** lab.admin (Global Administrator)

---

## Objective

Replace manual access provisioning with a self-service, approval-based, time-limited access package. A user requests access, a designated approver approves it, and Entra grants the access automatically and removes it when it expires.

---

## What I Built

**Part A — Resource and catalog**
- Created the security group **HR-Payroll-Access** with **Assigned** membership and no members
- Created the catalog **HR Resources** (enabled, not enabled for external users)
- Added two resources to the catalog:
  - **HR-Payroll-Access** (group)
  - **Microsoft Entra SAML Toolkit** (application)

**Part B — Access package: HR Payroll Access**
- **Resource roles:**
  - HR-Payroll-Access → **Member**
  - Microsoft Entra SAML Toolkit → **msiam_access**
- **Request policy:**
  - Who can get access: users in my directory → **All members (excluding guests)**
  - Who can request: **Self** (Admin is always allowed to assign directly)
  - Requestor justification: **Required**
  - Approval: **Required**, 1 stage, approver **Alex IT**, decision within **14 days**, approver justification **required**
- **Lifecycle:** assignments **expire after 30 days**

**Parts C–E — Request, approval, and delivery**
- **Sam Finance** requested *HR Payroll Access* at **myaccess.microsoft.com** with a business justification. The request showed **Pending approval**.
- **Alex IT** approved it from the **Approvals** page in My Access, with a justification
- Entitlement management **delivered** the assignment automatically:
  - Sam's assignment status: **Delivered**, expiring in 30 days
  - Sam added to **HR-Payroll-Access** with no manual group change

---

## Problems & Observations

### 1. No "User" role for the SAML Toolkit app
**Observation:** When adding the app as a resource role, there was no "User" role to choose.
**Explanation:** Gallery apps that don't publish their own app roles expose the default role **msiam_access**, which grants basic sign-in access.
**Result:** Selected **msiam_access**.

### 2. Self-request was not enabled by default
**Observation:** In the request policy, only **Admin** was checked under *Who can request access*.
**Impact:** Without **Self**, Sam couldn't request the package in My Access. Only admins could assign it directly.
**Fix:** Checked **Self** so users in scope can request it themselves.

### 3. Some policy options require Entra ID Governance
**Observation:** Several settings (sponsors as approvers, on-behalf-of requests, Verified ID requirements) were marked as requiring a **Microsoft Entra ID Governance** subscription.
**Result:** The core flow (catalog, access package, approval, expiration) works with **Entra ID P2**.

---

## Testing & Results

| Step | User | Result |
|---|---|---|
| Request access package with justification | Sam Finance | ✅ Request submitted, status Pending approval |
| Approve request with justification | Alex IT | ✅ Approved from My Access → Approvals |
| Assignment delivery | — | ✅ Delivered, expires in 30 days |
| Group membership | Sam Finance | ✅ Added to HR-Payroll-Access automatically |

---

## Key Takeaways

- **Entitlement management** automates the access lifecycle: request → approval → delivery → expiration, with every step audited
- **Catalog:** a container of resources (groups, Teams, apps, SharePoint sites) whose management can be **delegated** to catalog owners, so IT doesn't have to own every access decision
- **Access package:** a bundle of **resource roles** that users request as one unit
- **Policy:** defines **who can request**, **who approves** (up to 3 stages), **justification** requirements, and **how long** access lasts
- **Dynamic groups cannot be access package resources**, because their membership is controlled by a rule
- **myaccess.microsoft.com** is the user portal for requesting access and for approvers to approve
- **Expiration** enforces least privilege over time: access is removed automatically unless it's renewed or extended
- Approvers can be specific users, the requestor's **manager**, or (with Entra ID Governance) **sponsors**
- **Licensing:** entitlement management requires **Entra ID P2** or **Entra ID Governance**. Advanced features need Entra ID Governance.
- **Connected organizations** let external users request access packages (B2B), a common exam scenario

---

## Screenshots

Click a file name to open the screenshot.

| File | Shows |
|---|---|
| [lab5a-01-catalog-resources.png](../screenshots/lab5a-entitlement-management/lab5a-01-catalog-resources.png) | HR Resources catalog with the group and app |
| [lab5a-02-resource-roles.png](../screenshots/lab5a-entitlement-management/lab5a-02-resource-roles.png) | Resource roles: Member and msiam_access |
| [lab5a-03-request-policy.png](../screenshots/lab5a-entitlement-management/lab5a-03-request-policy.png) | Request policy: self-request, justification, approval by Alex |
| [lab5a-04-access-package-created.png](../screenshots/lab5a-entitlement-management/lab5a-04-access-package-created.png) | HR Payroll Access package created |
| [lab5a-05-sam-request-submitted.png](../screenshots/lab5a-entitlement-management/lab5a-05-sam-request-submitted.png) | Sam submits the request in My Access |
| [lab5a-06-sam-request-pending.png](../screenshots/lab5a-entitlement-management/lab5a-06-sam-request-pending.png) | Request pending approval |
| [lab5a-07-alex-approves.png](../screenshots/lab5a-entitlement-management/lab5a-07-alex-approves.png) | Alex approves the request |
| [lab5a-08-assignment-delivered.png](../screenshots/lab5a-entitlement-management/lab5a-08-assignment-delivered.png) | Assignment delivered with 30-day expiration |
| [lab5a-09-sam-group-member.png](../screenshots/lab5a-entitlement-management/lab5a-09-sam-group-member.png) | Sam added to HR-Payroll-Access automatically |
