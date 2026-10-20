# Lab 5c — Privileged Identity Management (PIM)

**Date:** October 20, 2026
**Environment:** Microsoft Entra ID tenant with Entra ID P2 trial
**Admin account used:** lab.admin (Global Administrator)

---

## Objective

Replace standing admin access with **just-in-time (JIT)** access. Alex is made *eligible* for User Administrator and must activate the role when needed, with MFA, a justification, and approval. The activation expires automatically.

---

## What I Built

**Part A — Eligible assignment**
- In **PIM → Microsoft Entra roles → User Administrator**, added **Alex IT** as **Eligible** (not Active)

**Part B — Role settings for User Administrator**
- **Activation maximum duration:** 2 hours
- **On activation, require:** Azure MFA
- **Require justification on activation:** enabled
- **Require approval to activate:** enabled, approver **lab.admin**

**Part C — Activation request**
- Signed in as Alex → **PIM → My roles → Eligible assignments** → **Activate** User Administrator
- Duration: **1 hour**, reason: "Updating user profiles for HR onboarding"
- Status: **pending approval**

**Part D — Approval**
- As lab.admin, approved the request in **PIM → Approve requests** with a justification

**Part E — JIT access verified**
- Alex's **Active assignments** showed:

| Role | Scope | State | End time |
|---|---|---|---|
| User Administrator | Default Directory | **Activated** | 10/20/2026, 2:14:53 PM (1 hour) |
| Helpdesk Administrator | HR-AU (Administrative unit) | Assigned | Permanent (from Lab 1b) |

- With User Administrator active, Alex edited **Sam Finance's** profile. This was outside his HR-AU Helpdesk scope from Lab 1b.

**Part F — Audit trail (PIM → Resource audit)**

| Time (10/20/2026) | Requestor | Action |
|---|---|---|
| 1:03:35 PM | Lab Admin | Add eligible member to role in PIM requested / completed |
| 1:07:12 PM | Lab Admin | Update role setting in PIM |
| 1:11:21 PM | Alex IT | Add member to role requested (PIM activation) |
| 1:11:22 PM | Alex IT | Add member to role approval requested |
| 1:14:52 PM | Lab Admin | Add member to role request **approved** |
| 1:14:59 PM | Alex IT | Add member to role **completed** (PIM activation) |

---

## Problems & Observations

### Approval screenshot captured from the audit log
**Observation:** I approved the request before taking the screenshot.
**Fix:** Captured the approval from **PIM → Resource audit**, which records the approver, timestamp, and outcome permanently.
**Lesson:** The audit log is the authoritative evidence for compliance reviews, not the approval screen.

### Two roles at once
**Observation:** Alex's active assignments showed both the PIM-activated User Administrator role (tenant-wide, time-bound) and his permanent Helpdesk Administrator role scoped to HR-AU.
**Takeaway:** Role assignments are additive. PIM is what keeps the more powerful role temporary.

---

## Testing & Results

| Test | Result |
|---|---|
| Alex has no User Administrator rights until activation | ✅ Eligible only |
| Activation requires justification and approval | ✅ Request went to pending approval |
| Approver can approve | ✅ lab.admin approved |
| Role active for the requested duration | ✅ Activated, ends 1 hour later |
| Alex can manage users outside HR-AU while active | ✅ Edited Sam Finance's profile |
| Full lifecycle audited | ✅ 7 entries in Resource audit |

---

## Key Takeaways

- **PIM** provides **just-in-time** and **time-bound** privileged access, reducing standing admin exposure
- **Eligible:** the user can activate the role when needed and has no rights until then. **Active:** the user has the role now, permanently or for a set time.
- **Activation requirements:** MFA (or a Conditional Access **authentication context**), justification, ticket information, and **approval**
- **Activation maximum duration** caps how long a role stays active (up to 24 hours)
- **Assignment duration** can make eligibility itself expire, e.g., for contractors
- **Approvers** can't approve their own requests
- **PIM covers:** Microsoft Entra roles, **Azure resource roles** (subscriptions, resource groups), and **PIM for Groups** (membership or ownership of a group)
- **Access reviews** can recertify privileged role assignments on a schedule
- **Alerts** flag risky configurations, e.g., too many Global Administrators or roles assigned outside PIM
- **Best practice:** make all highly privileged roles eligible-only. Keep permanent Global Administrator limited to **break-glass accounts** (Lab 2b).
- **Licensing:** PIM requires **Entra ID P2** or **Entra ID Governance**

---

## Screenshots

Click a file name to open the screenshot.

| File | Shows |
|---|---|
| [lab5c-01-alex-eligible.png](../screenshots/lab5c-pim/lab5c-01-alex-eligible.png) | Alex eligible for User Administrator |
| [lab5c-02-role-settings.png](../screenshots/lab5c-pim/lab5c-02-role-settings.png) | Activation settings: 2-hour max, MFA, justification, approval |
| [lab5c-03-alex-activation-pending.png](../screenshots/lab5c-pim/lab5c-03-alex-activation-pending.png) | Alex's activation pending approval |
| [lab5c-04-approve-request.png](../screenshots/lab5c-pim/lab5c-04-approve-request.png) | Approval recorded in the PIM audit log |
| [lab5c-05-alex-role-active.png](../screenshots/lab5c-pim/lab5c-05-alex-role-active.png) | User Administrator activated with a 1-hour end time |
| [lab5c-06-alex-edits-user.png](../screenshots/lab5c-pim/lab5c-06-alex-edits-user.png) | Alex edits Sam's profile while the role is active |
| [lab5c-07-pim-audit.png](../screenshots/lab5c-pim/lab5c-07-pim-audit.png) | Full PIM audit trail |
