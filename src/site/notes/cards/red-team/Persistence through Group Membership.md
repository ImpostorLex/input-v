---
{"dg-publish":true,"permalink":"/cards/red-team/persistence-through-group-membership/"}
---

~ [[Persisting Active Directory\|Persisting Active Directory]]
## Summary
---
**What it is:** Achieving persistent privileged access by adding compromised accounts to nested Active Directory groups that indirectly grant Domain Admin privileges, exploiting monitoring gaps where security teams primarily track changes to top-level privileged groups while missing modifications to deeply nested subgroups.

**Scope:** This note covers the concept of nested groups in Active Directory, how group nesting creates visibility and monitoring gaps, the technique of creating a chain of nested groups to place a low-privileged user indirectly into Domain Admins, and why this persistence method evades standard security monitoring focused on direct privileged group membership changes.

**Prerequisites:** 

- [[Active Directory Groups\|Active Directory Groups]]
- [[cards/active-directory/Group Policy Objects\|Group Policy Objects]]
- Domain Admin or equivalent privileges to create and modify security groups
- Access to Domain Controller or system with AD administrative tools
- Knowledge of target domain's OU structure
- Low-privileged target account for persistence

**Risks & Limitations:**

- Requires Domain Admin privileges to add groups to Domain Admins
- Nested groups create audit trail across multiple Event IDs
- Deeper nesting may trigger review during security audits
- Group naming conventions must blend with environment to avoid detection
- Transitive group membership can be enumerated with proper tools
- Removal requires identifying and deleting entire nested chain
## How It Works (2-4 lines)
---
Active Directory organizations use group nesting to simplify permission management, where parent groups contain subgroups that inherit permissions while having specific additional rights. This nesting reduces visibility into actual access since checking a parent group's membership only reveals immediate subgroups, not nested sub-subgroups, requiring recursive enumeration to understand full access. Security teams typically monitor direct additions to privileged groups like Domain Admins but often miss changes to nested subgroups due to organizational separation between AD management and InfoSec teams. Attackers exploit this monitoring gap by creating chains of nested groups where a low-privileged user is placed at the bottom level, and the top-level group is added to Domain Admins - AD's transitive permission system grants the low-privileged user Domain Admin access while direct membership checks show only the benign-looking top-level group.

Read first: [[cards/active-directory/Nested Group Concept\|Nested Group Concept]]

**Attacker Exploitation:**

As an attacker, you can exploit this by targeting unmonitored **subgroups** instead of directly adding yourself to **Domain Admins**.
## Steps (Hands-On)
---

**Legend:**

- `{LOW_PRIV_USER}` = your low-privileged username
- `{USERNAME}` = username prefix for group naming (use environment-appropriate names)
- `{DOMAIN}` = ZA.TRYHACKME.LOC
- `{DC_FQDN}` = thmdc.za.tryhackme.loc
- `{IT_OU}` = OU=IT,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC
- `{SALES_OU}` = OU=SALES,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC
- `{NESTGROUP1}` = {USERNAME}_nestgroup1
- `{NESTGROUP2}` = {USERNAME}_nestgroup2
- `{NESTGROUP3}` = {USERNAME}_nestgroup3
- `{NESTGROUP4}` = {USERNAME}_nestgroup4
- `{NESTGROUP5}` = {USERNAME}_nestgroup5

**Context:** You have Domain Admin credentials and want to establish persistent privileged access for a low-privileged account by creating a chain of nested groups that indirectly grants Domain Admin privileges while evading direct membership monitoring.

**Attack Chain:**

```
Domain Admins <- Group 5 <- Group 4 <- Group 3 <- Group 2 <- Group 1 <- Low Privilege User
```

**How This Works:**

- **Initial Check:** If security team only checks direct members of Domain Admins, they will see Group 5 (which looks benign or non-standard), but they will **NOT** see the low-privileged user.

- **Effective Access:** Because AD permissions are transitive, the low-privileged user which is member of Group 1 which is member of Group 2, and so on, until Group 5 grants its permissions to Domain Admins.

**Prerequisites:**

- Access to Active Directory domain controller (or system with appropriate AD administrative tools)
- Credentials for account with privileges to create and modify security groups (e.g., Administrator.ZA)
- Target low-privileged username
- Enumerated available Organizational Units (OUs)

### 1. Create Group 1 (Base Group)
---

**Action:**

- Create first nested group in IT OU:

```C
New-ADGroup -Path "OU=IT,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "{USERNAME} Net Group 1" -SamAccountName "{USERNAME}_nestgroup1" -DisplayName "{USERNAME} Nest Group 1" -GroupScope Global -GroupCategory Security
```

**Parameters:**

- **New-ADGroup** - Cmdlet creates new Active Directory security group
- **Path** - Specifies where group will be located (in this case, IT OU)
- **Name** - Friendly name for the group
- **SamAccountName** - Unique identifier for the group
- **DisplayName** - Friendly display name for the group
- **GroupScope** - Set to Global, meaning it can contain members from same domain
- **GroupCategory** - Set to Security, which indicates group is for security-related permissions

**Observable:**

- `powershell.exe` process execution
- LDAP queries to Domain Controller on port 389/636
- Event ID 4727 (Security-enabled global group created) on DC
- New group object created in IT OU
- Group attributes set: SamAccountName, DisplayName, GroupScope, GroupCategory
- Event ID 4662 (Operation performed on AD object) - group creation
- AD replication traffic if multiple DCs

### 2. Create Group 2 in Different OU
---

**Action:**

- Create second nested group in SALES OU:

```C
New-ADGroup -Path "OU=SALES,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "{USERNAME} Net Group 2" -SamAccountName "{USERNAME}_nestgroup2" -DisplayName "{USERNAME} Nest Group 2" -GroupScope Global -GroupCategory Security
```

**Observable:**

- `powershell.exe` process execution
- LDAP queries to Domain Controller
- Event ID 4727 (Security-enabled global group created) on DC
- New group object created in SALES OU
- Group created in different OU than Group 1 (spreads visibility across OUs)

### 3. Add Group 1 as Member of Group 2
---

**Action:**

- Nest Group 1 inside Group 2:

```C
Add-ADGroupMember -Identity "{USERNAME}_nestgroup2" -Members "{USERNAME}_nestgroup1"
```

**Parameters:**

- **Add-ADGroupMember** - Adds previously created `{USERNAME}_nestgroup1` as member of `{USERNAME}_nestgroup2`, establishing first level of nesting

**Observable:**

- `powershell.exe` executing `Add-ADGroupMember` cmdlet
- LDAP modify operation to Domain Controller
- Event ID 4728 (Member added to security-enabled global group) on DC:
  - Target: `{USERNAME}_nestgroup2`
  - Member: `{USERNAME}_nestgroup1`
- Event ID 4662 (Operation performed on AD object) - group modification
- First link in nested chain established

### 4. Create and Nest Groups 3, 4, and 5
---

**Action:**

- Create Group 3 and add Group 2 as member:

```C
New-ADGroup -Path "OU=CONSULTING,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "{USERNAME} Net Group 3" -SamAccountName "{USERNAME}_nestgroup3" -DisplayName "{USERNAME} Nest Group 3" -GroupScope Global -GroupCategory Security
```
```C
Add-ADGroupMember -Identity "{USERNAME}_nestgroup3" -Members "{USERNAME}_nestgroup2"
```

- Create Group 4 and add Group 3 as member:

```C
New-ADGroup -Path "OU=MARKETING,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "{USERNAME} Net Group 4" -SamAccountName "{USERNAME}_nestgroup4" -DisplayName "{USERNAME} Nest Group 4" -GroupScope Global -GroupCategory Security
```
```C
Add-ADGroupMember -Identity "{USERNAME}_nestgroup4" -Members "{USERNAME}_nestgroup3"
```

- Create Group 5 and add Group 4 as member:

```C
New-ADGroup -Path "OU=MANAGEMENT,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "{USERNAME} Net Group 5" -SamAccountName "{USERNAME}_nestgroup5" -DisplayName "{USERNAME} Nest Group 5" -GroupScope Global -GroupCategory Security
```
```C
Add-ADGroupMember -Identity "{USERNAME}_nestgroup5" -Members "{USERNAME}_nestgroup4"
```

**Observable:**

- Multiple `powershell.exe` cmdlet executions
- Multiple Event ID 4727 (Security-enabled global group created) - one per group
- Multiple Event ID 4728 (Member added to security-enabled global group) - one per nesting operation
- Groups created across different OUs (spreads audit trail)
- Nested chain deepens: Group 5 -> Group 4 -> Group 3 -> Group 2 -> Group 1
- Each nesting operation generates separate audit events

**Note:** Obviously you would not follow the `{USERNAME}_nestgroup` naming convention and try to imitate the environment.

### 5. Add Group 5 to Domain Admins
---

**Action:**

- Add final nested group to Domain Admins group:

```C
Add-ADGroupMember -Identity "Domain Admins" -Members "{USERNAME}_nestgroup5"
```

**Observable:**

- `powershell.exe` executing `Add-ADGroupMember` cmdlet
- LDAP modify operation to Domain Controller
- Event ID 4728 (Member added to security-enabled global group) on DC:
  - Target: **Domain Admins** (protected group)
  - Member: `{USERNAME}_nestgroup5`
- Event ID 4662 (Operation performed on AD object) - Domain Admins modification
- **This is the only change to Domain Admins that appears in direct membership**
- Protected group modification may trigger security alerts
- Security team sees `{USERNAME}_nestgroup5` added, but not the nested chain below it

**Effect:**

By adding `{USERNAME}_nestgroup5` to Domain Admins, any user who is member of `{USERNAME}_nestgroup5` (either directly or indirectly via nested groups) will now have Domain Admin privileges.

### 6. Add Low-Privileged User to Group 1
---
**Action:**

- Add your low-privileged AD user to the first (bottom) group in chain:

```C
Add-ADGroupMember -Identity "{USERNAME}_nestgroup1" -Members "{LOW_PRIV_USER}"
```

**Observable:**

- `powershell.exe` executing `Add-ADGroupMember` cmdlet
- LDAP modify operation to Domain Controller
- Event ID 4728 (Member added to security-enabled global group) on DC:
  - Target: `{USERNAME}_nestgroup1` (appears as benign group in IT OU)
  - Member: `{LOW_PRIV_USER}` (low-privileged user)
- No alert triggered for Domain Admins modification (user not added directly)
- Transitive membership now grants Domain Admin access through nested chain
- Security tools checking only direct Domain Admins membership will miss this

### 7. Verify Low-Privileged User Has Domain Admin Access
---

**Action:**

- Test Domain Admin access from low-privileged user account:

```C
dir \\thmdc.za.tryhackme.loc\c$\
```

**Observable:**

- Authentication as `{LOW_PRIV_USER}` (low-privileged account)
- SMB connection to DC on port 445
- Event ID 4624 (Logon) on DC:
  - Logon Type 3 (Network)
  - Account: `{LOW_PRIV_USER}` (appears as normal user)
  - Authentication Package: Kerberos or NTLM
- Event ID 4672 (Special privileges assigned) - **Domain Admin privileges granted**
- Access token contains:
  - User's primary SID
  - SID of Group 1
  - Transitive SIDs from Group 2, 3, 4, 5
  - **Domain Admins SID (through Group 5 membership)**
- Successful directory listing of DC's `C$` administrative share
- **User appears as normal domain user in AD Users and Computers**
- Direct check of Domain Admins membership shows only Group 5, not the user

### 8. Verify Domain Admins Direct Membership
---

**Action:**

- Check direct members of Domain Admins group:

```C
Get-ADGroupMember -Identity "Domain Admins"
```

**Observable:**

- `powershell.exe` process execution
- LDAP query to Domain Controller
- Event ID 4662 (Operation performed on AD object) - reading Domain Admins
- Output shows:
  - Legitimate Domain Admin accounts
  - `{USERNAME}_nestgroup5` (only new entry)
  - **Does NOT show:** Group 1, 2, 3, 4, or `{LOW_PRIV_USER}`
- Security team reviewing this output will see only one new group added
- Requires recursive enumeration (`Get-ADGroupMember -Recursive`) to discover full chain

**Reminder:** Only do this if it is necessary and has been agreed upon.
## Blue - Detection & Response
---

**Key Logs:**

- Event ID 4727 (Security-enabled global group created) - Multiple new groups
- Event ID 4728 (Member added to security-enabled global group) - Nested group additions
- Event ID 4662 (Operation performed on AD object) - Group modifications
- Event ID 4624 (Logon) - Low-privilege accounts with high-privilege access
- Event ID 4672 (Special privileges assigned) - Unexpected privileged access
- PowerShell script block logging - `New-ADGroup` and `Add-ADGroupMember` cmdlets
- Sysmon Event ID 1 (Process creation) - PowerShell with AD cmdlets

**Detection Patterns:**

- Multiple new security groups created in short timeframe
- Groups created across multiple OUs by same account
- Sequential group membership additions (nested chain pattern)
- New group added to Domain Admins or other protected groups
- Low-privilege accounts accessing high-privilege resources
- Groups with generic or templated naming patterns
- Recursive group membership enumeration revealing unexpected chains
- Event ID 4728 for Domain Admins from non-standard administrative account

**High-Value Indicators:**

- 5+ new security groups created within 1 hour
- Groups created in different OUs sequentially
- New group added to Domain Admins with no prior history
- Group naming patterns like "nestgroup1", "nestgroup2", etc.
- Low-privilege user account accessing DC administrative shares
- Event ID 4672 showing Domain Admin privileges for standard user
- Recursive membership check showing deep nesting (3+ levels)
- Groups with single member that is also a group (nesting indicator)
- Group creation and membership modification from same source in rapid succession