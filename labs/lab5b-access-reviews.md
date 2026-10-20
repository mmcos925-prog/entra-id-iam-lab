# Lab 5b — Access Reviews

**Date:** October 20, 2026
**Environment:** Microsoft Entra ID tenant with Entra ID P2 trial
**Admin account used:** lab.admin (Global Administrator)

---

## Objective

Run an access review of a group's membership, have a designated reviewer approve or deny each member with a justification, and confirm Entra automatically removes denied users.

---

## What I Built

**Part A — Group to review**
- Created the security group **Finance-Share-Access** (Assigned) with **Sam Finance** and **Jordan HR** as members

**Part B — Access review: Q4 Finance Share Access Review**
- **What to review:** Teams + Groups → **Finance-Share-Access**, scope **All users** (shown as *Everyone*)
- **Reviewer:** selected user → **Alex IT**
- **Duration:** 3 days (10/20/2026 – 10/23/2026), **one time**
- **Settings:**
  - **Auto apply results to resource:** enabled
  - **If reviewers don't respond:** remove access
  - **Justification required:** enabled
  - **Decision helpers** enabled where available

**Part C — Reviewer decisions**
- Alex completed the review at **myaccess.microsoft.com → Access reviews**:
  - **Jordan HR → Approve** ("Still needs access")
  - **Sam Finance → Deny** ("Moved teams, no longer needs finance share")

**Part D — Results**
- The review completed: **2 users reviewed, 1 approved, 1 denied, 0 not reviewed**
- Results were applied to the group. **Finance-Share-Access** now has **1 member: Jordan HR**. Sam was removed automatically.

---

## Problems & Observations

### Stop button was grayed out
**Observation:** On the review overview page, **Stop** wasn't clickable.
**Explanation:** Stop is only available while a review instance is **In progress**. It's grayed out while the review is still initializing, before its start date, or after it completes. The review overview showed **Review status: Complete** with results applied, so Stop and Apply were no longer needed.
**Lesson:** Check the review status first. Auto-apply removes denied users when the review ends, without any manual action.

---

## Testing & Results

| Check | Result |
|---|---|
| Review delivered to reviewer | ✅ Alex saw the review in My Access |
| Decisions with justification | ✅ Jordan approved, Sam denied |
| Review status | ✅ Complete: 1 approved, 1 denied, 0 not reviewed |
| Auto-apply | ✅ Sam removed from Finance-Share-Access; Jordan kept |

---

## Key Takeaways

- **Access reviews** recertify who should still have access, preventing **privilege creep** as people change roles
- **What can be reviewed:** group and Teams membership, application assignments, **access package assignments**, and **privileged roles** (Entra and Azure roles through PIM)
- **Reviewer options:** group owners, selected users or groups, **users review their own access (self-review)**, and **managers**
- **Auto apply results** removes denied users automatically when the review ends
- **If reviewers don't respond:** No change, **Remove access**, Approve access, or Take recommendations. *Remove access* is the secure default.
- **Decision helpers / recommendations** flag users who haven't signed in recently (e.g., 30 days) so reviewers can deny stale access
- **Recurrence** (weekly, monthly, quarterly, semi-annually, annually) supports ongoing compliance (SOX, HIPAA, FedRAMP)
- Reviews can target **guest users only**, a common way to clean up B2B access
- **Licensing:** access reviews require **Entra ID P2** or **Entra ID Governance**

---

## Screenshots

Click a file name to open the screenshot.

| File | Shows |
|---|---|
| [lab5b-01-group-members.png](../screenshots/lab5b-access-reviews/lab5b-01-group-members.png) | Finance-Share-Access with Sam and Jordan |
| [lab5b-02-review-scope-reviewers.png](../screenshots/lab5b-access-reviews/lab5b-02-review-scope-reviewers.png) | Review scope and Alex as reviewer |
| [lab5b-03-review-settings.png](../screenshots/lab5b-access-reviews/lab5b-03-review-settings.png) | Auto-apply, remove access if no response, justification |
| [lab5b-04-alex-review-decisions.png](../screenshots/lab5b-access-reviews/lab5b-04-alex-review-decisions.png) | Alex approves Jordan, denies Sam |
| [lab5b-05-review-results.png](../screenshots/lab5b-access-reviews/lab5b-05-review-results.png) | Review complete: 1 approved, 1 denied |
| [lab5b-06-sam-removed.png](../screenshots/lab5b-access-reviews/lab5b-06-sam-removed.png) | Group now contains only Jordan HR |
