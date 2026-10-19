# Lab 2a — MFA, Self-Service Password Reset & Password Protection

**Date:** October 14, 2026
**Environment:** Microsoft Entra ID tenant with Entra ID P2 trial (Security defaults enabled)
**Admin account used:** lab.admin (Global Administrator)

---

## Objective

Configure authentication methods, onboard a user to MFA with a Temporary Access Pass, enable and test Self-Service Password Reset (SSPR), and enforce a custom banned password list with smart lockout.

---

## What I Built

**Part A — Authentication methods (MFA)**
- Enabled **Microsoft Authenticator** for all users in the Authentication methods policy
- Enabled **Temporary Access Pass (TAP)** for all users
- **Alex IT:** already had Microsoft Authenticator registered (Security defaults prompted registration at first sign-in), then also registered a **passkey** (FIDO2) via My Sign-ins
- **Jordan HR:** started with no usable methods. Issued a **one-time TAP** (1-hour lifetime), signed in as Jordan with the TAP, and registered Microsoft Authenticator

**Part B — Self-Service Password Reset**
- SSPR enabled for **Selected** users, scoped to the **IT-Team** group
- Number of methods required to reset: **1**
- Require users to register when signing in: **Yes**
- Tested as Alex IT at passwordreset.microsoftonline.com using **Microsoft Authenticator**

**Part C — Password protection**
- Smart lockout configured (lockout threshold and duration)
- **Enforce custom list:** Yes
- Custom banned passwords: `Brentwood`, `Coston`, `Pharaoh`
- Tested by having Alex change his password to `Brentwood2026!` at My Sign-ins, then a character-substituted variant (`Brentw00d!2026`) to confirm normalization

---

## Problems & Observations

### 1. SSPR methods page only showed Security questions
**Observation:** The Password reset → Authentication methods page no longer lists Authenticator, SMS, email, etc. — only Security questions, with a banner that they retire in March 2027.
**Explanation:** Microsoft moved method management into the **converged Authentication methods policy**. Methods enabled there (like Microsoft Authenticator from Part A) apply to both MFA and SSPR.
**Result:** Left Security questions unchecked; Authenticator worked for SSPR because it was already enabled in the Authentication methods policy.

### 2. Alex was already registered for MFA
**Observation:** Alex had Microsoft Authenticator registered before this lab.
**Explanation:** **Security defaults** require all users to register for MFA at sign-in, so Alex was prompted during Lab 1b testing.
**Result:** Used Jordan HR instead to demonstrate a clean onboarding with a Temporary Access Pass.

---

## Testing & Results

| Test | User | Result |
|---|---|---|
| Register passkey (FIDO2) | Alex IT | ✅ Passkey created |
| Sign in with one-time TAP, register Authenticator | Jordan HR | ✅ Authenticator registered; TAP shows "One time used" |
| Self-service password reset with Authenticator | Alex IT | ✅ "Your password has been reset" |
| Change password to `Brentwood2026!` | Alex IT | ❌ Rejected — "Choose a password that is harder to guess" |
| Change password to `Brentw00d!2026` (character substitution) | Alex IT | ❌ Rejected — substitution normalized back to banned term |

---

## Key Takeaways

- The **Authentication methods policy** now controls which methods are available for **both MFA and SSPR** (converged policy)
- **Phishing-resistant MFA:** passkeys/FIDO2, Windows Hello for Business, certificate-based authentication. Authenticator push and SMS are **not** phishing-resistant
- **Temporary Access Pass** is a time-limited passcode for onboarding new users or recovering MFA without needing a password; one-time-use passes become unusable after sign-in
- SSPR can be enabled for **None, Selected (one group), or All** users
- **Admins are always enabled for SSPR** and must use **two methods**, regardless of SSPR settings (Administrator Policy)
- **Security questions are retiring** (March 2027) — move users to supported methods
- The **banned password list** supports up to 1,000 terms and **normalizes** common substitutions (e.g., `0→o`, `@→a`, `$→s`), so variations are also blocked
- **Smart lockout** locks out attackers after repeated failures while still letting the real user sign in from familiar locations
- **Password writeback** is required for SSPR to update on-premises AD passwords in hybrid environments

---

## Screenshots

Click a file name to open the screenshot.

| File | Shows |
|---|---|
| [lab2a-01-authenticator-enabled.png](../screenshots/lab2a-mfa-sspr-password-protection/lab2a-01-authenticator-enabled.png) | Microsoft Authenticator enabled in Authentication methods policy |
| [lab2a-02-tap-enabled.png](../screenshots/lab2a-mfa-sspr-password-protection/lab2a-02-tap-enabled.png) | Temporary Access Pass enabled |
| [lab2a-03-alex-security-info-registered.png](../screenshots/lab2a-mfa-sspr-password-protection/lab2a-03-alex-security-info-registered.png) | Alex IT's MFA registration (passkey created) |
| [lab2a-04-jordan-tap-issued.png](../screenshots/lab2a-mfa-sspr-password-protection/lab2a-04-jordan-tap-issued.png) | One-time TAP issued for Jordan HR |
| [lab2a-05-jordan-mfa-registered-via-tap.png](../screenshots/lab2a-mfa-sspr-password-protection/lab2a-05-jordan-mfa-registered-via-tap.png) | Jordan registers Microsoft Authenticator after signing in with TAP |
| [lab2a-06-jordan-auth-methods-after-tap.png](../screenshots/lab2a-mfa-sspr-password-protection/lab2a-06-jordan-auth-methods-after-tap.png) | Admin view: Authenticator usable, TAP "One time used" |
| [lab2a-07-sspr-enabled-it-team.png](../screenshots/lab2a-mfa-sspr-password-protection/lab2a-07-sspr-enabled-it-team.png) | SSPR enabled for IT-Team |
| [lab2a-08-sspr-methods.png](../screenshots/lab2a-mfa-sspr-password-protection/lab2a-08-sspr-methods.png) | SSPR methods page (Security questions retiring) |
| [lab2a-09-sspr-registration-required.png](../screenshots/lab2a-mfa-sspr-password-protection/lab2a-09-sspr-registration-required.png) | Registration required at sign-in |
| [lab2a-10-sspr-reset-success.png](../screenshots/lab2a-mfa-sspr-password-protection/lab2a-10-sspr-reset-success.png) | Alex's password reset succeeds |
| [lab2a-11-password-protection-settings.png](../screenshots/lab2a-mfa-sspr-password-protection/lab2a-11-password-protection-settings.png) | Smart lockout and custom banned password list |
| [lab2a-12-banned-password-rejected.png](../screenshots/lab2a-mfa-sspr-password-protection/lab2a-12-banned-password-rejected.png) | Banned password rejected |
| [lab2a-13-banned-password-fuzzy-match.png](../screenshots/lab2a-mfa-sspr-password-protection/lab2a-13-banned-password-fuzzy-match.png) | Character-substituted variant also rejected |
