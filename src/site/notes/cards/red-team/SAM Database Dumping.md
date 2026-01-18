---
{"dg-publish":true,"permalink":"/cards/red-team/sam-database-dumping/"}
---

~ [[cards/red-team/Credential Harvesting\|Credential Harvesting]]
## Summary
---

**What it is:** Extracting the Security Account Manager (SAM) database from a Windows system to obtain local account password hashes, using methods like Metasploit hashdump, Volume Shadow Copy Service, or registry hive extraction.

**Scope:** This note covers three methods for dumping SAM database: Metasploit in-memory injection, Volume Shadow Copy Service (VSS), and Registry Hives extraction. All methods require the SYSTEM file for decryption.

**Prerequisites:**

- [[Security Account Manager (SAM)\|Security Account Manager (SAM)]]
- [[Volume Shadow Copy Service\|Volume Shadow Copy Service]]
- Administrator-level privileges on target system
- Understanding of Windows registry structure
- SAM database encryption knowledge (RC4/AES)

**Risks & Limitations:**

- Requires Administrator privileges
- SAM database is encrypted and locked while OS is running
- Needs SYSTEM file for decryption
- VSS method creates forensic artifacts (shadow copies)
- Registry method generates Event ID 4663 logs
- Hashes may be salted or use strong encryption
## How It Works (2-4 lines)
---

The Security Account Manager (SAM) is a Microsoft Windows database containing local account information including usernames and password hashes, stored at `C:\Windows\System32\config\sam` and encrypted to prevent unauthorized access while the OS is running. Administrators rely on the SAM for local authentication and account management. Attackers with Administrator privileges bypass the file lock using in-memory injection (Metasploit), Volume Shadow Copy Service to create point-in-time snapshots, or registry hive exports, then decrypt the extracted SAM using the SYSTEM file which contains the necessary encryption keys (Boot Key).

## Steps (Hands-On)
---

**Legend:**

- `{TARGET_SYSTEM}` = Target Windows machine
- `{ADMIN_USER}` = Administrator account
- `{SAM_EXPORT_PATH}` = C:\users\Administrator\Desktop\sam
- `{SYSTEM_EXPORT_PATH}` = C:\users\Administrator\Desktop\system
- `{SHADOW_COPY_ID}` = HarddiskVolumeShadowCopy1
- `{ATTACKER_IP}` = Attacker machine IP address

**Context:** You have Administrator-level access to the target Windows system and want to extract local account password hashes from the SAM database.

### Method 1: Metasploit Hash Dump
---
#### 1. Verify Privilege Level

**Action:**

- Check current user privileges in Meterpreter session:
```C
getuid
```

**Observable:**

- `meterpreter` process execution
- Output shows user context (e.g., `NT AUTHORITY\SYSTEM` or `Administrator`)
- Event ID 4688 (Process creation) if auditing enabled
- Meterpreter session established from external IP

#### 2. Dump SAM Hashes Using Hashdump
---
**Action:**

- Execute hashdump command in Meterpreter:

```C
hashdump
```

**Observable:**

- In-memory code injection into `lsass.exe` process
- Sysmon Event ID 8 (CreateRemoteThread) - remote thread creation in LSASS
- Sysmon Event ID 10 (ProcessAccess) - Meterpreter accessing LSASS memory
- Event ID 4673 (Sensitive privilege use) - SeDebugPrivilege usage
- SAM database hashes dumped from memory without file access
- Network traffic back to attacker machine on Meterpreter port
- Output format: `username:RID:LM_hash:NTLM_hash:::`
- No file system artifacts (SAM file not accessed directly)

### Method 2: Volume Shadow Copy Service
---
#### 1. Create Volume Shadow Copy

**Action:**

- Open `cmd.exe` with Administrator privileges
- Create shadow copy of C: drive:

```C
wmic shadowcopy call create Volume='C:\'
```

**Observable:**

- `cmd.exe` spawned with elevated privileges
- `wmic.exe` process execution
- Volume Shadow Copy Service (`vss.exe`, `vssvc.exe`) activity
- Event ID 8222 (Shadow copy created) in System log
- Event ID 4688 (Process creation) - wmic.exe
- Disk I/O spike during snapshot creation
- New shadow copy device created: `\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy#`
- Forensic artifact: Shadow copy persists until deleted

#### 2. Verify Shadow Copy Creation
---

**Action:**

- List all shadow copies to confirm creation:

```C
vssadmin list shadows
```

**Observable:**

- `vssadmin.exe` process execution
- Output displays shadow copy details:
  - Shadow Copy ID
  - Shadow Copy Volume (e.g., `\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1`)
  - Creation time
  - Source volume
- No additional Event IDs (read-only operation)

#### 3. Copy SAM Database from Shadow Copy
---

**Action:**

- Copy SAM file from shadow copy to accessible location:

```C
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\sam C:\users\Administrator\Desktop\sam
```

**Observable:**

- `cmd.exe` executing `copy` command
- File read from shadow copy path
- File write to `C:\users\Administrator\Desktop\sam`
- Event ID 4663 (Object access attempt) on shadow copy path
- Event ID 4656 (Handle to object requested) - SAM file access
- Sysmon Event ID 11 (FileCreate) - SAM file created at destination
- File with SAM database structure at Desktop location
#### 4. Copy SYSTEM File from Shadow Copy
---

**Action:**

- Copy SYSTEM file (contains decryption key) from shadow copy:

```C
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\system C:\users\Administrator\Desktop\system
```

**Observable:**

- Similar file access patterns as SAM copy
- Event ID 4663 (Object access attempt)
- Sysmon Event ID 11 (FileCreate) - SYSTEM file created
- Both SAM and SYSTEM files now present at Desktop
- Files ready for exfiltration to attacker machine

#### 5. Transfer Files to Attacker Machine
---

**Action:**

- Transfer SAM and SYSTEM files to attacker machine for offline cracking
- (Method depends on available channels - SMB, SSH, HTTP, etc.)

**Observable:**

- Network connections from target to attacker IP
- SMB traffic on port 445 (if using SMB)
- SSH traffic on port 22 (if using SCP)
- HTTP/HTTPS traffic (if using web upload)
- Large file transfers detected by network monitoring
- Event ID 5140 (Network share object accessed) if SMB used
- Sysmon Event ID 3 (Network connection) to external IP

### Method 3: Registry Hives
---
#### 1. Export SAM Registry Hive
---

**Action:**

- Use `reg save` to export SAM registry hive:

```C
reg save HKLM\sam C:\users\Administrator\Desktop\sam-reg
```

**Observable:**

- `reg.exe` process execution
- Registry hive export operation
- Event ID 4656 (Handle to registry key requested) - `HKLM\SAM`
- Event ID 4663 (Attempt to access registry key)
- Sysmon Event ID 12 (RegistryEvent - Object create/delete) - registry access
- Sysmon Event ID 11 (FileCreate) - `sam-reg` file created
- File created at `C:\users\Administrator\Desktop\sam-reg`

#### 2. Export SYSTEM Registry Hive
---

**Action:**

- Use `reg save` to export SYSTEM registry hive (contains Boot Key):

```C
reg save HKLM\system C:\users\Administrator\Desktop\system-reg
```

**Observable:**

- `reg.exe` process execution
- Registry hive export operation
- Event ID 4656 (Handle to registry key requested) - `HKLM\SYSTEM`
- Event ID 4663 (Attempt to access registry key)
- Sysmon Event ID 12 (RegistryEvent)
- Sysmon Event ID 11 (FileCreate) - `system-reg` file created
- Both registry hive exports now available

#### 3. Crack Hashes Offline
---

**Action:**

- Transfer files to attacker machine
- Use Impacket's secretsdump.py to decrypt SAM:

```C
python3.9 /opt/impacket/examples/secretsdump.py -sam /tmp/sam-reg -system /tmp/system-reg LOCAL
```

**Parameters:**

- **-sam** - Path to exported SAM registry hive
- **-system** - Path to exported SYSTEM registry hive  
- **LOCAL** - Argument to decrypt Local SAM file (not domain credentials)

**Observable:**

- Python process execution on attacker machine
- SAM database decryption using Boot Key from SYSTEM file
- Output format: `domain\username:RID:LM_hash:NTLM_hash:::`
- NTLM hashes revealed for all local accounts
- **Note:** Output may differ from Metasploit version - may contain Active Directory account hashes if system is domain-joined (requires SECURITY file to fully decrypt AD accounts)

## Blue - Detection & Response
---

**Key Logs:**

- Event ID 4688 (Process creation) - wmic.exe, vssadmin.exe, reg.exe
- Event ID 4673 (Sensitive privilege use) - SeDebugPrivilege, SeBackupPrivilege
- Event ID 4656 (Handle to object requested) - SAM, SYSTEM file/registry access
- Event ID 4663 (Object access attempt) - SAM database access attempts
- Event ID 8222 (Shadow copy created) - VSS shadow copy creation
- Event ID 5140 (Network share accessed) - file exfiltration via SMB
- Sysmon Event ID 8 (CreateRemoteThread) - LSASS injection (Metasploit method)
- Sysmon Event ID 10 (ProcessAccess) - LSASS memory access
- Sysmon Event ID 11 (FileCreate) - SAM/SYSTEM files created outside System32
- Sysmon Event ID 12 (RegistryEvent) - SAM/SYSTEM registry hive access
- Sysmon Event ID 3 (Network connection) - file exfiltration to external IP

**Detection Patterns:**

- LSASS process accessed by non-system processes (Metasploit method)
- Volume Shadow Copy creation outside backup windows
- SAM/SYSTEM files copied from shadow copy paths
- Registry hive exports of HKLM\SAM and HKLM\SYSTEM
- SAM/SYSTEM files in unusual locations (Desktop, Temp, user directories)
- SeDebugPrivilege usage by non-security tools
- Network file transfers of SAM/SYSTEM files
- Meterpreter or suspicious processes accessing LSASS memory
- wmic.exe creating shadow copies on non-server systems
- reg.exe exporting sensitive registry hives

**High-Value Indicators:**

- LSASS memory access by Meterpreter or unknown processes
- SAM database files outside `C:\Windows\System32\config\`
- Shadow copy creation followed immediately by SAM/SYSTEM copy
- reg.exe exporting both SAM and SYSTEM within short timeframe
- Files named `sam`, `system`, `sam-reg`, `system-reg` in user directories
- Network exfiltration of files matching SAM database size (~40KB-1MB)
- SeBackupPrivilege usage from non-backup software
- Registry hive exports during non-business hours
- Multiple credential dumping methods attempted in sequence