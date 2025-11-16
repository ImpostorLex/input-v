---
{"dg-publish":true,"permalink":"/cards/blue-team/digital-forensics/common-windows-forensics-artifacts/"}
---

[[atlas/blue-team\|blue-team]]
### Introduction
---
These are common sources of evidence that can be found on a Windows system.
## Registry 
---

- Additional notes: [[cards/windows/Windows Registry\|Windows Registry]]
- Located at `C:\Windows\System32\Config`, in a DFIR scenario include the `RegBack` or registry backup everytime the Windows shutdown.
- `%USERPROFILE%` then show hidden files. (NTUSER.DAT)
- Extract via [[KAPE\|KAPE]] and then view with Registry Explorer.

**Dirty hive detected:** it means that our transaction logs is not applied to the `ntuser.dat`, transaction logs tracks every registry modification this ensures it matches.
![Common Windows Forensics Artifacts.png|450](/img/user/cards/blue-team/digital-forensics/images/Common%20Windows%20Forensics%20Artifacts.png)
### Kroll Artifact Parser And Extracter (KAPE)
---

- **Purpose:** Extract registries for later analysis.
- Search for "registry" and check all checkboxes, it all should start with the word 'registry'
- Again, ideally the target destination should be an **external storage**.
- [[KAPE\|KAPE]]
- See also [[cards/windows/Windows Registry#Autoruns Forensics (Persistence)\|Autoruns persistence]]
## System (Investigation)
---
**Operating system** information can be found on, including the **install date (EPOCH time)**, build version and more:

```C
SOFTWARE\Microsoft\Windows NT\CurrentVersion
```
#### Current Control Set
---
It contains data and artifacts related to system settings and device drivers, and services that is required to boot properly.

- Controlset002 will represent the last good configuration.
- To determine which control set is being used, we can check the `current` key value found on this registry:
	- `SYSTEM\Select\Current` 
	- `SYSTEM\Select\LastKnownGood` 

**Determining system architecture:**
```C
SYSTEM\CurrentControlSet\Control\Session Manager\Environment
```

- Includes the path where PowerShell will look for imported modules and system environment variables path.

**Determining computer name:**
```C
SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName
```

**Timezone information:**
Knowledge of the timezone is important when we are merging two or more data sources.
```C
SYSTEM\CurrentControlSet\Control\TimeZoneInformation
```

**Network Interfaces and Past Networks** 

```C
SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces
```

Each interface is given a GUID subkey, that contains values related to the interface's TCP/IP configuration such as IP addresses, DHCP IP address and more.

For past networks:
```C
SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Unmanaged
```

And here:
```C
SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Managed
```
## User (Investigation)
---

- _Security Accounts Manager (SAM)_ that contains **username**, their **hashed passwords**, **group information**, **login**, and more.
- Found on: `SAM\Domains\Account\Users`
	- Invalid logins good to determine brute-force and lockouts.
	- Total Logins.
	- Account date creation: such as persistence.
	- Last login and password changed.
	- It includes (RESET data or the security question.)

**Different groups information:**
```C
SAM\Domains\Builtin\Aliases
```

Output:
**1001** refers to our user belonging to the Administrator group.
![Common Windows Forensics Artifacts-1.png](/img/user/cards/blue-team/digital-forensics/images/Common%20Windows%20Forensics%20Artifacts-1.png)
**Logon information**
```C
SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\LogonUI
```

https://github.com/Ahmed-AL-Maghraby/Windows-Registry-Analysis-Cheat-Sheet

## Files (Investigation)
---

- _Shellbags_ stores the user's folder viewing preferences, such as windows size, or icon size, view type, and more.
- Reconstruct past interaction to the filesystem even **deleted**.
- `%USERPROFILE%` then `NTUSER.dat`.

```C
NTUSER.DAT\Software\Microsoft\Windows\Shell\Bags //(or BagMRU)
```

Recommended is **ShellBag Explorer**.

It includes a cool feature MRU which stands for most recently used:
![Common Windows Forensics Artifacts-2.png](/img/user/cards/blue-team/digital-forensics/images/Common%20Windows%20Forensics%20Artifacts-2.png)
**Viewing 'Open File' and 'Save As' dialogues:**

The actual files
```C
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSavePIDlMRU
```

And (The binaries that were used to interact with the files)

```C
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedPidlMRU
```

**Viewing Recent Documents**
```C
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
```

Run dialog:
```C
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
```

**Viewing typed paths (Explorer.exe)**
```C
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths
```

- Useful for viewing probably deleted files and remote network paths.

## Program Execution (Investigation)
---

**Note:** replace NTUSER.dat with (HKCU):

```C
reg query "HKCU\Software\Microsoft\Windows\Shell\Bags"
```

**Viewing what program the user executed and how many times:**

```C
NTUSER.DAT\Software\Microsoft\Windows\Currentversion\Explorer\UserAssist\{GUID}\Count
```

**Viewing Autoruns and other persistence mechanism:**
In this part we need the user specific settings (`NTUSER.DAT`) and the system wide configuration in this case HKLM's `Software` registry hive, so load them both.

```C
SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

```C
SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
```

Then look at here:

```C
NTUSER.DAT\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

```C
NTUSER.DAT\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
```

**Configured Services or Background Services:**

```C
SYSTEM\CurrentControlSet\Services
```

- Start key with a value of `0x02` means it starts at boot.

**Scheduled Tasks:**
```C
SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache
```

- **Note:** remember the subkey of interest in the general view then view it using specific folder for each subkey for more information.
	- The image path of the binary can be found on the `Action` row and then viewing the hexadecimal to ASCII.

**Command History (PowerShell):**

```C
C:\Users\matthew.collins\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine
```

- Stores **PowerShell command history** for each user.
- Similar to Linux's `.bash_history`
- File persists after reboot unless cleared manually.