# Lab 4 — Application Integration: SAML SSO, App Registrations & Consent

**Date:** October 19, 2026
**Environment:** Microsoft Entra ID tenant with Entra ID P2 trial
**Admin account used:** lab.admin (Global Administrator)

---

## Objective

Integrate an enterprise application with SAML single sign-on, restrict access with user assignment, register an application and compare delegated and application permissions, and lock down user consent with an admin consent workflow.

---

## What I Built

**Part A — Enterprise app with SAML SSO**
- Added **Microsoft Entra SAML Toolkit** from the Entra app gallery
- Assigned **Alex IT** to the app
- Configured SAML-based single sign-on:
  - Identifier (Entity ID): `https://samltoolkit.azurewebsites.net`
  - Reply URL (ACS): `https://samltoolkit.azurewebsites.net/SAML/Consume/22731`
  - Sign on URL: `https://samltoolkit.azurewebsites.net/SAML/Login/22731`
- Downloaded the **Certificate (Raw)** and copied the **Login URL**, **Microsoft Entra Identifier**, and **Logout URL** into the toolkit's SAML configuration
- Registered a toolkit account matching Alex's UPN, since the service provider matches users by email

**Part B — Require user assignment**
- Set **Assignment required? = Yes** and **Visible to users? = Yes**
- Verified an unassigned user (Sam Finance) could not see or sign in to the app

**Part C — App registration and API permissions**
- Registered **SC300-Lab-GraphReader** as a single-tenant app with no redirect URI
- Default delegated permission: **User.Read**
- Added the application permission **User.Read.All**, which showed *"Not granted for Default Directory"* until I granted **admin consent**
- Confirmed the registration automatically created a matching **enterprise application** (service principal) showing the granted permissions

**Part D — Consent settings and the admin consent workflow**
- **User consent:** *Allow user consent for apps from verified publishers, for selected permissions*
- **Permission classifications (low impact):** `openid`, `profile`, `email`, `offline_access`, `User.Read`
- **Admin consent workflow:** enabled, with lab.admin as reviewer, email notifications and reminders on, and requests expiring after 30 days
- **Test:** Sam tried to consent to a higher-privilege permission in Graph Explorer and was blocked with *"Need admin approval"*
- Reviewed the request as lab.admin. The consent screen showed it would grant **Read directory data** tenant-wide (*"for all users in your organization"*), so I **denied** it for lack of a business justification

---

## Problems & Observations

### 1. SAML Toolkit returned to its home page instead of signing Alex in
**Observation:** After launching the app from My Apps as Alex, the toolkit redirected back to its home page without signing in, in both Chrome and Safari private windows.
**Troubleshooting:**
- Confirmed the Entra configuration matched the toolkit's configuration exactly (Entity ID, ACS URL with config ID 22731, Sign on URL, tenant Login URL, Entra Identifier, Raw certificate)
- Registered the toolkit account with Alex's UPN, after first misreading the requirement
- Checked the app's **sign-in logs**: Alex's SAML sign-ins showed **Status: Success**
**Conclusion:** Entra authenticated the user, enforced Conditional Access, and issued a signed SAML token. The failure was on the **service provider side**: the alpha-stage test app didn't consume the assertion.
**Lesson:** For SSO problems, check the IdP sign-in log first. A **Success** there points the investigation at the application. A **Failure** with an `AADSTS` code points at the Entra configuration.

### 2. MFA wasn't prompted again for the SAML app
**Observation:** The sign-in log showed *"MFA requirement satisfied by claim in the token."*
**Explanation:** CA001 requires MFA for all resources, but Alex had already completed MFA when signing in to My Apps. The MFA claim in his session satisfied the policy, so SSO didn't trigger a second prompt.

### 3. Admin consent workflow took time to apply
**Observation:** Sam's first attempt showed *"Need admin approval"* with no option to request approval.
**Fix:** Confirmed the admin consent settings were saved, waited for the settings to apply, and retried in a new session. The request then reached lab.admin.

---

## Testing & Results

| Test | User | Result |
|---|---|---|
| SAML sign-in to toolkit (IdP side) | Alex IT (assigned) | ✅ Entra sign-in log: Success, SAML token issued |
| SAML sign-in to toolkit (SP side) | Alex IT | ⚠️ Toolkit didn't complete sign-in (service provider issue) |
| App visibility in My Apps | Sam Finance (not assigned) | ✅ App not listed |
| Direct sign-in to app | Sam Finance (not assigned) | ❌ Blocked — **AADSTS50105** (user not assigned) |
| Application permission before consent | — | ✅ "Not granted for Default Directory" |
| Admin consent grant | lab.admin | ✅ Permissions show as granted |
| User consent to high-privilege permission | Sam Finance | ❌ Blocked — "Need admin approval" |
| Admin consent request review | lab.admin | ✅ Request received and **denied** |

---

## Key Takeaways

- **SAML roles:** Entra is the **identity provider (IdP)** that signs the token; the app is the **service provider (SP)** that trusts it. Key values: Entity ID (identifier), Reply URL (ACS), Sign on URL, and the signing certificate.
- **SP-initiated vs IdP-initiated:** the Sign on URL starts sign-in at the app (SP-initiated). Launching from My Apps without one is IdP-initiated.
- **Assignment required = Yes** restricts an app to assigned users and groups. Unassigned users get **AADSTS50105**. Assigning **groups** requires P1 or higher.
- **App registration vs enterprise app:** the app registration is the app's global definition (application object). The enterprise app is its **service principal** in a tenant, where assignment, SSO, and consent are managed.
- **Delegated permissions** act *as the signed-in user* and are limited to what that user can access. **Application permissions** act *as the app with no user*, apply **tenant-wide**, and **always require admin consent**.
- **User consent settings** reduce the risk of **illicit consent grant** attacks, where users are tricked into giving a malicious app access to their data.
- **Permission classifications** define which low-impact permissions users may consent to themselves.
- The **admin consent workflow** lets users request access instead of hitting a dead end, while keeping approval with designated reviewers.
- Admin consent is **tenant-wide**: one approval applies to every user, so reviewers should require a business justification.

---

## Screenshots

Click a file name to open the screenshot.

| File | Shows |
|---|---|
| [lab4-01-saml-basic-config.png](../screenshots/lab4-app-integration/lab4-01-saml-basic-config.png) | Initial Basic SAML Configuration |
| [lab4-01b-saml-config-updated.png](../screenshots/lab4-app-integration/lab4-01b-saml-config-updated.png) | Updated Reply URL and Sign on URL with toolkit config ID |
| [lab4-01c-toolkit-connection.png](../screenshots/lab4-app-integration/lab4-01c-toolkit-connection.png) | Toolkit SP-initiated login page |
| [lab4-03-saml-signin-log-success.png](../screenshots/lab4-app-integration/lab4-03-saml-signin-log-success.png) | Alex's SAML sign-in: Success, MFA satisfied by token claim |
| [lab4-04-assignment-required.png](../screenshots/lab4-app-integration/lab4-04-assignment-required.png) | Assignment required set to Yes |
| [lab4-05-sam-no-app.png](../screenshots/lab4-app-integration/lab4-05-sam-no-app.png) | Toolkit not visible in Sam's My Apps |
| [lab4-06-sam-aadsts50105.png](../screenshots/lab4-app-integration/lab4-06-sam-aadsts50105.png) | Sam blocked with AADSTS50105 |
| [lab4-07-app-registration-overview.png](../screenshots/lab4-app-integration/lab4-07-app-registration-overview.png) | SC300-Lab-GraphReader app registration |
| [lab4-08-permissions-not-granted.png](../screenshots/lab4-app-integration/lab4-08-permissions-not-granted.png) | User.Read.All not yet granted |
| [lab4-09-admin-consent-granted.png](../screenshots/lab4-app-integration/lab4-09-admin-consent-granted.png) | Admin consent granted |
| [lab4-10-enterprise-app-permissions.png](../screenshots/lab4-app-integration/lab4-10-enterprise-app-permissions.png) | Service principal showing granted permissions |
| [lab4-11-user-consent-settings.png](../screenshots/lab4-app-integration/lab4-11-user-consent-settings.png) | User consent limited to verified publishers and selected permissions |
| [lab4-12-permission-classifications.png](../screenshots/lab4-app-integration/lab4-12-permission-classifications.png) | Low-impact permission classifications |
| [lab4-13-admin-consent-workflow.png](../screenshots/lab4-app-integration/lab4-13-admin-consent-workflow.png) | Admin consent workflow settings |
| [lab4-14-sam-need-admin-approval.png](../screenshots/lab4-app-integration/lab4-14-sam-need-admin-approval.png) | Sam blocked from consenting |
| [lab4-15-admin-consent-review.png](../screenshots/lab4-app-integration/lab4-15-admin-consent-review.png) | Admin review showing tenant-wide consent scope |
| [lab4-16-request-denied.png](../screenshots/lab4-app-integration/lab4-16-request-denied.png) | Consent request denied |
