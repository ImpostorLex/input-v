---
{"dg-publish":true,"permalink":"/cards/red-team/persistence-through-sid-history/"}
---

~ [[Persisting Active Directory\|Persisting Active Directory]]
## Summary
---
**What it is:** Injecting arbitrary Security Identifiers (SIDs) into a low-privileged account's SID History attribute to grant persistent elevated privileges by placing high-privilege group SIDs (Domain Admins, Enterprise Admins) into the user's access token during authentication without modifying actual group membership.

**Scope:** This note covers the legitimate use of SID History during AD migrations, how SID History is processed during authentication, the attack technique of injecting privileged SIDs into low-privilege accounts by patching the ntds.dit database, and why this persistence method is extremely difficult for defenders to detect and remove.

**Prerequisites:** 
- [[Active Directory Security Principals\|Active Directory Security Principals]]
- [[Access Tokens\|Access Tokens]]
- Domain Admin privileges or equivalent to modify ntds.dit
- Direct access to Domain Controller
- DSInternals PowerShell module or equivalent tools
- Understanding of SID structure and group SIDs

**Risks & Limitations:**

- Requires stopping NTDS service (authentication outage during patch)
- Direct modification of ntds.dit database (high risk of corruption)
- Requires Domain Admin or equivalent privileges to perform
- SID History attribute cannot be removed using standard AD tools
- Extremely invasive technique - difficult to reverse without deep AD knowledge
- Persistence remains effective even after KRBTGT rotation and certificate revocation
## How It Works (2-4 lines)
---
Security Identifiers (SIDs) track security principals and account access when connecting to resources. During AD migrations, administrators use SID History to clone account access by attaching old domain SIDs to new accounts, allowing users to retain access to original domain resources while migrating. When users authenticate, Active Directory builds access tokens containing the user's primary SID, group SIDs, and all SIDs from the SID History attribute - Windows does not distinguish between real group membership and SID History SIDs, trusting all SIDs in the token equally. Attackers with Domain Admin privileges exploit this by injecting high-privilege group SIDs (Domain Admins, Enterprise Admins) into low-privilege accounts' SID History attribute by directly patching the ntds.dit database, granting persistent elevated access that survives credential rotation and does not modify visible group membership.

Read first: [[cards/active-directory/SID History Concept\|SID History Concept]]
## Steps (Hands-On)
---

**Legend:**

- `{LOW_PRIV_USER}` = lauren.oconnor (your low-privileged AD account)
- `{TARGET_SID}` = S-1-5-21-3885271727-2693558621-2658995185-512 (Domain Admins group SID)
- `{DOMAIN_SID}` = S-1-5-21-3885271727-2693558621-2658995185
- `{DC_FQDN}` = thmdc.za.tryhackme.loc
- `{NTDS_PATH}` = C:\Windows\NTDS\ntds.dit

**Context:** You have Domain Admin privileges and want to inject Domain Admins SID into a low-privileged account's SID History for persistent elevated access without modifying visible group membership.

### 1. Verify Current SID History (Baseline)
---

**Action:**

- Check low-privileged user's current SID History attribute:

```C
  Get-ADUser {LOW_PRIV_USER} -properties sidhistory,memberof
```

**Observable:**

- `powershell.exe` process execution
- LDAP queries to Domain Controller on port 389/636
- Event ID 4662 (Operation performed on AD object) on DC
- User object queried with SIDHistory attribute
- Expected output showing empty SID History:

```C
  SIDHistory: {}
```

### 2. Retrieve Target Group SID
---

**Action:**

- Get SID of Domain Admins group (or desired target group/user):

```C
  Get-ADGroup "Domain Admins" -Properties SID
```

**Observable:**

- `powershell.exe` process execution
- LDAP queries to Domain Controller
- Event ID 4662 (Operation performed on AD object)
- Group object queried
- Expected output (similar):
```C
  SID               : S-1-5-21-3885271727-2693558621-2658995185-512
```

### 3. Stop NTDS Service
---

**Action:**
- Stop Active Directory Domain Services to unlock ntds.dit database:
```C
  Stop-Service -Name ntds -force
```

**Observable:**

- `powershell.exe` executing `Stop-Service` cmdlet
- `ntds.exe` process termination
- Event ID 1074 (System shutdown/restart initiated) or Event ID 7036 (Service entered stopped state)
- NTDS service status changes to Stopped
- **CRITICAL: Domain authentication stops working for entire network**
- All Kerberos and NTLM authentication requests fail
- Users cannot authenticate until service restarted
- Event ID 1168 (Active Directory Web Services stopped) if ADWS running

### 4. Patch ntds.dit Database with SID History
---

**Action:**

- Use DSInternals PowerShell module to directly patch ntds.dit:

```C
  Add-ADDBSidHistory -SamAccountName '{LOW_PRIV_USER}' -SidHistory '{TARGET_SID}' -DatabasePath C:\Windows\NTDS\ntds.dit
```

**Observable:**

- `powershell.exe` executing DSInternals cmdlet
- Direct file access to `C:\Windows\NTDS\ntds.dit` (locked while NTDS running)
- File modification of ntds.dit database
- No standard AD replication occurs (database offline)
- No Event ID 5136 (Directory Service object modified) - service stopped
- Database transaction logged in ntds.dit transaction logs
- Critical database modification outside normal AD operations

### 5. Restart NTDS Service
---
**Action:**

- Restart Active Directory Domain Services to restore authentication:

```C
  Start-Service -Name ntds
```

**Observable:**

- `powershell.exe` executing `Start-Service` cmdlet
- `ntds.exe` process starts
- Event ID 7036 (Service entered running state)
- NTDS service status changes to Running
- Event ID 1394 (Active Directory Domain Services startup complete)
- Domain authentication restored
- AD replication resumes
- SYSVOL share becomes available again
- **Authentication for entire network restored**

### 6. Verify SID History Injection
---

**Action:**

- Check low-privileged user's SID History attribute after patch:

```C
  Get-ADUser {LOW_PRIV_USER} -Properties sidhistory
```

**Observable:**

- `powershell.exe` process execution
- LDAP queries to Domain Controller
- Event ID 4662 (Operation performed on AD object)
- User object queried with SIDHistory attribute
- SID History now contains injected SID:
```C
  SIDHistory: {S-1-5-21-3885271727-2693558621-2658995185-512}
```
- **Group membership unchanged** - user not actually member of Domain Admins
- No Event ID 4728 (Member added to security-enabled global group) - no actual group change

### 7. Test Elevated Access
---

**Action:**

- Authenticate as low-privileged user and test Domain Admin access:

```C
  dir \\thmdc.za.tryhackme.loc\c$
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
  - User's primary SID (low-privileged)
  - User's actual group SIDs (normal user groups)
  - **SID from SID History (Domain Admins SID)**
- Successful directory listing of DC's `C$` administrative share
- Windows treats SID History SID as legitimate group membership
- **User appears as normal domain user in AD Users and Computers**
- **No visible indication of elevated privileges in AD GUI tools**
## Blue - Detection & Response
---

**Key Logs:**

- Event ID 7036 (Service state change) - NTDS service stopped and started
- Event ID 1074 (System shutdown/restart) - Service stop initiated
- Event ID 1394 (Active Directory startup complete) - NTDS restart
- Event ID 4662 (Operation performed on AD object) - SIDHistory attribute queries
- Event ID 4624 (Logon) - Low-privilege accounts with high-privilege access
- Event ID 4672 (Special privileges assigned) - Unexpected privileged logon from standard user
- File modification logs for ntds.dit (if file integrity monitoring enabled)
- PowerShell script block logging - DSInternals cmdlet usage
- Sysmon Event ID 1 (Process creation) - PowerShell with DSInternals module

**Detection Patterns:**

- NTDS service stopped outside maintenance windows
- Direct ntds.dit file access while service stopped
- PowerShell execution of DSInternals cmdlets (Add-ADDBSidHistory)
- Low-privilege accounts accessing high-privilege resources (DC C$ shares, SYSVOL)
- Event ID 4672 with special privileges for accounts not in privileged groups
- Authentication outages (NTDS service stop) from unexpected sources
- SIDHistory attribute populated for accounts not involved in domain migration
- Accounts with Domain Admin level access but no visible group membership
- Network authentication failures during NTDS service downtime

**High-Value Indicators:**

- SIDHistory attribute non-empty for accounts never involved in migration
- Low-privilege user accessing Domain Controller admin shares successfully
- Event ID 4672 showing privileged access for standard user accounts
- NTDS service stop/start pattern outside change windows
- DSInternals PowerShell module loaded on Domain Controllers
- ntds.dit modification timestamp inconsistent with normal AD operations
- User with Domain Admin access not member of any privileged groups (visible in AD GUI)
- Access token analysis showing Domain Admins SID without corresponding group membership

**CAUTION:**

SID History is a nightmare for defenders:

- Even with highest privileges, it cannot be removed using standard tools like Active Directory Users and Computers
- Requires AD-RSAT PowerShell cmdlets to remove
- Detecting it is worse: does not change group membership, raises no obvious alerts
- Only becomes effective when user authenticates and SID is added to access token
- Result is infuriating: after rotating KRBTGT account twice, invalidating Golden and Silver Tickets, and even rebuilding entire CA infrastructure, attacker can still perform Domain Admin actions using account that appears as normal domain user