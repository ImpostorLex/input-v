---
{"dg-publish":true,"permalink":"/cards/red-team/persistence-through-credentials/"}
---

~ [[Persisting Active Directory\|Persisting Active Directory]]
## Summary
---
**What it is:** Using an account with domain replication permissions to perform a DC Sync attack, harvesting credentials from a Domain Controller by mimicking the replication process between DCs.

**Scope:** This note covers credential harvesting via DC Sync for persistence purposes, focusing on identifying valuable credential targets beyond privileged accounts, and performing mass credential extraction to maintain access after initial compromise.

**Prerequisites:** 
- [[cards/active-directory/DC Sync\|DC Sync]]
- [[Pass-the-Hash\|Pass-the-Hash]]
- Account with domain replication permissions (Replicating Directory Changes, Replicating Directory Changes All)
- Access to compromised Domain Administrator or equivalent account
- Mimikatz or similar DC Sync capable tool

**Risks & Limitations:**
- Requires specific replication permissions (typically Domain Admin or equivalent)
- DC Sync activity generates Event ID 4662 logs on Domain Controllers
- Privileged credentials will be rotated first upon Blue Team detection
- Mass credential dumps create large log files and network traffic
- Harvested NTLM hashes may be rotated or detected during offline cracking

## How It Works (2-4 lines)
---
DC Sync exploits the Directory Replication Service (DRS) Remote Protocol used by Domain Controllers to replicate Active Directory data between each other. Administrators rely on this legitimate replication mechanism to maintain consistent AD state across DCs. Attackers with accounts possessing replication permissions (Replicating Directory Changes and Replicating Directory Changes All) can impersonate a Domain Controller and request password hashes for any account, including KRBTGT, without touching LSASS or requiring local admin access on a DC.

## Steps (Hands-On)
---
**Legend:**

- {DOMAIN} = za.tryhackme.loc
- {DA_USERNAME} = Administrator
- {DA_PASSWORD} = tryhackmewouldnotguess1@
- {LOW_PRIV_USER} = your low-privilege AD username
- {LOG_FILE} = {LOW_PRIV_USER}_dcdump.txt

**Context:** You have compromised Domain Administrator credentials and want to perform persistence techniques targeting near-privileged accounts rather than only high-privilege accounts that will be rotated first.

**Domain Administrator Credentials:**
```C
User: Administrator
Password: tryhackmewouldnotguess1@
Domain: ZA
```

**Note:** In AD environments, credentials refer to username and password, but also username and hash through pass-the-hash techniques.

**Prerequisite:** Read [[cards/active-directory/DC Sync\|DC Sync]] for foundational understanding of the technique.
### Understanding Credential Targeting Strategy
---
**Concept:**

If you have access to an account with domain replication permissions, you can stage a DC Sync attack to harvest credentials from a DC.

**Not All Credentials Are Created Equal:**

While privileged credentials are valuable, they are also rotated first when Blue Team detects compromise. If you only have privileged credentials, you can lose access immediately upon discovery.

**Target near-privileged credentials instead:**

The goal is to harvest credentials with enough 'keys' to ensure goal execution while remaining under the radar. Here are valuable credential targets:

- **Credentials with local administrator rights on several machines** - Organizations typically have groups with local admin rights on most computers, usually divided into workstation admins and server admins. Harvesting members of these groups maintains access to most estate computers.

- **Service accounts with delegation permissions** - These accounts enable forcing golden and silver tickets to perform Kerberos delegation attacks.

- **Accounts used for privileged AD services** - Compromising accounts of services like Exchange, Windows Server Update Services (WSUS), or System Center Configuration Manager (SCCM) allows leveraging AD exploitation to regain privileged foothold.

**Note:** This is case-by-case basis. Every organization has different setup and valuable targets.
### 1. Test DC Sync on Single Account
---
**Action:**

- Test DC Sync capability on a single account (ideally your own low-privilege account) to verify permissions and avoid detection:

```C
  lsadump::dcsync /domain:za.tryhackme.loc /user:{LOW_PRIV_USER}
```

**Observable:**

- `mimikatz.exe` process execution
- Directory Replication Service (DRS) RPC calls to Domain Controller
- Network traffic on port 135 (RPC) and dynamic high ports
- Event ID 4662 (Operation performed on AD object) on DC:
  - Object: Single user account CN
  - Properties: Replicating Directory Changes
  - Subject: Account performing DC Sync
- LDAP queries to Domain Controller
- Single account's credential data returned (SAM username, NTLM hash, Kerberos keys)
### 2. Enable Logging for Mass Credential Dump
---

**Action:**

- Enable Mimikatz logging to file before performing mass DC Sync:

```C
  log {LOG_FILE}
```

**Observable:**

- File creation: `{LOG_FILE}` in current working directory
- All subsequent Mimikatz output redirected to log file
- File size will grow significantly during DC Sync all operation
### 3. Perform DC Sync on All Domain Accounts
---

**Action:**

- Execute DC Sync for all domain accounts:

```C
  lsadump::dcsync /domain:za.tryhackme.loc /all
```

**Observable:**

- Extended DRS RPC communication with Domain Controller
- Sustained network traffic on RPC ports (135 + dynamic high ports)
- Multiple Event ID 4662 (Operation performed on AD object) entries on DC:
  - Rapid succession of replication requests
  - Properties: Replicating Directory Changes, Replicating Directory Changes All
  - Objects: Multiple user accounts and domain objects
- Large log file generation (`{LOG_FILE}`) containing all domain credentials
- Significant CPU and network utilization on both attacking machine and DC
- Potential Event ID 4673 (Sensitive privilege use) if auditing enabled
- Time duration proportional to number of domain accounts (thousands of accounts = several minutes)

**Output Contains:**

- SAM usernames for all domain accounts
- NTLM password hashes
- Kerberos encryption keys (AES256, AES128, DES)
- Password history (if configured)
- KRBTGT account hash (critical for Golden Ticket attacks)
### 4. Extract Credentials from Log File
---

**Action:**

- Extract all usernames from log file:

```C
  cat {LOG_FILE} | grep "SAM Username"
```

- Extract all NTLM hashes from log file:

```C
  cat {LOG_FILE} | grep "Hash NTLM"
```

**Observable:**

- Command line activity parsing log file
- Filtered output showing SAM usernames and NTLM hashes
- Credentials ready for pass-the-hash attacks or offline password cracking

**Critical Credentials to Remember:**

- **KRBTGT hash** - Required for Golden Ticket attacks and long-term persistence
- **Service accounts** - Accounts with delegation permissions or privileged service access
- **Local admin group members** - Accounts with widespread local administrator rights
- **Near-privileged accounts** - Identified targets from strategy planning
### 5. Leverage Harvested Credentials
---

**Action:**

- Perform [[pass-the-hash\|pass-the-hash]] attacks using extracted NTLM hashes
- Conduct offline password cracking on dumped hashes
- Store KRBTGT hash for later Golden Ticket techniques

**Observable:**

- Credential usage varies based on technique employed
- Refer to [[pass-the-hash\|pass-the-hash]] and related TECH notes for specific observables
## Blue - Detection & Response
---

**Key Logs:**

- Event ID 4662 (Operation performed on AD object) on DC - replication requests
  - Properties: {1131f6aa-9c07-11d1-f79f-00c04fc2dcd2} (Replicating Directory Changes)
  - Properties: {1131f6ad-9c07-11d1-f79f-00c04fc2dcd2} (Replicating Directory Changes All)
- Event ID 4673 (Sensitive privilege use) - replication permissions exercised
- Event ID 5136 (Directory Service object modified) if detailed auditing enabled
- Network traffic logs showing sustained RPC communication (port 135 + high ports)
- Sysmon Event ID 1 (Process creation) - mimikatz.exe execution
- Sysmon Event ID 3 (Network connection) - connections to DC on replication ports

**Detection Patterns:**

- Non-DC computers initiating DRS replication requests
- Replication requests from workstations or member servers
- Account with replication permissions not assigned to DC computer account
- Unusual timing of replication activity (outside maintenance windows)
- Multiple Event ID 4662 in rapid succession from single source
- Replication requests for KRBTGT or high-value service accounts
- DC Sync from accounts not typically used for replication

**High-Value Indicators:**

- Event ID 4662 with subject account that is NOT a domain controller
- Source IP for replication request is workstation or member server
- Replication of KRBTGT account by non-DC entity
- Account with replication permissions used interactively (not service)
- Multiple accounts replicated in short timeframe (mass dump pattern)
- Mimikatz process artifacts or memory signatures on endpoints

**Quick Response:**

1. Identify source of DC Sync activity and isolate machine immediately
2. Disable account performing DC Sync and reset password
3. Review account permissions - remove excessive replication rights from non-DC accounts
4. Reset KRBTGT password TWICE (24-hour gap) to invalidate potential Golden Tickets
5. Force password reset for all privileged accounts and service accounts
6. Enable advanced auditing for replication activity (Event ID 4662 on all DCs)
7. Hunt for Mimikatz artifacts and pass-the-hash activity using harvested credentials
8. Review AD replication permissions across all accounts - implement least privilege
9. Deploy detection rules for non-DC replication attempts
10. Investigate lateral movement and persistence mechanisms deployed with harvested credentials

