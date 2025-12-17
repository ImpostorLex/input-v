---
{"dg-publish":true,"permalink":"/cards/red-team/persistence-through-tickets/"}
---

~ [[Persisting Active Directory\|Persisting Active Directory]]
## Summary
---
**What it is:** Forging Kerberos Ticket Granting Tickets (Golden Tickets) using the KRBTGT account hash to gain unrestricted domain access, or forging Ticket Granting Service tickets (Silver Tickets) using machine account hashes to gain targeted server access, bypassing normal Kerberos authentication.

**Scope:** This note covers Golden Ticket creation for domain-wide persistence, Silver Ticket creation for targeted host persistence, and the differences in scope, detection, and defensive countermeasures between both techniques.

**Prerequisites:** 

- [[cards/active-directory/Kerberos#Kerberos Authentication Process\|Kerberos#Kerberos Authentication Process]]
- [[cards/active-directory/DC Sync\|DC Sync]] or equivalent method to obtain password hashes
- KRBTGT account NTLM hash (for Golden Tickets)
- Machine account NTLM hash (for Silver Tickets)
- Domain SID
- Mimikatz or equivalent ticket forging tool

**Risks & Limitations:**
- Golden Tickets require KRBTGT hash (needs DA privileges or DC compromise)
- Golden Tickets valid until KRBTGT password reset twice (24-hour gap)
- Silver Tickets limited to single host/service
- Machine account passwords rotate every 30 days (affects Silver Ticket persistence)
- Forged tickets may have unrealistic lifetimes (10 years default) triggering detection
- KRBTGT password rotation causes service disruptions across environment

## How It Works (2-4 lines)
---
Kerberos authentication normally requires users to prove identity to the KDC by encrypting a timestamp with their password hash, receiving a TGT signed by the KRBTGT account. Golden Tickets exploit possession of the KRBTGT hash (the master Kerberos key) to forge TGTs for any user without KDC contact, bypassing authentication steps 1-2. Silver Tickets forge TGS tickets signed by target machine account hashes, skipping all KDC communication (steps 1-4) to directly access specific services. Both techniques allow attackers to create tickets with arbitrary lifetimes and permissions, granting persistent access until the signing key (KRBTGT or machine account) is rotated.

Read first: [[cards/red-team/Golden Ticket vs Silver Ticket\|Golden Ticket vs Silver Ticket]]
## Steps (Hands-On)
---

**Legend:**
- `{KRBTGT_HASH}` = 16f9af38fca3ada405386b3b57366082 (from DC Sync)
- `{MACHINE_ACCOUNT_HASH}` = 4c02d970f7b3da7f8ab6fa4dc77438f4 (THMSERVER1$ from DC Sync)
- `{DOMAIN_SID}` = S-1-5-21-3885271727-2693558621-2658995185
- `{DOMAIN}` = za.tryhackme.loc
- `{DC_FQDN}` = THMDC.za.tryhackme.loc
- `{TARGET_SERVER}` = THMSERVER1.za.tryhackme.loc
- `{FAKE_USERNAME}` = ReallyNotALegitAccount (Golden) / StillNotALegitAccount (Silver)

**Context:** You need the KRBTGT NTLM hash and Domain SID for Golden Tickets. For Silver Tickets, you need the target machine account's NTLM hash. This information can be obtained during DC Sync attack.

### 1. Obtain Domain SID
---

**Action:**
- Query Active Directory for domain information:
```C
  Get-ADDomain
```

**Observable:**
- `powershell.exe` process execution
- LDAP queries to Domain Controller on port 389/636
- Event ID 4662 (Operation performed on AD object) on DC
- Domain information returned including SID: `S-1-5-21-3885271727-2693558621-2658995185`

**Note:** KRBTGT hash and machine account hash obtained from previous DC Sync attack.

### 2. Forge Golden Ticket
---

**Action:**

- Launch Mimikatz and forge Golden Ticket:

```C
  kerberos::golden /admin:ReallyNotALegitAccount /domain:za.tryhackme.loc /id:500 /sid:{DOMAIN_SID} /krbtgt:{KRBTGT_HASH} /endin:600 /renewmax:10080 /ptt
```

**Parameters:**

- **/admin** - Username to impersonate. Does not have to be valid user.
- **/domain** - FQDN of domain to generate ticket for.
- **/id** - User RID. By default, Mimikatz uses RID 500 (default Administrator account RID).
- **/sid** - SID of domain to generate ticket for.
- **/krbtgt** - NTLM hash of KRBTGT account.
- **/endin** - Ticket lifetime. By default, Mimikatz generates ticket valid for 10 years. Default Kerberos policy is 10 hours (600 minutes).
- **/renewmax** - Maximum ticket lifetime with renewal. By default, Mimikatz generates ticket valid for 10 years. Default Kerberos policy is 7 days (10080 minutes).
- **/ptt** - Inject ticket directly into session (ready to use).

**Observable:**

- `mimikatz.exe` process execution
- Forged TGT created in memory (not requested from KDC)
- No Event ID 4768 (TGT request) on DC - ticket created locally
- Ticket injected into LSASS memory
- Unusual ticket properties if inspected:
  - Lifetime: 10 years (default) vs 10 hours (policy default)
  - Renewal: 10 years vs 7 days
  - User may be non-existent or disabled
  - No corresponding DC logs for ticket issuance

### 3. Verify Golden Ticket Access
---

**Action:**

- Test access to Domain Controller administrative share:

```C
  dir \\yourDC.domain.tld\c$\
```

Example:

```C
dir \\THMDC.za.tryhackme.loc\c$\
```

**Observable:**


- SMB connection to DC on port 445
- Kerberos authentication using forged TGT (no credential prompt)
- Event ID 4624 (Logon) on DC:
  - Logon Type 3 (Network)
  - Account: `{FAKE_USERNAME}` (may not exist in AD)
  - Authentication Package: Kerberos
  - Unusual ticket lifetime if logged
- Event ID 4672 (Special privileges assigned) - Administrator privileges
- Successful directory listing of DC's `C$` administrative share
- No corresponding Event ID 4768 (TGT request) prior to authentication

### 4. Forge Silver Ticket
---

**Action:**

- Launch Mimikatz and forge Silver Ticket:

```C
  kerberos::golden /admin:StillNotALegitAccount /domain:za.tryhackme.loc /id:500 /sid:{DOMAIN_SID} /target:THMSERVER1.za.tryhackme.loc /rc4:{MACHINE_ACCOUNT_HASH} /service:cifs /ptt
```

**Parameters:**

- **/admin** - Username to impersonate. Does not have to be valid user.
- **/domain** - FQDN of domain to generate ticket for.
- **/id** - User RID. By default, Mimikatz uses RID 500 (default Administrator account RID).
- **/sid** - SID of domain to generate ticket for.
- **/target** - Hostname of target server (e.g., `THMSERVER1.za.tryhackme.loc`). Can be any domain-joined host.
- **/rc4** - NTLM hash of machine account of target. Look through DC Sync results for NTLM hash of `THMSERVER1$`. The `$` indicates machine account.
- **/service** - Service requested in TGS. CIFS is safe bet since it allows file access.
- **/ptt** - Inject ticket directly into session (ready to use).

**Observable:**

- `mimikatz.exe` process execution
- Forged TGS created in memory (not requested from KDC)
- No communication with Domain Controller
- No Event ID 4769 (Service ticket request) on DC
- Ticket signed by target machine account, not KRBTGT
- Ticket injected into LSASS memory for immediate use
- Scope limited to specified service (`cifs`) on target server

### 5. Verify Silver Ticket Access
---

**Action:**

- Test access to target server administrative share:

```C
  dir \\thmserver1.za.tryhackme.loc\c$\
```

**Observable:**

- SMB connection to target server (`THMSERVER1`) on port 445
- Kerberos authentication using forged TGS (no credential prompt)
- Event ID 4624 (Logon) on TARGET SERVER only (not DC):
  - Logon Type 3 (Network)
  - Account: `{FAKE_USERNAME}` (may not exist in AD)
  - Authentication Package: Kerberos
- Event ID 4672 (Special privileges assigned) on target server
- Successful directory listing of target server's `C$` administrative share
- No corresponding DC logs - authentication handled entirely by target server
- Target server validates ticket using its own machine account hash
## Blue - Detection & Response
---

**Key Logs:**

- Event ID 4624 (Logon) - Authentication with unusual ticket properties
- Event ID 4672 (Special privileges assigned) - High privileges for non-existent or disabled users
- Event ID 4768 (TGT requested) - Absence when Golden Ticket used
- Event ID 4769 (Service ticket requested) - Absence when Silver Ticket used, or unusual service access patterns
- Event ID 4634 (Logon session terminated) - Session patterns inconsistent with ticket lifetime
- Sysmon Event ID 1 (Process creation) - mimikatz.exe execution
- Sysmon Event ID 10 (Process access) - Mimikatz accessing LSASS for ticket injection

**Detection Patterns:**

- Kerberos authentication (Event ID 4624) without corresponding TGT request (Event ID 4768)
- TGT with lifetime exceeding 10 hours (policy default)
- TGT renewal exceeding 7 days (policy default)
- Authentication by disabled or non-existent user accounts
- Logon from account with no recent password change or TGT request history
- Service access without prior TGT issuance
- Unusual encryption type downgrade (RC4 instead of AES) in tickets
- Mimikatz process artifacts or in-memory detection
- LSASS process access by non-system processes

**High-Value Indicators:**

- Golden Ticket: Domain-wide access with no Event ID 4768 logs
- Silver Ticket: Service access with no Event ID 4769 on DC (logs only on target)
- TGT lifetime of 10 years (Mimikatz default)
- Authentication by non-existent users with administrative privileges
- Ticket encryption type mismatch with domain policy
- Multiple hosts accessed with same forged ticket in short timeframe
- Kerberos authentication from non-domain-joined machines