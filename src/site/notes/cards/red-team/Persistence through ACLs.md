---
{"dg-publish":true,"permalink":"/cards/red-team/persistence-through-ac-ls/"}
---

~ [[cards/red-team/images/Persisting Active Directory\|Persisting Active Directory]]
## Summary
---

**What it is:** Injecting Access Control Entries (ACEs) into the AdminSDHolder container to gain persistent full control permissions over all Protected Groups (Domain Admins, Enterprise Admins, Schema Admins) by exploiting the Security Descriptor Propagator (SDProp) service that automatically propagates AdminSDHolder ACL to protected groups every 60 minutes.

**Scope:** This note covers injecting low-privileged user permissions into AdminSDHolder container using MMC, forcing SDProp propagation with PowerShell to immediately apply changes, and verifying persistent full control over Protected Groups even after manual removal.

**Prerequisites:**

- [[AdminSDHolder Container\|AdminSDHolder Container]]
- [[Security Descriptor Propagator (SDProp)\|Security Descriptor Propagator (SDProp)]]
- [[Access Control Lists (ACLs)\|Access Control Lists (ACLs)]]
- [[Active Directory Protected Groups\|Active Directory Protected Groups]]
- Domain Administrator credentials
- Access to Domain Controller or system with AD management tools
- Understanding of ACE injection and permission inheritance

**Risks & Limitations:**

- Requires Domain Admin privileges to modify AdminSDHolder
- SDProp propagation occurs every 60 minutes (unless forced)
- Changes apply to ALL protected groups (wide footprint)
- Modifications to AdminSDHolder logged in Event ID 5136
- Blue team can identify unauthorized ACEs in AdminSDHolder
- Persistence survives membership removal but not ACL cleanup
- Forces SDProp can trigger detection if monitored

**MITRE ATT&CK:** T1098.002 - Account Manipulation: Additional Email Delegate Permissions

**Note:** MITRE does not have specific sub-technique for AdminSDHolder. This technique shares persistence goals with T1098 parent.

## How It Works (2-4 lines)
---

The AdminSDHolder container exists in every Active Directory domain with its Access Control List (ACL) serving as a template to copy permissions to all Protected Groups including Domain Admins, Administrators, Enterprise Admins, and Schema Admins. The Security Descriptor Propagator (SDProp) service automatically takes the AdminSDHolder container's ACL and applies it to all protected groups every 60 minutes through normal AD processes. Administrators rely on AdminSDHolder to maintain consistent security permissions across privileged groups and use SDProp for automated ACL enforcement. Attackers with Domain Admin access inject Access Control Entries granting low-privileged accounts Full Control permissions into AdminSDHolder, then either wait 60 minutes or force SDProp propagation using PowerShell to immediately apply these permissions to all Protected Groups - even if blue team removes the account from protected groups, SDProp automatically re-grants permissions every 60 minutes, making cleanup extremely difficult without identifying and removing the source ACE from AdminSDHolder.

## AdminSDHolder Concept
---

**The Problem:**

Adding account to every single privileged group still allows blue team to perform cleanup and remove membership. 

**The Solution:**

Instead, inject into templates that generate default groups. Even if they remove your membership, you only need to wait until template refreshes to be granted membership again.

**AdminSDHolder Container:**

- Exists in every AD domain
- Access Control List (ACL) used as template to copy permissions to all protected groups
- Protected groups include:
  - Domain Admins (DA)
  - Administrators
  - Enterprise Admins (EA)
  - Schema Admins

**SDProp Process:**

Process called SDProp takes ACL of AdminSDHolder container and applies it to all protected groups every 60 minutes.

**Attack Vector:**

You can write an ACE that grants full permissions on all protected groups. Since this reconstruction occurs through normal AD processes, it would not show alerts to blue team, making it harder to pinpoint source of persistence.

## Steps (Hands-On)
---

**Legend:**

- `{DOMAIN_ADMIN}` = Administrator
- `{DOMAIN}` = tryhackme.loc
- `{DC_FQDN}` = thmchilddc.tryhackme.loc
- `{LOW_PRIV_USER}` = your low-privileged username
- `{PROTECTED_GROUP}` = Domain Admins
- `{SCRIPT_PATH}` = .\Invoke-ADSDPropagation.ps1

**Context:** You have Domain Administrator credentials and want to establish persistent full control over all Protected Groups by injecting ACE into AdminSDHolder container.

---

### 1. Inject Domain Admin Credentials

**Action:**

- Use runas to inject Domain Administrator credentials into memory:
```C
runas /netonly /user:thmchilddc.tryhackme.loc\Administrator cmd.exe
```

**Observable:**

- `cmd.exe` process execution
- `runas.exe` child process spawned
- Event ID 4688 (Process creation):
  - Process: `runas.exe`
  - CommandLine: `/netonly /user:{DC_FQDN}\{DOMAIN_ADMIN} cmd.exe`
- New `cmd.exe` process spawned with injected credentials
- Event ID 4648 (Logon using explicit credentials):
  - Subject: Current user
  - Target: `{DOMAIN_ADMIN}@{DC_FQDN}`
- Credentials injected for network authentication only
- Local `whoami` still shows original user

---

### 2. Launch MMC Console

**Action:**

- Launch Microsoft Management Console:
```C
mmc
```

**Observable:**

- `mmc.exe` process spawned from runas command prompt
- Event ID 4688 (Process creation) - mmc.exe
- MMC console opens with Domain Admin network credentials
- Process tree: `cmd.exe` -> `runas.exe` -> `cmd.exe` -> `mmc.exe`

---

### 3. Add Users and Groups Snap-in

**Action:**

- In MMC:
  - Click **File** -> **Add/Remove Snap-in**
  - Select **Active Directory Users and Computers**
  - Click **Add** -> Click **OK**
- Enable **Advanced Features**:
  - In AD Users and Computers snap-in
  - Click **View** -> Check **Advanced Features**

**Observable:**

- MMC snap-in loading activity
- LDAP queries to Domain Controller on port 389/636
- Event ID 4662 (Operation performed on AD object) on DC
- Advanced Features enabled (shows hidden containers)
- Registry reads under `HKLM\Software\Microsoft\MMC`

---

### 4. Navigate to AdminSDHolder Container

**Action:**

- In Active Directory Users and Computers snap-in:
  - Expand **Domain** (e.g., `tryhackme.loc`)
  - Expand **System** container
  - Locate **AdminSDHolder** object

![Persistence through ACLs.png|250](/img/user/cards/red-team/images/Persistence%20through%20ACLs.png)

**Observable:**

- LDAP queries to DC retrieving System container objects
- Event ID 4662 (Operation performed on AD object)
- AdminSDHolder object visible in System container
- Distinguished Name: `CN=AdminSDHolder,CN=System,DC=tryhackme,DC=loc`

---

### 5. Access AdminSDHolder Security Properties

**Action:**

- Right-click **AdminSDHolder** object
- Select **Properties**
- Navigate to **Security** tab

![Persistence through ACLs-1.png|350](/img/user/cards/red-team/images/Persistence%20through%20ACLs-1.png)

**Observable:**

- LDAP query retrieving AdminSDHolder ACL
- Event ID 4662 (Operation performed on AD object)
- Security descriptor read operation
- Current ACL displayed showing existing permissions
- Legitimate admin accounts visible in permissions list

---

### 6. Add Low-Privileged User with Full Control

**Action:**

- In Security tab:
  1. Click **Add**
  2. Search for `{LOW_PRIV_USER}` username
  3. Click **Check Names**
  4. Click **OK**
  5. Select added user
  6. Check **Allow** on **Full Control** permission
  7. Click **Apply**
  8. Click **OK**

**Observable:**

- LDAP modify operation to Domain Controller
- Event ID 5136 (Directory Service object modified) on DC:
  - Object: `CN=AdminSDHolder,CN=System,DC=tryhackme,DC=loc`
  - AttributeLDAPDisplayName: `nTSecurityDescriptor`
  - AttributeValue: New ACE with `{LOW_PRIV_USER}` Full Control
  - Subject: `{DOMAIN_ADMIN}`
- Event ID 4662 (Operation performed on AD object)
- New ACE injected into AdminSDHolder ACL
- Security descriptor updated with low-privileged user permissions
- ACL change timestamp recorded

![Persistence through ACLs-2.png|300](/img/user/cards/red-team/images/Persistence%20through%20ACLs-2.png)

---

### 7. Wait for SDProp Propagation (60 Minutes)

**Action:**

- Wait for Security Descriptor Propagator (SDProp) service to execute automatically
- SDProp runs every 60 minutes and propagates AdminSDHolder ACL to all Protected Groups

**Observable:**

- No immediate observable changes (waiting period)
- After 60 minutes:
  - SDProp service execution on Domain Controller
  - Event ID 4662 (Operation performed on AD object) - multiple entries for each Protected Group
  - ACL modifications on all Protected Groups:
    - Domain Admins
    - Enterprise Admins
    - Schema Admins
    - Administrators
    - Account Operators
    - Backup Operators
    - Print Operators
    - Server Operators
    - Replicator
  - `{LOW_PRIV_USER}` Full Control ACE propagated to all protected groups
  - No alerts generated (normal AD process)

---

### 8. Force Immediate SDProp Propagation (Optional)

**Action:**

- To avoid 60-minute wait, force SDProp propagation using PowerShell:
```C
Import-Module .\Invoke-ADSDPropagation.ps1
```
```C
Invoke-ADSDPropagation
```

**Observable:**

- `powershell.exe` process execution
- PowerShell module import from `{SCRIPT_PATH}`
- Event ID 4688 (Process creation) - powershell.exe
- PowerShell script block logging (Event ID 4104) if enabled:
  - ScriptBlock: `Invoke-ADSDPropagation` function execution
- Forced SDProp execution on Domain Controller
- Immediate LDAP modify operations to all Protected Groups
- Multiple Event ID 5136 (Directory Service object modified):
  - Object: Each Protected Group
  - AttributeLDAPDisplayName: `nTSecurityDescriptor`
  - ACL updated with `{LOW_PRIV_USER}` Full Control
- Event ID 4662 (Operation performed on AD object) - rapid succession for each group
- ACL propagation completes within 1-2 minutes

---

### 9. Verify Permissions on Protected Group

**Action:**

- Wait one minute after forcing SDProp
- Navigate to Protected Group (e.g., Domain Admins):
  - Right-click **Domain Admins** group
  - Select **Properties**
  - Go to **Security** tab
- Verify `{LOW_PRIV_USER}` has Full Control

![Persistence through ACLs-3.png|300](/img/user/cards/red-team/images/Persistence%20through%20ACLs-3.png)

**Observable:**

- LDAP query retrieving Domain Admins ACL
- Event ID 4662 (Operation performed on AD object)
- Security descriptor shows `{LOW_PRIV_USER}` with Full Control
- Same ACE present on ALL Protected Groups
- Permissions propagated successfully from AdminSDHolder

---

### 10. Test Persistence (Remove and Re-propagate)

**Action:**

- Test persistence by removing `{LOW_PRIV_USER}` from Domain Admins security permissions
- Rerun PowerShell script to force SDProp:
```C
Invoke-ADSDPropagation
```

- Wait 1 minute and verify permissions restored

**Observable:**

- Manual ACE removal:
  - Event ID 5136 (Directory Service object modified) - ACE removed from Domain Admins
  - Event ID 4662 (Operation performed on AD object)
- Forced SDProp execution:
  - `powershell.exe` executing Invoke-ADSDPropagation
  - Event ID 4688 (Process creation)
  - LDAP modify operations to Protected Groups
  - Event ID 5136 - ACE re-added to Domain Admins from AdminSDHolder template
- **Persistence verified:** Permissions automatically restored
- Blue team cleanup ineffective without removing source ACE from AdminSDHolder

---

### 11. Add User to Protected Group (Leverage New Permissions)

**Action:**

- Using newly granted Full Control permissions, add `{LOW_PRIV_USER}` to Domain Admins group:
  - Right-click **Domain Admins** group
  - Select **Properties**
  - Go to **Members** tab
  - Click **Add**
  - Enter `{LOW_PRIV_USER}`
  - Click **OK**

![Persistence through ACLs-4.png|350](/img/user/cards/red-team/images/Persistence%20through%20ACLs-4.png)

**Observable:**

- LDAP modify operation to Domain Controller
- Event ID 4728 (Member added to security-enabled global group) on DC:
  - Target: Domain Admins
  - Member: `{LOW_PRIV_USER}`
  - Subject: `{LOW_PRIV_USER}` (using Full Control permissions)
- Event ID 4662 (Operation performed on AD object)
- `{LOW_PRIV_USER}` now member of Domain Admins
- Full Domain Admin privileges granted
- Even if removed from group, Full Control permissions persist via SDProp

---

## Blue - Detection & Response
---

**Indicators of Compromise:**

- Event ID 5136 (Directory Service object modified) on AdminSDHolder container with nTSecurityDescriptor changes
- Event ID 4662 showing operations on `CN=AdminSDHolder,CN=System,DC=domain,DC=com`
- Event ID 4648 (Logon using explicit credentials) with runas to Domain Admin account
- Event ID 4688 with mmc.exe process creation following runas
- PowerShell script execution of Invoke-ADSDPropagation or similar SDProp manipulation
- Event ID 4104 (PowerShell script block logging) containing SDProp-related cmdlets
- Multiple Event ID 5136 in rapid succession across all Protected Groups (forced SDProp)
- Event ID 4728 (Member added to security-enabled global group) for Domain Admins by non-admin account
- Unauthorized user account in AdminSDHolder ACL with Full Control permissions
- ACE modifications on AdminSDHolder outside change control windows
- Low-privileged accounts with Full Control on Protected Groups
- SDProp execution outside normal 60-minute schedule
- Event ID 4662 spike for multiple Protected Group objects from single source
- Same ACE appearing on multiple Protected Groups simultaneously
- Permissions restored on Protected Groups after manual removal (indicates AdminSDHolder persistence)