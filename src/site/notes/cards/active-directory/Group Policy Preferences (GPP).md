---
{"dg-publish":true,"permalink":"/cards/active-directory/group-policy-preferences-gpp/"}
---

~ [[cards/active-directory/windows active directory\|Active Directory]] | ~ [[cards/red-team/Credential Harvesting\|TA0006 - Credential Access]]
## How it Works
---
**Historical Context:**

In large Windows environments, every computer has a **local Administrator account**.

**Problems:**

- Same password everywhere = **very insecure**
- Different password everywhere = **hard to manage**

**GPP Solution (Pre-2015):**

Microsoft introduced **Group Policy Preferences (GPP)** to solve this:

- Admins could **centrally set or change local admin passwords**
- Changes would automatically apply to all domain-joined machines
- When admin configures local account via GPP, Windows writes settings (including password, encrypted) to **XML files** like `Groups.xml` or `Services.xml`
- These XML files placed in **SYSVOL**, shared folder replicated across all domain controllers so policies apply to all domain-joined machines

**Vulnerability:**

The encryption key for GPP passwords was published by Microsoft, allowing any domain user to decrypt passwords from SYSVOL XML files.

**LAPS Solution (2015+):**

Microsoft removed storing encrypted password in SYSVOL folder and introduced Local Administrator Password Solution (LAPS), offering much more secure approach to remotely managing local administrator password.

**LAPS Implementation:**

- Two new attributes added to computer objects:
  - **ms-mcs-AdmPwd** - Contains plaintext password of local administrator
  - **ms-mcs-AdmPwdExpirationTime** - Contains expiration time to reset password
- LAPS uses `admpwd.dll` to change local administrator password and update value of `ms-mcs-AdmPwd`
- Passwords stored in Active Directory (not SYSVOL)
- Only accounts with specific permissions can read passwords
- Passwords automatically rotate based on policy