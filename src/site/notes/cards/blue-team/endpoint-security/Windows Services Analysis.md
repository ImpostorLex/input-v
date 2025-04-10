---
{"dg-publish":true,"permalink":"/cards/blue-team/endpoint-security/windows-services-analysis/"}
---

[[Endpoint Security MOC\|Endpoint Security MOC]]
### Introduction
---
A type of process that runs in a background and most of the time has no user interface.

- It can run from boot
- It can executed by different types of user privileges such as SYSTEM

Demonstration:
```Powershell
sc create BackupService binPath= "C:\Users\tcm\malware.exe" start= auto
```

Then 

```bash
sc start BackupService
```

It will have an error as our meterpreter payload is not configured for interaction with the Service Control Manager or in other words, the binaries (in this case service) should be tailored to SCM.
## Investigations
---

- It can be also found on AutoRuns

**Start Menu:**
```
services.msc
```

Output:

![Windows Services Analysis.png](/img/user/cards/windows/images/Windows%20Services%20Analysis.png)
- It includes viewing the file properties and start parameters for the binary.

**CommandLine version**

Shows all services running in the system
```PowerShell
sc query
```

- add parameter: `state=all`

Query configuration, including commandLine  and path to executable:
```PowerShell
sc qc BackupService
```

Output:
![Windows Services Analysis-1.png](/img/user/cards/windows/images/Windows%20Services%20Analysis-1.png)
**Powershell**

```PowerShell
Get-Service
```

- Pipe into `| Where-Object { $_.Status -eq 'Running'`
- Display specific service: `-Name "BackupService"` or use a wildcard '*'.
	- `| Select-Object *` does not show path to image.

Alternative for powershell:
```Powershell
Get-WmiObject -Class Win32_Service -Filter "Name= 'BackupService'" | Select-Object *
```