---
{"dg-publish":true,"permalink":"/cards/windows/windows-registry/","tags":["sunday","template"]}
---

[[atlas/windows\|windows]]
### Introduction 
---
The windows registry is a collection of database that contain system configuration such as hardware, software, or the user's information.
## Prerequisites

- **Registry Hives** are logical group of keys, subkeys, and values in the registry.
## Five Root Keys

- **HKEY_CURRENT_USER** contains root configuration of the current user who is logged on such as user's folders, sreen colors, and Control Panel settings are stored here called user's profile.
- **HKEY_USERS** contains all the actively loaded user's profile.
- **HKEY_LOCAL_MACHINE** contains configuration information related to the computer.
- **HKEY_CLASSES_ROOT** ensures the correct program opens when you open a file.
- **HKEY_CURRENT_CONFIG** contains information about the hardware profile used by the computer at startup.
### Forensics
---
- Contains user information:
	- **NTUSER.DAT** (mounted on HKEY_CURRENT_USER when a user logs in) 
		- Located at `C:\Users\<username>\`.
	- **USRCLASS.DAT** (mounted on HKEY_CURRENT_USER\\Software\\CLASSES) 
		- Located at `C:\Users\<username>\AppData\Local\Microsoft\Windows`.
- Recently executed programs:
	- **Amcache.hve** located at `Windows\AppCompat\Programs\Amcache.hve`.
- Network Interfaces and Past Networks
	- **SYSTEM**
	- Located at `\CurrentControlSet\Services\Tcpip\Parameters\Interfaces`.
	- **SOFTWARE** past networks connected.
	- Located at: `Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Unmanaged(managed)`.
- Program tha runs on user startup:
	- **NTUSER.DAT**
	- Located at: `\Software\Microsoft\Windows\CurrentVersion\Run(RunOnce)`.
	- **SOFTWARE**
	- Located at: `SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce`.
	- `SOFTWARE\Microsoft\Windows\CurrentVersion\policies\Explorer\Run`.
	- `SOFTWARE\Microsoft\Windows\CurrentVersion\Run`.
- Recently opened files:
	-  **NTUSER.DAT**
	- Located at: `\Software\Microsoft\Windows\CurrentVersion\Explorer\`.
- What user searched for:
	- **NTUSER.DAT**
	- `\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths`
		- or 'WordWheelQuery'.
- Evidence of user executions:
	- **NTUSER.DAT**
	- `\Software\Microsoft\Windows\Currentversion\Explorer\UserAssist\{GUID}\Count`.
		- Untwirl the folders.
## Conclusion


