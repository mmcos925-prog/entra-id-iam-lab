# Lab 2b — Conditional Access & Break-Glass Accounts

**Date:** October 14, 2026
**Environment:** Microsoft Entra ID tenant with Entra ID P2 trial
**Admin account used:** lab.admin (Global Administrator)

---

## Objective

Replace Security defaults with a Conditional Access policy that requires MFA for all users. Protect the tenant from lockout with an excluded break-glass account. Validate the policy with Report-only mode and the What If tool before enforcing it, then confirm enforcement in the sign-in logs.

---

## What I Built

**Part A — Pre-checks and break-glass account**
- Confirmed lab.admin was **MFA capable** (3 methods registered) before touching any MFA settings
- Created **breakglass01** (Emergency Access 01) as a cloud-only account with a long random password stored in a password manager
- Assigned **Global Administrator** to breakglass01
- Signed in as breakglass01 and registered a **passkey** (phishing-resistant MFA)
- Created the security group **CA-Exclude-BreakGlass** (Assigned) with breakglass01 as its only member

**Part B — Disable Security defaults**
- Entra ID → Overview → Properties → Manage security defaults → **Disabled**
- Reason: "My organization is planning to use Conditional Access"

**Part C — Conditional Access policy (Report-only)**
- Name: **CA001-Require-MFA-All-Users**
- Users: **All users**, excluding **CA-Exclude-BreakGlass**
- Target resources: **All resources** (formerly "All cloud apps")
- Grant: **Require multifactor authentication**
- Enable policy: **Report-only**

**Part D — What If testing**
- **Alex IT:** CA001 listed under "Policies that will apply" with Require MFA, state Report-only
- **Emergency Access 01:** 0 policies found. CA001 listed under "Policies that will not apply" because the user is excluded

**Part E — Enforcement and verification**
- Switched CA001 from Report-only to **On**
- Signed in as **Sam Finance** in incognito. Sam had no MFA method, so CA001 **forced MFA registration** (Microsoft Authenticator) before granting access
- Verified in **Sign-in logs → Conditional Access tab** that CA001 was applied with result **Success**
- Checked the **Authentication Details** tab to confirm password + Authenticator were both used

---

## Problems & Observations

### 1. "Don't lock yourself out!" warning
**Observation:** When saving the policy, Entra warned that the current admin (lab.admin) would be affected and offered to exclude it.
**Decision:** Chose **"I understand that my account will be impacted by this policy. Proceed anyway."**
**Reasoning:** lab.admin already had MFA registered, and breakglass01 is the excluded emergency account. Excluding everyday admin accounts from MFA would leave the most privileged, most-targeted accounts unprotected. Only break-glass accounts should be excluded.

### 2. Sam Finance's password was rejected
**Problem:** Sign-in as Sam failed with "Your password is incorrect."
**Cause:** Sam's password had been reset during Lab 1b (while signed in as lab.admin), so the original password was no longer valid.
**Fix:** Reset Sam's password as lab.admin, signed in with the temporary password, and set a new one stored in the password manager. Avoided repeated guessing because **smart lockout** (configured in Lab 2a) would lock the account.

### 3. What If screenshot missing the selected user
**Problem:** The first What If result screenshot didn't show which user was being tested.
**Fix:** Zoomed out the browser (Cmd + −) so the identity selection and the evaluation result appear in the same screenshot.

---

## Testing & Results

| Test | User | Result |
|---|---|---|
| What If (Report-only) | Alex IT | ✅ CA001 will apply — Require MFA |
| What If (Report-only) | Emergency Access 01 | ✅ CA001 will not apply — excluded |
| Live sign-in, policy On | Sam Finance (no MFA registered) | ✅ Forced to register Microsoft Authenticator |
| Sign-in log review | Sam Finance | ✅ CA001 applied — Success (password + Authenticator) |

---

## Key Takeaways

- **Security defaults and Conditional Access can't be used together.** Disable Security defaults before enforcing CA policies.
- Conditional Access requires **Entra ID P1** (risk-based conditions need **P2**)
- CA policies are **if-then statements**: assignments (users, target resources, conditions) → access controls (grant/block, session)
- **Report-only** mode evaluates and logs a policy without enforcing it. Always start new policies here.
- The **What If** tool simulates which policies apply to a given user, app and condition without a real sign-in
- **Break-glass accounts:** at least **two**, cloud-only, Global Administrator, excluded from CA via a group, strong phishing-resistant MFA (passkey/FIDO2), credentials stored securely, and sign-ins **monitored with alerts**
- A user with no MFA method who hits a Require-MFA policy is **forced to register** at sign-in
- **Sign-in logs → Conditional Access tab** shows each policy's result (Success, Failure, Not applied, Report-only) and is the first place to troubleshoot access issues
- Sign-in logs can take **several minutes** to appear

---

## Screenshots

Click a file name to open the screenshot.

| File | Shows |
|---|---|
| [lab2b-00-labadmin-mfa-capable.png](../screenshots/lab2b-conditional-access/lab2b-00-labadmin-mfa-capable.png) | lab.admin is MFA capable before changes |
| [lab2b-01-breakglass-global-admin.png](../screenshots/lab2b-conditional-access/lab2b-01-breakglass-global-admin.png) | breakglass01 created with Global Administrator |
| [lab2b-01b-breakglass-passkey-registered.png](../screenshots/lab2b-conditional-access/lab2b-01b-breakglass-passkey-registered.png) | Passkey registered for breakglass01 |
| [lab2b-02-ca-exclude-group.png](../screenshots/lab2b-conditional-access/lab2b-02-ca-exclude-group.png) | CA-Exclude-BreakGlass group with breakglass01 |
| [lab2b-03-security-defaults-disabled.png](../screenshots/lab2b-conditional-access/lab2b-03-security-defaults-disabled.png) | Security defaults disabled |
| [lab2b-04-ca001-users-exclude.png](../screenshots/lab2b-conditional-access/lab2b-04-ca001-users-exclude.png) | CA001 users: All users, break-glass group excluded |
| [lab2b-05-ca001-grant-mfa.png](../screenshots/lab2b-conditional-access/lab2b-05-ca001-grant-mfa.png) | CA001 grant: Require multifactor authentication |
| [lab2b-05a-lockout-warning.png](../screenshots/lab2b-conditional-access/lab2b-05a-lockout-warning.png) | "Don't lock yourself out!" warning |
| [lab2b-06-ca001-report-only.png](../screenshots/lab2b-conditional-access/lab2b-06-ca001-report-only.png) | CA001 created in Report-only |
| [lab2b-07-whatif-alex-applies.png](../screenshots/lab2b-conditional-access/lab2b-07-whatif-alex-applies.png) | What If: CA001 applies to Alex IT |
| [lab2b-08-whatif-breakglass-excluded.png](../screenshots/lab2b-conditional-access/lab2b-08-whatif-breakglass-excluded.png) | What If: 0 policies apply to Emergency Access 01 |
| [lab2b-08b-whatif-not-apply-reason.png](../screenshots/lab2b-conditional-access/lab2b-08b-whatif-not-apply-reason.png) | What If: CA001 not applied — user excluded |
| [lab2b-09-ca001-on.png](../screenshots/lab2b-conditional-access/lab2b-09-ca001-on.png) | CA001 switched to On |
| [lab2b-10-sam-mfa-registration-forced.png](../screenshots/lab2b-conditional-access/lab2b-10-sam-mfa-registration-forced.png) | Sam forced to install Microsoft Authenticator |
| [lab2b-10b-sam-mfa-registered.png](../screenshots/lab2b-conditional-access/lab2b-10b-sam-mfa-registered.png) | Sam's MFA registration complete |
| [lab2b-11-signin-log-sam-entry.png](../screenshots/lab2b-conditional-access/lab2b-11-signin-log-sam-entry.png) | Sam's sign-in in the Sign-in logs |
| [lab2b-12-signin-log-ca001.png](../screenshots/lab2b-conditional-access/lab2b-12-signin-log-ca001.png) | Conditional Access tab: CA001 Success |
| [lab2b-13-signin-log-auth-details.png](../screenshots/lab2b-conditional-access/lab2b-13-signin-log-auth-details.png) | Authentication Details: password + Authenticator |
