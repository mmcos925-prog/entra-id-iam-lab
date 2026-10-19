# Lab 1a — Users, Groups, Licensing & Dynamic Membership

**Date:** October 13–14, 2026
**Environment:** Microsoft Entra ID tenant (free Azure account) with Entra ID P2 trial
**Admin account used:** lab.admin (Global Administrator)

---

## Objective

Set up a lab tenant, create test users with department attributes, compare assigned vs. dynamic group membership, and license users for Entra ID P2 features.

---

## What I Built

**Tenant setup**
- Created a free Azure account, which automatically provisioned a Microsoft Entra ID tenant and an Azure subscription
- Created resource group `rg-sc300-lab` and reviewed Azure RBAC built-in roles (Owner, Contributor, Reader)
- Confirmed my personal account holds **Owner** on the resource group, **inherited from the subscription**
- Created a dedicated tenant admin account, `lab.admin`, with the Global Administrator role

**Users**

| Display name | UPN prefix | Department |
|---|---|---|
| Alex IT | alex.it | IT |
| Jordan HR | jordan.hr | HR |
| Sam Finance | sam.finance | Finance |

**Groups**
- `IT-Team` — Security group, **Assigned** membership (Alex IT added manually)
- `HR-Dynamic` — Security group, **Dynamic User** membership with the rule:

```
(user.department -eq "HR")
```

**Licensing**
- Activated the **Microsoft Entra ID P2** trial (100 licenses)
- Assigned P2 to lab.admin, Alex IT, Jordan HR, and Sam Finance (4 of 100 used)

---

## Problems & Fixes

### 1. Couldn't start the P2 trial from my personal account
**Problem:** The Try / Buy option wasn't available while signed in with my personal Microsoft account.
**Cause:** A personal Microsoft account behaves like an external identity in the tenant it creates, which limits some admin and billing features.
**Fix:** Created a tenant-native admin account (`lab.admin`) with Global Administrator, signed in as that account, and activated the trial from **Billing → Trials**.

### 2. License assignment failed: "invalid usage location"
**Problem:** All 4 license assignments failed with *"License assignment cannot be done for user with invalid usage location."*
**Cause:** Microsoft requires a user's **Usage location** to be set before a license can be assigned, since service availability depends on country.
**Fix:** Set Usage location to **United States** for each user in the Microsoft 365 admin center (Users → Active users → Licenses and apps), then assigned the licenses. License assignment is now done in the M365 admin center rather than the Entra admin center.

### 3. Dynamic group processed but showed 0 members
**Problem:** HR-Dynamic showed processing status "Succeeded" but had 0 members.
**Cause:** The rule ran correctly but no user matched — the department value didn't line up with the rule.
**Fix:** Verified Jordan HR's Department attribute was set exactly to `HR`, then used **Validate Rules** to confirm the match. Jordan was added automatically after processing.

---

## Testing & Results

- **Validate Rules** confirmed Jordan HR matches the rule and Sam Finance does not
- Changed Sam Finance's department to `HR` — Sam was **automatically added** to HR-Dynamic with no manual change to the group
- Changed Sam back to `Finance` afterward to reset the lab

---

## Key Takeaways

- **Assigned groups** require manual membership changes; **dynamic groups** update automatically based on user attributes — the foundation for joiner/mover/leaver automation
- Dynamic groups require Entra ID **P1 or P2**
- **Role-assignable groups cannot use dynamic membership** (prevents users from gaining admin rights by changing their own attributes)
- **Usage location** must be set before license assignment
- **Entra roles** (identity side) and **Azure RBAC roles** (resource side) are separate — Global Administrator does not automatically grant access to Azure resources
- Azure RBAC roles are **inherited** down the hierarchy: Management group → Subscription → Resource group → Resource

---

## Screenshots

Click a file name to open the screenshot.

| File | Shows |
|---|---|
| [lab1a-01-tenant-overview.png](../screenshots/lab1a-users-groups/lab1a-01-tenant-overview.png) | Tenant ID and primary domain |
| [lab1a-02-resource-group-rbac-roles.png](../screenshots/lab1a-users-groups/lab1a-02-resource-group-rbac-roles.png) | Resource group RBAC: Owner inherited from subscription, built-in roles |
| [lab1a-03-lab-admin-global-admin.png](../screenshots/lab1a-users-groups/lab1a-03-lab-admin-global-admin.png) | lab.admin with Global Administrator |
| [lab1a-04-test-users-list.png](../screenshots/lab1a-users-groups/lab1a-04-test-users-list.png) | Test users with departments |
| [lab1a-05-it-team-assigned-group.png](../screenshots/lab1a-users-groups/lab1a-05-it-team-assigned-group.png) | IT-Team assigned membership |
| [lab1a-06-entra-p2-trial-active.png](../screenshots/lab1a-users-groups/lab1a-06-entra-p2-trial-active.png) | Entra ID P2 trial (100 licenses) |
| [lab1a-07-license-error-usage-location.png](../screenshots/lab1a-users-groups/lab1a-07-license-error-usage-location.png) | Usage location license error |
| [lab1a-08-usage-location-fix.png](../screenshots/lab1a-users-groups/lab1a-08-usage-location-fix.png) | Usage location set to United States |
| [lab1a-09-licenses-assigned-4-of-100.png](../screenshots/lab1a-users-groups/lab1a-09-licenses-assigned-4-of-100.png) | 4 of 100 licenses assigned |
| [lab1a-10-hr-dynamic-rule.png](../screenshots/lab1a-users-groups/lab1a-10-hr-dynamic-rule.png) | Dynamic membership rule |
| [lab1a-11-hr-dynamic-validate-rules.png](../screenshots/lab1a-users-groups/lab1a-11-hr-dynamic-validate-rules.png) | Validate Rules result |
| [lab1a-12-hr-dynamic-members.png](../screenshots/lab1a-users-groups/lab1a-12-hr-dynamic-members.png) | Jordan HR auto-added |
| [lab1a-13-sam-auto-added.png](../screenshots/lab1a-users-groups/lab1a-13-sam-auto-added.png) | Sam auto-added after department change |
