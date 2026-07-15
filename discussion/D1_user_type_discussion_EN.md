# [Decision] Login Screen and Admin Account Creation Policy by User Type

**Assignee**: PM & Service Design Team · **Status**: Draft
---

## 1. Business Requirements

### Background
We need to decide the login method for general users and exchange office admins in TravelX. To avoid building a system that automatically verifies if an admin is a real staff member, we planned to manually issue admin accounts. However, this divides the login screens and causes inefficiency. If we integrate admins into Google login, we only need one screen. However, this creates a security risk where anyone can become an admin without verification. To prevent this, we would need to build a separate approval system. Therefore, we need to balance reducing development scope and ensuring secure management.

### Operation Policy

| Category | Option 1: Separate Login Screens and Manual Account Creation (Current) | Option 2: Integrated Google Login and User Type Selection (Alternative) |
| :--- | :--- | :--- |
| **Login Screen** | General users and admins use completely different login screen URLs | All users see the same Google login screen |
| **Account Creation** | **General User**: Auto-signup via Google<br>**Admin**: Created manually by PM after contract signing | **Common**: Select user role (General User / Admin) after the first Google login |
| **Staff Verification** | Outside our system (Manual verification of documents by PM team offline) | Inside our system (Auto/manual approval process after user role selection) |

---

## 2. Decision Options Analysis

### Option 1 — Separate Login Screens and Manual Account Creation (Current)
* **User Flow**: 
  * General User: Enter main screen $\rightarrow$ Sign up/in via Google OAuth2 (→ [TravelX_PRD_v1.3_EN_final.docx](../prd/TravelX_PRD_v1.3_EN_final.docx) §24).
  * Admin: Enter partner screen $\rightarrow$ Sign in using ID/PW sent by email (→ [admin_descriptions.md](../prd/feedBack/Admin/admin_descriptions.md) ADM-1).

* **Pros**
  * **Save early development time and scope**: No need to design registration approval screens, branch mapping, or business verification features. This saves MVP launch time.
  * **Security and Safety**: Prevents general users from accessing or hacking the admin screen by separating entry points.
* **Cons**
  * **Manual management effort**: PM team must manually reset and send passwords if admins forget them.

---

### Option 2 — Integrated Google Login and User Type Selection (Alternative)
* **User Flow**: 
  * Login: Both general users and admins use the same Google login button.
  * Role Selection: After login, choose between "Start as Admin" or "Start as General User".

* **Pros**
  * **No password management burden**: Since admins use Google accounts, they can reset passwords via Google. Our team does not need to manage passwords.
  * **Unified login experience**: Only one login screen is needed, improving interface consistency.
* **Cons**
  * **Identity theft and leak risk**: Without verification, anyone can select "Admin" to change currency rates or access reservation data of other branches.
  * **Larger development scope**: To prevent this, we must build verification steps, approval screens, and user-to-branch mapping features, delaying the launch.

---

## 3. Option Comparison

| Items | Option 1 (Current / Recommended) | Option 2 (Alternative) |
| :--- | :--- | :--- |
| **PM Team Workload** | Medium (Needs manual support for password resets) | High (Must review and approve new admin requests constantly) |
| **Initial Dev Burden** | **Very Low** (Simply separate the login pages) | **Very High** (Requires admin registration and review system) |
| **Security & Safety** | **Very Good** (Completely separate access paths prevent data leaks) | Weak (Incorrect clicks or fake approvals can expose branch data) |

---

## 4. Recommendation and Discussion Points
* **Recommendation**: We recommend **[Option 1]** to launch the MVP faster and prevent data leak risks.
* **Questions for Discussion**:
  * How should we securely send initial IDs/PWs to admins after contracts are signed? (e.g., via email or offline meeting)
  * What is the timeline for moving to Option 2 in the future if manual management becomes too heavy?

## Related Documents
- [TravelX_PRD_v1.3_EN_final.docx](../prd/TravelX_PRD_v1.3_EN_final.docx)
- [admin_descriptions.md](../prd/feedBack/Admin/admin_descriptions.md)
