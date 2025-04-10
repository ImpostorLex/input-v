---
{"dg-publish":true,"permalink":"/cards/blue-team/endpoint-security/windows-registry/","tags":["windows"]}
---

[[cards/blue-team/endpoint-security/Endpoint Security\|Endpoint Security]]
### Introduction
---
The windows registry is a collection of hierarchical database that contain system configuration such as hardware, software, or the user's information.

- See also [[cards/blue-team/digital-forensics/Common Windows Forensics Artifacts\|Registry Forensics]]
### Registry Hives
---
Are logical group of keys, subkeys, and values in the registry:
- Located at `C:\Windows\System32\Config`

- **HKEY_CURRENT_USER (HKCU)** contains root configuration of the current user who is logged on such as user's folders, sreen colors, and Control Panel settings are stored here called user's profile.
	- Logical links to HKU specific subkeys or user's profile.
- **HKEY_USERS (HKU)** contains all the actively loaded user's profile.
	- Contains a DEFAULT for newly added users.
	- **NTUSER.dat** located at user's home directory (hidden file)
- **HKEY_LOCAL_MACHINE (HKLM)** contains configuration information related to the computer (system wide, applied to all users).
	- **SAM** contains user information including password hashes.
	- **SECURITY** contains security policy settings such as user rights and other security settings.
	- **SOFTWARE** contains installed software on the system as well as system-wide settings.
	- **SYSTEM** system settings such as device drivers configurations, system services, and hardware profiles. (contains materials that can decrypt SAM)
- **HKEY_CLASSES_ROOT** ensures the correct program opens when you open a file.
- **HKEY_CURRENT_CONFIG (HKCC)** contains information about the hardware profile used by the computer at startup.
	- A link to specific subkey within HKLM.
#### Interacting with Registry
---
Best and most simple:
```C
regedit.exe
```

```Powershell
reg add "HKCU\SOFTWARE\MICROSOFT\Notepad" /v lfFaceName /t REG_SZ /d "Comic Sans MS" /f
```

- `/v` which key value to change.
- `/t` data type in this case string.
- `/d` the value for the key.
- `/f` force no need for confirmation.

- Query using cmd: `reg query "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run"`
- PowerShell: `Get-ItemProperty -Path "Registry::HKCU\.....\Run"`

## Autoruns Forensics (Persistence)
---
Are tasks, programs or scripts that executed under certain conditions usually when the user logs on.

Task that will run every time the user logs on:
```bash
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

Task that will run only once and deletes itself:
```
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

- `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` 
- `HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce`
	- Difference is this is regardless of the user
- `HKLM\SYSTEM\CurrentControlSet\Services`
	- Attackers could re-write or write an image path to point to their malware on disks.
	- We can use autoruns sysinternal to identify malicious process then query using the below command for more information:
	
```
sc qc "servicename"
```

---
**Autoruns** from the sysinternal suit basically does what the above:

![Windows Registry-1.png](/img/user/cards/windows/images/Windows%20Registry-1.png)
- **Including color coded**:
	- Green verified.
	- Yellow highlight shows the program has unknown publisher or unsigned.
	- Red highlight shows a potential security risks.
### Baselining from scratch
---
1. [https://github.com/p0w3rsh3ll/AutoRuns](https://github.com/p0w3rsh3ll/AutoRuns) (Commands are in the docs or with `Get-Help <function_name>`)
2. Choose `.psm1`
3. Enable unverified script execution with : `Set-ExecutionPolicy unrestricted` 
4. `Import-Module .\AutoRuns.psm1`
5. Check with `Get-Module` by displaying then choose a module `Get-Command -Module AutoRuns`

![Windows Registry-2.png](/img/user/cards/windows/images/Windows%20Registry-2.png)
Always make sure **the system is in a clean state:** 

```PowerShell
Get-PSAutorun -VerifyDigitalSignature |
Where { -not($_.isOSbinary)} |
New-AutoRunsBaseLine -Verbose -FilePath .\Baseline.ps1
```

- Can run the same command to create a current_state 'image' for comparing.
- **Obviously**: we want to store these files in a secure manner alongside with proper naming convention and file integrity checking.

Output:

![Windows Registry-3.png](/img/user/cards/windows/images/Windows%20Registry-3.png)
**Compare baseline** with current state:

```C
Compare-AutoRunsBaseLine -ReferenceBaseLineFile .\Baseline.ps1 -DifferenceBaseLineFile .\CurrentState.ps1 -Verbose
```

Output:

![Windows Registry-4.png](/img/user/cards/windows/images/Windows%20Registry-4.png)
