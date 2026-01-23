---
{"dg-publish":true,"permalink":"/cards/red-team/laps-password-extraction/"}
---

~ [[cards/red-team/Credential Harvesting\|Credential Harvesting]]
## Summary
---
**What it is:** Extracting Local Administrator Password Solution (LAPS) managed passwords from Active Directory by enumerating computers with LAPS enabled, identifying users/groups with read permissions on the ms-mcs-AdmPwd attribute, and using PowerShell cmdlets to retrieve plaintext local administrator passwords.

**Scope:** This note covers LAPS enumeration to identify enabled systems, permission discovery to find accounts with LAPS read rights, and password extraction using Get-AdmPwdPassword cmdlet. Includes background on Group Policy Preferences (GPP) vulnerability that LAPS replaced.

**Prerequisites:**

- [[Local Administrator Password Solution (LAPS)\|Local Administrator Password Solution (LAPS)]]
- [[cards/active-directory/Group Policy Preferences (GPP)\|Group Policy Preferences (GPP)]]
- [[Active Directory Organizational Units\|Active Directory Organizational Units]]
- Domain user account (for enumeration)
- Compromised account with LAPS read permissions (for extraction)
- LAPS deployed in target environment
- PowerShell AD module or LAPS cmdlets available

**Risks & Limitations:**

- Requires LAPS to be deployed in environment (not always present)
- Need account with specific "All extended rights" on LAPS-enabled OUs
- Password extraction logged in AD if auditing enabled
- LAPS passwords expire based on policy (time-limited access)
- Only provides local admin access to specific computers, not domain-wide
- Requires identifying correct OU with LAPS delegation

**MITRE ATT&CK:** T1003.008 - OS Credential Dumping: /etc/passwd and /etc/shadow

**Note:** MITRE does not have specific sub-technique for LAPS. This technique shares credential dumping goals with T1003 parent.

## How It Works (2-4 lines)
---
In large Windows environments, managing local Administrator passwords across thousands of computers is challenging - using the same password everywhere is insecure, while different passwords everywhere is unmanageable. Microsoft's Group Policy Preferences (GPP) originally allowed centralized password management but stored encrypted passwords in SYSVOL XML files accessible to all domain users, which was exploitable after the encryption key was published. In 2015, Microsoft introduced Local Administrator Password Solution (LAPS) which stores local admin passwords in the `ms-mcs-AdmPwd` attribute (plaintext) and expiration time in `ms-mcs-AdmPwdExpirationTime` attribute on computer objects in Active Directory, using `admpwd.dll` to rotate passwords automatically. Attackers enumerate LAPS deployment by checking for admpwd.dll, identify Organizational Units with delegated "All extended rights" for LAPS attributes, compromise users/groups with these permissions, and use Get-AdmPwdPassword cmdlet to retrieve plaintext local administrator passwords for targeted systems.

Read first: [[cards/active-directory/Group Policy Preferences (GPP)\|Group Policy Preferences (GPP)]]

## Steps (Hands-On)
---

**Legend:**

- `{TARGET_COMPUTER}` = creds-harvestin (LAPS-enabled computer)
- `{LAPS_DLL_PATH}` = C:\Program Files\LAPS\CSE
- `{TARGET_OU}` = THMorg
- `{LAPS_GROUP}` = THMGroupReader
- `{COMPROMISED_USER}` = bk-admin (member of THMGroupReader)
- `{DOMAIN}` = Domain name

**Context:** You have domain user access and want to extract LAPS-managed local administrator passwords to gain privileged access to workstations or servers.

---
### 1. Check for LAPS Deployment

**Action:**

- Verify if LAPS is deployed by checking for admpwd.dll:
```C
dir "C:\Program Files\LAPS\CSE"
```

**Observable:**

- `cmd.exe` process execution
- Directory listing operation
- File system enumeration of LAPS installation path
- If LAPS installed, output shows:
  - `admpwd.dll` file present
  - LAPS CSE (Client-Side Extension) directory exists
- No Event IDs (read-only directory listing)
- If directory not found, LAPS may not be deployed

---
### 2. Enumerate LAPS PowerShell Cmdlets

**Action:**

- View available commands for AdmPwd cmdlets:
```C
Get-Command *AdmPwd*
```

**Observable:**

- `powershell.exe` process execution
- PowerShell cmdlet enumeration
- Event ID 4688 (Process creation) - powershell.exe
- PowerShell module query for LAPS cmdlets
- Output displays available LAPS cmdlets:
	- `Find-AdmPwdExtendedRights`
	- `Get-AdmPwdPassword`

	- `Reset-AdmPwdPassword`
		- `Set-AdmPwdComputerSelfPermission`
		- Others
		- Confirms LAPS PowerShell module installed

### 3. Find OUs with LAPS Extended Rights
---

**Action:**

- Identify Organizational Units with "All extended rights" attribute for LAPS:

```C
Find-AdmPwdExtendedRights -Identity *
```

**Observables:**

- `powershell.exe` process execution
- LDAP queries to Domain Controller on port 389/636
- Active Directory enumeration for LAPS permissions
- Event ID 4662 (Operation performed on AD object) on DC
- Output displays OUs with LAPS extended rights:
    - OU Name
    - Status (look for **Delegated** value)
    - ExtendedRightHolders (users/groups with LAPS read permission)
- Multiple OUs may be listed - identify those with Status: Delegated

### 4. Identify LAPS Permission Holders for Target OU

**Action:**

View detailed LAPS permissions for specific OU (e.g., THMorg):

```C
Find-AdmPwdExtendedRights -Identity THMorg
```

**Observables:**

- `powershell.exe` process execution
- LDAP queries to DC for specific OU permissions
- Event ID 4662 (Operation performed on AD object)
- Output displays:
    - OU: `THMorg`
    - Status: `Delegated`
    - **ExtendedRightHolders:** `THMGroupReader` (group with LAPS read permission)
- Identifies which users/groups can read LAPS passwords for this OU

### 5. Enumerate Group Membership
---

**Action:**

Check members of LAPS-enabled group:

```C
net groups "THMGroupReader"
```
**Observable:**

- `cmd.exe` process execution
- `net.exe` child process spawned
- LDAP queries to Domain Controller
- Event ID 4688 (Process creation) - net.exe
- Event ID 4662 (Operation performed on AD object) - group enumeration
- Output shows group members:

```C
  Group name     THMGroupReader
  Comment
  
  Members
  
  -------------------------------------------------------------------------------
  bk-admin
  The command completed successfully.
```

- Identifies `bk-admin` as member with LAPS read permission
- Need to compromise this account to extract LAPS passwords

### 6. Extract LAPS Password
---
**Action:**

- Compromise or impersonate member account (`bk-admin`)
- Use Get-AdmPwdPassword to retrieve local admin password:

```C
Get-AdmPwdPassword -ComputerName creds-harvestin
```

**Observable:**

- `powershell.exe` process execution with `{COMPROMISED_USER}` credentials
- LDAP queries to Domain Controller
- Event ID 4688 (Process creation) - PowerShell
- Event ID 4662 (Operation performed on AD object) on DC:
    - Object: Computer object (`creds-harvestin`)
    - Properties: `ms-mcs-AdmPwd` attribute read
    - Subject: `{COMPROMISED_USER}` (bk-admin)
- LAPS password attribute access logged
- Output displays:
    - **ComputerName:** `creds-harvestin`
    - **DistinguishedName:** Full AD path of computer
    - **Password:** **Plaintext local administrator password**
    - **ExpirationTimestamp:** When password will be rotated
- Local administrator password now available for use

### 7. Use Retrieved Password
---
**Action:**

- Use extracted local administrator password to access target computer:
    - RDP to `{TARGET_COMPUTER}` with local Administrator account
    - PSExec, WinRM, or other remote access methods
    - Local privilege escalation if already on system

**Observable:**

- Authentication attempt to `{TARGET_COMPUTER}` using local Administrator account
- RDP connection (Event ID 4624, Logon Type 10) if using RDP
- Event ID 4624 (Logon Type 3) if using network authentication (PSExec, WinRM)
- Successful logon as local Administrator on target system
- Administrative actions performed with LAPS-managed account


