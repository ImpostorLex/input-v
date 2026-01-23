---
{"dg-publish":true,"permalink":"/cards/red-team/harvesting-windows-credential-manager/"}
---

~ [[cards/red-team/Credential Harvesting\|Credential Harvesting]]
## Summary
---
**What it is:** Extracting stored credentials from Windows Credential Manager vaults (Web Credentials and Windows Credentials) using built-in tools like vaultcmd, PowerShell scripts, or Mimikatz to obtain usernames, passwords, and internet addresses for websites, applications, and networks.

**Scope:** This note covers enumerating and extracting credentials from Windows Credential Manager using GUI, vaultcmd command-line utility, PowerShell extraction scripts (Get-WebCredentials.ps1), RunAs with /savecred argument, and Mimikatz sekurlsa::credman. Methods include both direct credential reading and leveraging saved credentials for execution.

**Prerequisites:**

- [[Windows Credential Manager\|Windows Credential Manager]]
- [[LSASS Process\|LSASS Process]]
- User-level access to target system (for own credentials)
- Administrator privileges (for other users' credentials or Mimikatz)
- Understanding of Windows credential storage vaults

**Risks & Limitations:**

- User-level access only retrieves current user's stored credentials
- Administrator access needed for other users' vaults
- Credentials may be encrypted or require additional decryption
- Web Credentials vault may not display passwords without additional tools
- RunAs /savecred only works if credentials were previously saved
- Mimikatz requires SeDebugPrivilege and may trigger AV/EDR
## How It Works (2-4 lines)
---
Windows Credential Manager is a built-in feature that stores logon-sensitive information for websites, applications, and networks, including usernames, passwords, and internet addresses, organized into Web Credentials and Windows Credentials vaults. Users and administrators rely on Credential Manager to save and auto-fill credentials for seamless access to resources without repeated authentication prompts. Attackers with user-level access enumerate and extract their own stored credentials using vaultcmd or GUI, while those with elevated privileges access other users' vaults or use tools like Mimikatz to decrypt credential data from LSASS memory, or leverage RunAs with /savecred to execute commands using previously saved credentials without knowing the actual password.

## Steps (Hands-On)
---

**Legend:**

- `{TARGET_USER}` = Current or target username
- `{VAULT_NAME}` = Web Credentials / Windows Credentials
- `{SAVED_USER}` = thm-local (user with saved credentials)
- `{DOMAIN}` = THM.red
- `{CREDENTIAL_TARGET}` = Domain:interactive=thm\thm-local
- `{SCRIPT_PATH}` = C:\Tools\Get-WebCredentials.ps1

**Context:** You have access to a Windows system and want to extract stored credentials from Credential Manager vaults or leverage saved credentials for execution.
### Method 1: Graphical User Interface (GUI)
---
#### 1. Access Credential Manager via GUI

**Action:**

- Navigate to Credential Manager:
  - Open **Control Panel**
  - Go to **User Accounts**
  - Click **Credential Manager**
- Review stored credentials in both vaults:
  - Web Credentials
  - Windows Credentials

**Observable:**

- `explorer.exe` or `control.exe` process execution
- GUI navigation to Control Panel
- Credential Manager interface opened
- User viewing stored credential entries
- No Event IDs generated (read-only GUI access)
- Credentials visible but passwords may be masked/encrypted

### Method 2: VaultCmd Command-Line Utility
---
#### 1. List Available Vaults

**Action:**

- Enumerate all current available Windows vaults:
```C
vaultcmd /list
```

**Observable:**

- `cmd.exe` or `powershell.exe` process execution
- `vaultcmd.exe` child process spawned
- Event ID 4688 (Process creation) - vaultcmd.exe
- Output shows two default vaults:
  - Web Credentials
  - Windows Credentials
- No sensitive data revealed yet (vault names only)

---

#### 2. List Vault Properties

**Action:**

- List all stored credentials in Web Credentials vault:
```C
VaultCmd /listproperties:"Web Credentials"
```

**Observable:**

- `vaultcmd.exe` process execution
- Vault enumeration operation
- Output shows number of stored credentials:
```
  Number of credentials: 1
```
- Vault properties displayed but not credential details yet
- Event ID 4688 (Process creation)

---
#### 3. List Stored Credentials

**Action:**

- List detailed information about stored credentials:
```C
VaultCmd /listcreds:"Web Credentials"
```

**Observable:**

- `vaultcmd.exe` process execution
- Credential enumeration from vault
- Output displays credential metadata:
  - Credential name/target
  - Type (Web Credential, Windows Credential)
  - Username
  - **Password may be hidden or not displayed**
- Event ID 4688 (Process creation)
- If credentials exist but password not shown, additional tools needed

---
#### 4. Extract Passwords Using PowerShell Script

**Action:**

- If vaultcmd does not reveal passwords, use Get-WebCredentials.ps1 script:
```C
Import-Module C:\Tools\Get-WebCredentials.ps1
```
```C
Get-WebCredentials
```

**Observable:**

- `powershell.exe` process execution
- PowerShell module import from `{SCRIPT_PATH}`
- Script accessing Windows Credential Manager APIs
- Sysmon Event ID 1 (Process creation) - PowerShell with script
- PowerShell script block logging (Event ID 4104) if enabled:
  - ScriptBlock: `Get-WebCredentials` function execution
- Credentials extracted and displayed including:
  - Username
  - **Password in plaintext**
  - Target URL/resource
- Vault decryption operations via DPAPI

### Method 3: RunAs with Saved Credentials
---
#### 1. Enumerate Saved Credentials

**Action:**

- Use cmdkey to enumerate stored Windows Credentials:
```C
cmdkey /list
```

**Observable:**

- `cmd.exe` process execution
- `cmdkey.exe` child process spawned
- Event ID 4688 (Process creation) - cmdkey.exe
- Output displays stored credentials for RunAs:
```
  Currently stored credentials:
  
      Target: Domain:interactive=thm\thm-local
      Type: Domain Password
      User: thm\thm-local
```
- Credential target and username revealed
- **Password not displayed** but stored for RunAs usage

---

#### 2. Execute Commands Using Saved Credentials

**Action:**

- Use runas with /savecred to execute commands as saved user:
```C
runas /savecred /user:THM.red\thm-local cmd.exe
```

**Observable:**

- `cmd.exe` executing `runas.exe`
- `runas.exe` child process spawned
- Event ID 4688 (Process creation):
  - Process: `runas.exe`
  - CommandLine: `/savecred /user:THM.red\thm-local cmd.exe`
- New `cmd.exe` process spawned with `{SAVED_USER}` credentials
- Event ID 4624 (Logon):
  - Logon Type 2 (Interactive) or 9 (NewCredentials)
  - Account: `THM.red\thm-local`
  - Logon Process: Advapi (RunAs)
- **No password prompt** - credentials retrieved from Credential Manager
- New command prompt opens under `{SAVED_USER}` context
- Process tree: `cmd.exe` -> `runas.exe` -> `cmd.exe` (different user)

### Method 4: Mimikatz Extraction
---
#### 1. Launch Mimikatz and Remove LSASS Protection

**Action:**

- Launch Mimikatz with elevated privileges:
```C
mimikatz.exe
```

- Remove LSASS process protection (if PPL enabled):
```C
!+
```
```C
!processprotect /process:lsass.exe /remove
```

**Observable:**

- `mimikatz.exe` process execution
- Driver loading for process protection manipulation
- Sysmon Event ID 6 (Driver loaded) - Mimikatz driver
- Event ID 4688 (Process creation) - mimikatz.exe
- LSASS process protection modification
- Kernel-level operations to disable Protected Process Light (PPL)
- Event ID 3033 (Driver signing verification) if Windows enforces signature checks

---

#### 2. Enable Debug Privileges

**Action:**

- Enable SeDebugPrivilege to access LSASS memory:
```C
privilege::debug
```

**Observable:**

- SeDebugPrivilege enabled for Mimikatz process
- Event ID 4673 (Sensitive privilege use):
  - PrivilegeName: SeDebugPrivilege
  - Process: mimikatz.exe
- Mimikatz now has capability to access protected processes like LSASS

---
#### 3. Dump Credential Manager Data

**Action:**

- Extract Credential Manager data from LSASS memory:
```C
sekurlsa::credman
```

**Observable:**

- Mimikatz accessing LSASS process memory
- Sysmon Event ID 10 (ProcessAccess):
  - SourceImage: mimikatz.exe
  - TargetImage: `C:\Windows\System32\lsass.exe`
  - GrantedAccess: `0x1010` or higher
- LSASS memory read operations
- Event ID 4656 (Handle to object requested) - LSASS process handle
- Credential Manager vault data extracted from memory
- Output displays:
  - Usernames
  - Domains
  - **Passwords in plaintext** (if available)
  - Target resources
  - Credential types
- DPAPI master keys used to decrypt stored credentials
- All users' Credential Manager data accessible (if running as SYSTEM)
## Blue - Detection & Response
---

**Key Logs:**

- Event ID 4688 (Process creation) - vaultcmd.exe, cmdkey.exe, runas.exe, mimikatz.exe
- Event ID 4673 (Sensitive privilege use) - SeDebugPrivilege (Mimikatz method)
- Event ID 4624 (Logon) - RunAs creating new logon session with saved credentials
- Event ID 4104 (PowerShell script block logging) - Get-WebCredentials.ps1 execution
- Event ID 4656 (Handle to object requested) - LSASS access (Mimikatz)
- Event ID 3033 (Driver signing verification) - Mimikatz driver loading
- Sysmon Event ID 1 (Process creation) - PowerShell, Mimikatz, vaultcmd
- Sysmon Event ID 6 (Driver loaded) - Mimikatz driver
- Sysmon Event ID 7 (Image loaded) - Suspicious DLL loads by Mimikatz
- Sysmon Event ID 10 (ProcessAccess) - Mimikatz accessing LSASS

**Detection Patterns:**

- VaultCmd execution outside normal user workflows
- CmdKey enumeration followed immediately by RunAs execution
- PowerShell importing scripts from non-standard locations
- Get-WebCredentials.ps1 or similar credential extraction scripts
- Mimikatz.exe process execution or in-memory execution
- LSASS memory access by non-security tools
- RunAs with /savecred flag usage from non-administrative contexts
- Process protection removal on LSASS (Mimikatz driver)
- SeDebugPrivilege usage by non-system processes
- Credential Manager API calls from suspicious processes

**High-Value Indicators:**

- Mimikatz process execution or known Mimikatz strings in memory
- LSASS accessed by user-mode processes (not security tools)
- Vaultcmd or cmdkey executed by recently compromised accounts
- PowerShell script execution from Temp, Downloads, or user directories
- Driver loading for process protection bypass (Mimikatz !+ command)
- Sequential execution: cmdkey -> runas within seconds
- Sysmon Event ID 10 with TargetImage lsass.exe from non-whitelisted sources
- Event ID 4673 SeDebugPrivilege from PowerShell or unknown executables
- Credential enumeration during non-business hours
- Multiple credential extraction methods attempted in sequence