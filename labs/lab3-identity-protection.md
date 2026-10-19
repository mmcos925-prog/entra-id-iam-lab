# Lab 3 — Identity Protection & Risk-Based Conditional Access

**Date:** October 15–16, 2026
**Environment:** Microsoft Entra ID tenant with Entra ID P2 trial
**Admin account used:** lab.admin (Global Administrator)

---

## Objective

Build risk-based Conditional Access policies with Microsoft Entra ID Protection, simulate a real risky sign-in, mark a user compromised through Microsoft Graph, and confirm that both users fixed their own risk without an admin stepping in.

---

## What I Built

**Part A — Identity Protection overview**
- Reviewed the ID Protection dashboard and the Risky users, Risky sign-ins, and Risk detections reports (baseline: no risky users)

**Part B — Sign-in risk policy**
- Name: **CA002-Require-MFA-Medium-High-Signin-Risk**
- Users: All users, excluding **CA-Exclude-BreakGlass**
- Target resources: All resources
- Condition: **Sign-in risk = High, Medium**
- Grant: **Require multifactor authentication**
- Session: **Sign-in frequency = Every time**
- State: **On**

**Part C — User risk policy**
- Name: **CA003-Require-PasswordChange-High-User-Risk**
- Users: All users, excluding **CA-Exclude-BreakGlass**
- Target resources: All resources
- Condition: **User risk = High**
- Grant: **Require risk remediation** (the newer grant control; it replaces manually combining "Require MFA" + "Require password change")
- Session: **Sign-in frequency = Every time** (set automatically with risk remediation)
- State: **On**
- With Require risk remediation, Entra handles the whole remediation flow itself: the user completes MFA and a secure password change, and their existing sessions are revoked

**Part D — Simulated risky sign-in (Tor Browser)**
- Signed in as **Jordan HR** to My Apps through the **Tor Browser**, which sends traffic out through anonymous exit nodes
- Jordan was prompted for Microsoft Authenticator (number matching) and approved it
- Identity Protection logged **11 "Anonymous IP address" detections** (High and Medium), from two Tor exit nodes:
  - 104.167.242.118 — Willis, Texas, US (IPv4)
  - 2001:67c:e60:c0c:192:42:116:… — Camperduin, Noord-Holland, Netherlands (IPv6)
- Detections were **Real-time**, so CA002 acted during the sign-in itself
- Sign-in log Conditional Access tab: **CA001 Success, CA002 Success, CA003 Not Applied**
- Every detection showed **Remediated** — risk detail: *"User passed multifactor authentication"*

**Part E — Confirm user compromised (Microsoft Graph)**
- Entra only offers "Confirm compromised" for users who already have risk, so I used **Graph Explorer** signed in as lab.admin:
  ```http
  POST https://graph.microsoft.com/v1.0/identityProtection/riskyUsers/confirmCompromised
  Content-Type: application/json

  {
    "userIds": ["7150f6e3-383e-4759-bba5-ebb2acd63a83"]
  }
  ```
- Consented to the **IdentityRiskyUser.ReadWrite.All** permission
- Response: **204 No Content** (success)
- Sam Finance moved to **Risk level: High / Risk state: Confirmed compromised**
- Signed in as Sam in incognito → **"Additional verification required"** → MFA → **"Update your password"**
- Sam ended at **Risk state: Remediated, Risk level: None, Identity Risk Score: 0**

**Sam's risk timeline (Risky User Details):**

| Time (Oct 15) | Event | Actor |
|---|---|---|
| 2:07 PM | Admin confirmed user compromised | Lab Admin |
| 2:11 PM | Microsoft revoked sessions for user | Sam Finance |
| 2:14 PM | User performed secured password change | Sam Finance |

---

## Problems & Observations

### 1. Legacy Identity Protection policies are retired
**Observation:** The old user risk and sign-in risk policies inside Identity Protection have been retired.
**Result:** Built both risk policies in **Conditional Access** instead, which is Microsoft's current recommended approach.

### 2. Risk reports lagged behind the sign-in log
**Observation:** About 20 minutes after the Tor sign-in, the Risk detections and Risky sign-ins reports were still empty.
**Fix:** Checked Jordan's **sign-in log → Conditional Access tab** first. CA002 showing **Success** proved the sign-in risk had been detected in real time. The reports filled in a few hours later.
**Lesson:** The sign-in log is the fastest place to confirm whether a policy fired. Reports can lag behind.

### 3. No portal option to confirm Sam compromised
**Observation:** Sam had no existing risk, so he didn't appear in Risky users and there was no "Confirm compromised" button.
**Fix:** Used the **Microsoft Graph API** (`confirmCompromised`) from Graph Explorer, using Sam's Object ID.
**Lesson:** Graph can do things the portal UI doesn't, and the same call can be scripted for incident response.

### 4. CA003 didn't appear cleanly in Sam's sign-in log
**Observation:** None of Sam's sign-in entries listed CA003 as Success. The remediation flow was split across several entries.
**Fix:** Used the **Risky User Details → Timeline** view, which shows the full chain: admin confirmation → session revocation → secured password change.

### 5. Generic password-change message
**Observation:** The "Update your password" screen said *"this is the first time you are signing in, or your password has expired"* even though CA003 triggered it.
**Lesson:** Users see Microsoft's generic wording. Admins confirm the real cause in the risk timeline and logs.

---

## Testing & Results

| Test | User | Result |
|---|---|---|
| Sign in through Tor (anonymous IP) | Jordan HR | ✅ 11 Anonymous IP detections, CA002 applied, MFA required |
| Risk self-remediation via MFA | Jordan HR | ✅ All detections Remediated; user risk Dismissed |
| Confirm compromised via Graph API | Sam Finance | ✅ 204 No Content; user risk High / Confirmed compromised |
| Sign in with High user risk | Sam Finance | ✅ CA003 required MFA, then secure password change |
| Post-remediation state | Sam Finance | ✅ Remediated, No risk, Identity Risk Score 0, sessions revoked |

---

## Key Takeaways

- **Sign-in risk** = is *this sign-in* suspicious (anonymous IP, unfamiliar location, atypical travel)? Remediate with **MFA**.
- **User risk** = is *the account* likely compromised (leaked credentials, admin confirmation)? Remediate with **MFA + secure password change**.
- The **Require risk remediation** grant control is the modern way to remediate user risk: one control that triggers MFA, a secure password change, and session revocation. The older approach combined "Require MFA" + "Require password change" by hand, and the exam may show either one.
- Risk-based Conditional Access requires **Entra ID P2**.
- **Real-time** detections can be acted on during sign-in. **Offline** detections are calculated afterward and feed user risk.
- **Self-remediation** keeps risky users out of the helpdesk queue, because users clear their own risk by passing MFA or changing their password.
- **Risk states:**
  - **At risk** — open risk
  - **Remediated** — the user fixed it (MFA or secure password change)
  - **Dismissed** — the risk was closed without a remediation action
  - **Confirmed compromised / Confirmed safe** — an admin made the call manually
- When a compromised user's risk is remediated, **sessions are revoked**, so a stolen token stops working along with the old password.
- Always exclude **break-glass accounts** from risk policies too.
- **Microsoft Graph** (`identityProtection/riskyUsers`) supports confirmCompromised, dismiss, and listing risky users, which is useful for automation and SOAR playbooks.
- Use **Sign-in frequency: Every time** on risk policies so a fresh check happens on each risky sign-in instead of reusing an old session.

---

## Screenshots

Click a file name to open the screenshot.

| File | Shows |
|---|---|
| [lab3-01-idp-dashboard.png](../screenshots/lab3-identity-protection/lab3-01-idp-dashboard.png) | ID Protection dashboard (baseline) |
| [lab3-01b-risky-users-empty.png](../screenshots/lab3-identity-protection/lab3-01b-risky-users-empty.png) | Risky users report before testing |
| [lab3-02-ca002-signin-risk-condition.png](../screenshots/lab3-identity-protection/lab3-02-ca002-signin-risk-condition.png) | CA002 sign-in risk condition (High, Medium) |
| [lab3-03-ca002-grant-session.png](../screenshots/lab3-identity-protection/lab3-03-ca002-grant-session.png) | CA002 grant (MFA) and session (sign-in frequency every time) |
| [lab3-04-ca003-user-risk-condition.png](../screenshots/lab3-identity-protection/lab3-04-ca003-user-risk-condition.png) | CA003 user risk condition (High) |
| [lab3-05-ca003-grant-password-change.png](../screenshots/lab3-identity-protection/lab3-05-ca003-grant-password-change.png) | CA003 grant: Require risk remediation |
| [lab3-06-tor-signin-mfa-prompt.png](../screenshots/lab3-identity-protection/lab3-06-tor-signin-mfa-prompt.png) | Jordan's MFA number-match prompt in the Tor Browser |
| [lab3-07-risk-detection-anonymous-ip.png](../screenshots/lab3-identity-protection/lab3-07-risk-detection-anonymous-ip.png) | 11 Anonymous IP address detections, all Remediated |
| [lab3-08-risky-signin-jordan.png](../screenshots/lab3-identity-protection/lab3-08-risky-signin-jordan.png) | Risky sign-in details: High, Real-time, remediated by MFA |
| [lab3-09-signin-log-ca002-applied.png](../screenshots/lab3-identity-protection/lab3-09-signin-log-ca002-applied.png) | Sign-in log: CA001 + CA002 Success, CA003 Not Applied |
| [lab3-10-graph-confirm-compromised.png](../screenshots/lab3-identity-protection/lab3-10-graph-confirm-compromised.png) | Graph Explorer confirmCompromised call — 204 No Content |
| [lab3-11-sam-confirmed-compromised.png](../screenshots/lab3-identity-protection/lab3-11-sam-confirmed-compromised.png) | Sam: High risk, Confirmed compromised |
| [lab3-12a-sam-additional-verification.png](../screenshots/lab3-identity-protection/lab3-12a-sam-additional-verification.png) | "Additional verification required" at Sam's sign-in |
| [lab3-12b-sam-forced-password-change.png](../screenshots/lab3-identity-protection/lab3-12b-sam-forced-password-change.png) | CA003 forcing "Update your password" |
| [lab3-13-sam-risk-remediated.png](../screenshots/lab3-identity-protection/lab3-13-sam-risk-remediated.png) | Risky users: Sam Remediated, Jordan Dismissed |
| [lab3-14-sam-risk-timeline.png](../screenshots/lab3-identity-protection/lab3-14-sam-risk-timeline.png) | Sam's timeline: confirmed → sessions revoked → password changed |
