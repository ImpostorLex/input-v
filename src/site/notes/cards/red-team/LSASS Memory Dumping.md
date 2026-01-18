---
{"dg-publish":true,"permalink":"/cards/red-team/lsass-memory-dumping/"}
---

~ [[cards/red-team/Credential Harvesting\|Credential Harvesting]]
## Summary
---

**What it is:** Dumping the Local Security Authority Subsystem Service (LSASS) process memory to extract credentials, cached passwords, Kerberos tickets, and NTLM hashes stored in memory during runtime.

**Scope:** This note covers dumping LSASS memory using Windows Task Manager GUI and extracting credentials offline. Methods requiring tools like Mimikatz for live extraction are referenced but not detailed here.

**Prerequisites:**

- [[Local Security Authority Subsystem Service (LSASS)\|Local Security Authority Subsystem Service (LSASS)]]
- [[Windows Credential Storage\|Windows Credential Storage]]
- Administrator or SYSTEM privileges on target
- Understanding of Windows authentication mechanisms
- Knowledge of credential caching in memory

**Risks & Limitations:**

- Requires elevated privileges (Administrator/SYSTEM)
- LSASS is a protected process on modern Windows (PPL - Protected Process Light)
- Dumping LSASS can trigger EDR/AV alerts
- LSASS dump files are large (typically 40-100MB+)
- Credentials only available for currently logged-on or recently authenticated users
- Dump file contains sensitive data and must be securely handled

## How It Works (2-4 lines)
---

The Local Security Authority Subsystem Service (LSASS) process (`lsass.exe`) handles Windows authentication and enforces security policies, storing credentials in memory during runtime including plaintext passwords (older systems), NTLM hashes, Kerberos tickets (TGTs/TGS), and cached domain credentials. Administrators and security tools access LSASS for legitimate authentication debugging and troubleshooting. Attackers with elevated privileges dump LSASS process memory using built-in Windows utilities (Task Manager, procdump) or security tools (Mimikatz), then extract stored credentials offline to obtain password hashes, tickets, or plaintext credentials for lateral movement and privilege escalation.

## Steps (Hands-On)
---

**Legend:**

- `{TARGET_SYSTEM}` = Target Windows machine
- `{DUMP_FILE}` = lsass.dmp
- `{DUMP_PATH}` = C:\Users\Administrator\AppData\Local\Temp\lsass.dmp
- `{ATTACKER_IP}` = Attacker machine IP

**Context:** You have Administrator privileges on target system and want to extract credentials from LSASS process memory.
### 1. Open Task Manager with Elevated Privileges
---

**Action:**

- Launch Task Manager as Administrator:
  - Press `Ctrl + Shift + Esc` OR
  - Right-click taskbar -> Task Manager
  - Ensure running as Administrator

**Observable:**

- `taskmgr.exe` process execution with elevated token
- Event ID 4688 (Process creation) - Taskmgr.exe with elevated privileges
- UAC prompt if UAC enabled (Event ID 4648 - explicit credential logon)
- Task Manager window opens with "Administrator" in title bar
### 2. Navigate to Details Tab
---

**Action:**

- In Task Manager, click on **Details** tab
- Locate `lsass.exe` process in the process list
- Sort by Name to easily find LSASS

**Observable:**

- GUI interaction with Task Manager
- No additional Event IDs (read-only operation)
- LSASS process visible in Details tab:
  - Name: `lsass.exe`
  - PID: (varies, typically low number like 600-800)
  - User: `NT AUTHORITY\SYSTEM`
  - Memory usage visible

### 3. Create LSASS Memory Dump
---

**Action:**

- Right-click on `lsass.exe` process
- Select **"Create dump file"** from context menu
- Wait for dump operation to complete (may take 10-30 seconds)

**Observable:**

- Task Manager creating memory dump of LSASS
- `taskmgr.exe` accessing LSASS process memory
- Sysmon Event ID 10 (ProcessAccess):
  - SourceImage: `C:\Windows\System32\taskmgr.exe`
  - TargetImage: `C:\Windows\System32\lsass.exe`
  - GrantedAccess: `0x1fffff` (PROCESS_ALL_ACCESS) or `0x1410`
- Event ID 4656 (Handle to object requested) - LSASS process handle
- Event ID 4663 (Object access attempt) - LSASS memory access
- High CPU and disk I/O during dump creation
- Dump file created at default location (usually Temp folder)
- Progress dialog showing dump creation

### 4. Locate Dump File
---

**Action:**

- After dump completes, Task Manager displays file location
- Typical path: `C:\Users\{USERNAME}\AppData\Local\Temp\lsass.DMP`
- Note the exact path for file transfer

**Observable:**

- Sysmon Event ID 11 (FileCreate):
  - TargetFilename: `C:\Users\Administrator\AppData\Local\Temp\lsass.DMP`
  - Image: `C:\Windows\System32\taskmgr.exe`
- Large file created (typically 40-100MB+ depending on system memory)
- File timestamp matches dump creation time
- File contains raw LSASS process memory

### 5. Transfer Dump File to Attacker Machine
---

**Action:**

- Transfer `lsass.DMP` file to attacker machine for offline analysis
- (Methods: SMB, SCP, HTTP upload, physical media, etc.)
- Securely delete dump file from target after transfer

**Observable:**

- Network connections from target to attacker IP
- File read operations on `lsass.DMP`
- SMB traffic on port 445 (if using network share)
- SSH/SCP traffic on port 22 (if using SCP)
- HTTP/HTTPS traffic (if using web upload)
- Event ID 5140 (Network share accessed) if SMB used
- Sysmon Event ID 3 (Network connection):
  - Image: (transfer tool - explorer.exe, cmd.exe, powershell.exe, etc.)
  - DestinationIp: `{ATTACKER_IP}`
- Large outbound file transfer detected by network monitoring
- Event ID 4663 (File access) - lsass.DMP read operation

### 6. Extract Credentials Offline (Reference)
---

**Action:**

- Use tools like Mimikatz, pypykatz, or other LSASS parsers on attacker machine to extract credentials from dump file
- (Detailed extraction steps intentionally omitted - reference other TECH notes for credential extraction tools)

**Observable:**

- Tool execution on attacker machine (outside target environment)
- No observable indicators on target system (offline analysis)
- Extracted credentials include:
  - NTLM hashes
  - Kerberos tickets (TGTs, TGS)
  - Plaintext passwords (if WDigest enabled)
  - Cached domain credentials
  - Certificate private keys

## Blue - Detection & Response
---

**Key Logs:**

- Event ID 4688 (Process creation) - taskmgr.exe with elevated privileges
- Event ID 4656 (Handle to object requested) - LSASS process handle requested
- Event ID 4663 (Object access attempt) - LSASS memory access
- Event ID 4673 (Sensitive privilege use) - SeDebugPrivilege usage
- Event ID 5140 (Network share accessed) - dump file exfiltration via SMB
- Sysmon Event ID 10 (ProcessAccess):
  - TargetImage: `C:\Windows\System32\lsass.exe`
  - GrantedAccess: `0x1410`, `0x1fffff`, or similar high-privilege access
- Sysmon Event ID 11 (FileCreate) - lsass.DMP file creation
- Sysmon Event ID 3 (Network connection) - dump file exfiltration
- EDR/AV alerts on LSASS access

**Detection Patterns:**

- LSASS process accessed by non-security tools
- Task Manager accessing LSASS with PROCESS_ALL_ACCESS rights
- Large `.DMP` files created in Temp directories
- Files named `lsass.dmp`, `lsass.DMP`, or similar in user directories
- LSASS memory dumps created outside incident response or debugging
- Dump files immediately followed by network file transfers
- SeDebugPrivilege usage by Task Manager or non-standard processes
- LSASS access attempts from unsigned or unknown binaries
- Multiple LSASS access attempts in short timeframe

**High-Value Indicators:**

- Sysmon Event ID 10 with TargetImage `lsass.exe` and SourceImage not in whitelist
- Files with `.dmp` extension in Temp, Desktop, or user directories
- LSASS dump creation during non-business hours
- Dump file exfiltration within minutes of creation
- Task Manager running under non-administrative user attempting LSASS access
- LSASS access by processes spawned from suspicious parent processes
- Protected Process Light (PPL) bypass attempts
- LSASS dumps on systems with no active troubleshooting tickets
- Sequential credential dumping (SAM dump followed by LSASS dump)