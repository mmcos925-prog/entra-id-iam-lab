# Lab 1b — Roles & Administrative Units

**Date:** October 14, 2026
**Environment:** Microsoft Entra ID tenant with Entra ID P2 trial
**Admin account used:** lab.admin (Global Administrator)

---

## Objective

Compare a **tenant-wide** admin role assignment with a role **scoped to an Administrative Unit**, and prove that scoped admins can only manage users inside their unit.

---

## What I Built

**Part A — Tenant-wide role**
- Assigned **User Administrator** to Alex IT with scope **Directory** (entire tenant)
- Verified the assignment under Alex IT → Assigned roles
- Removed the assignment so Alex had no tenant-wide rights before Part B

**Part B — Scoped role with an Administrative Unit**
- Created Administrative Unit `HR-AU`
- Added **Jordan HR** as the only member
- Assigned **Helpdesk Administrator** to Alex IT, scoped to **HR-AU** only

**Part C — Testing**
- Signed in as alex.it in a separate incognito window
- Reset Jordan HR's password → **allowed**
- Attempted to reset Sam Finance's password → **blocked**

---

## Problems & Fixes

### Tested as the wrong account
**Problem:** During the first test, I was able to reset Sam Finance's password, which should have been blocked.
**Cause:** I was still signed in as **lab.admin** (Global Administrator), not alex.it — so the test ran with full tenant rights.
**Fix:** Kept my lab Chrome profile signed in as lab.admin and opened a **separate incognito window** signed in as alex.it. Re-ran the test and confirmed Jordan's reset worked while Sam's was blocked.
**Lesson:** Always verify the signed-in identity (top-right corner of the portal) before testing permissions.

---

## Testing & Results

| Action (signed in as alex.it) | Target | Result |
|---|---|---|
| Reset password | Jordan HR (member of HR-AU) | ✅ Allowed |
| Reset password | Sam Finance (not in HR-AU) | ❌ Blocked |

Same role, different scope — Alex's Helpdesk Administrator rights apply only to members of HR-AU.

---

## Key Takeaways

- **Directory scope** = the role applies to the entire tenant
- **Administrative Unit scope** = the role applies only to members of that unit
- AUs support **least privilege for admins** — e.g., a regional or departmental helpdesk can manage only its own users
- AUs can contain **users, groups, and devices**, and membership can be **dynamic**
- Not every role can be scoped to an AU — **Global Administrator** is always tenant-wide
- Role changes may not apply to an existing session immediately; signing out and back in refreshes the user's token

---

## Screenshots

Click a file name to open the screenshot.

| File | Shows |
|---|---|
| [lab1b-01-alex-user-admin-directory-scope.png](../screenshots/lab1b-roles-admin-units/lab1b-01-alex-user-admin-directory-scope.png) | User Administrator with Directory scope |
| [lab1b-02-hr-au-members.png](../screenshots/lab1b-roles-admin-units/lab1b-02-hr-au-members.png) | HR-AU with Jordan HR as member |
| [lab1b-03-hr-au-role-assignment.png](../screenshots/lab1b-roles-admin-units/lab1b-03-hr-au-role-assignment.png) | Helpdesk Administrator assigned to Alex on HR-AU |
| [lab1b-04-alex-helpdesk-au-scope.png](../screenshots/lab1b-roles-admin-units/lab1b-04-alex-helpdesk-au-scope.png) | Alex's role scoped to HR-AU |
| [lab1b-05-jordan-reset-success.png](../screenshots/lab1b-roles-admin-units/lab1b-05-jordan-reset-success.png) | Jordan's password reset succeeds as alex.it |
| [lab1b-06-sam-reset-blocked.png](../screenshots/lab1b-roles-admin-units/lab1b-06-sam-reset-blocked.png) | Sam's password reset blocked as alex.it |
