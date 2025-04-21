---
{"dg-publish":true,"permalink":"/cards/blue-team/endpoint-security/windows-process-analysis/"}
---

[[cards/blue-team/endpoint-security/Endpoint Security\|Endpoint Security]]
### Introduction
---
Are programs/applications running within an operating system, the OS allocates system resources to these processes such as CPU, memory and input and output.

![windows/images/Windows Process Analysis.png|450](/img/user/cards/windows/images/Windows%20Process%20Analysis.png)
#### System (Windows) Processes (Core Functions)
---
**Note:** for all processes, always look out for mispelling or typos.

- [[cards/windows/System (Windows Process)\|System (Windows Process)]]
	-  ![Windows Process Analysis-2.png|350](/img/user/cards/blue-team/endpoint-security/images/Windows%20Process%20Analysis-2.png)
- [[cards/windows/Session Manager Subsystem (smss.exe)\|Session Manager Subsystem (smss.exe)]] launches processes at startup these are _wininit.exe_ and [[cards/windows/Server Runtime Subsystem (csrss.exe)\|csrss.exe]] 
	- **session** represents a single user log on Windows: each sessions has it's own processes and user environments.
	-  Unexpected registry entries for Subsystem (Look into notes for location)
	- ![Windows Process Analysis-3.png|450](/img/user/cards/blue-team/endpoint-security/images/Windows%20Process%20Analysis-3.png)
- [[cards/windows/Server Runtime Subsystem (csrss.exe)\|Server Runtime Subsystem (csrss.exe)]] manages console windows, importing DLLs for Windows API, and GUI tasks around shutdown.
	-  ![Windows Process Analysis-4.png|450](/img/user/cards/blue-team/endpoint-security/images/Windows%20Process%20Analysis-4.png)
	- As for instances: For **each active session** on the system (whether a physical login or a remote session), there will typically be **one instance of `csrss.exe`** running.

- **Windows Initialization (winnit.exe)** initialize all the things, starts at session 0. spawns _services.exe_ and _lsass.exe_. 
	- ![Windows Process Analysis-5.png|450](/img/user/cards/blue-team/endpoint-security/images/Windows%20Process%20Analysis-5.png)

 - **Service Control Manager (services.exe)** with a purpose of starting, stopping, and interacting with services.
	 - Sets the **LastKnownGood CurrentControlSet** registry value.
	 - ![Windows Process Analysis-6.png|450](/img/user/cards/blue-team/endpoint-security/images/Windows%20Process%20Analysis-6.png)

- [[cards/windows/Service Host (svchost.exe)\|Service Host (svchost.exe)]] hosting and managing windows services.
	- Runs with the -k parameter to differentiate instances/services
	- **Attackers**: spell their malware the same as svchost.exe, or something similar (typos) then the absence of the `-k` parameter.
	- ![Windows Process Analysis-7.png|450](/img/user/cards/blue-team/endpoint-security/images/Windows%20Process%20Analysis-7.png)
- **Local Security Authority Subsystem Service (lsass.exe)** 
	- Authenticating users (store credentials from Active Directory server and Kerberos Tickets).
	- Implementing local security policies and writing events to the security event log.
	- ![Windows Process Analysis-8.png|450](/img/user/cards/blue-team/endpoint-security/images/Windows%20Process%20Analysis-8.png)
- **Windows Login (winlogon.exe)** (Starts at session 1)
	- Manages login (sends to LSASS.exe) and logout procedures.
	- Loads user profiles (NTUSER.dat)
	- Responds to Secure Attention Sequence (SAS) the 'ctrl + alt + delete'
	- Should have a shell value of `explorer.exe` in the registry.
	- ![Windows Process Analysis-9.png|450](/img/user/cards/blue-team/endpoint-security/images/Windows%20Process%20Analysis-9.png)
- **Windows Explorer (explorer.exe)** provides GUI for files, folders, and system settings: taskbar, start menu, and desktop
	- It can have one or more `explorer.exe` process if the option is enabled: View -> Options -> View -> 'launch folder windows in seperating process'
	- Outbound TCP/IP connections
	- ![Windows Process Analysis-10.png|450](/img/user/cards/blue-team/endpoint-security/images/Windows%20Process%20Analysis-10.png)

### Questions to Ask during an Investigation
---
1. Parent Process: is this the expected process hierarchy?
2. Child Process: is this the expected child process hierarchy? winword.exe spawns a powershell?
3. Command Line Arguments: svchost `-k`?
4. Process Names: typos, lookalikes, copies.
5. User Account: Is this process running from the expected user account?
6. Image Path: is this process running the expected executable?
## Investigation
---

- [[SANS_DFPS_FOR508_v4.11_0624.pdf\|SANS cheatsheet for evil processes]]
- Identify PID of a network connection [[cards/blue-team/endpoint-security/Windows Network Analysis\|here]] then apply using `wmic`
- Task Manager is good but need to apply other available columns.
- [[cards/blue-team/endpoint-security/Windows Services Analysis\|Windows Services Analysis]] are background processes typically no UI.

7. **Display processes**: identify PID (or any value in column) of process:
```bash
tasklist /V
```

2. **View specific process**: and analyze loaded [[cards/windows/Dynamic Link Library\|.dll]]  with `/m` flag:
```Powershell
tasklist -m -fi "PID eq 2088"
```

3. **Show parent process ID**
```bash
wmic process where processid=4072 get name, parentprocessid, processid
```

- remove the `where processid=..` if for some reason you dont know the PID.

4. **Determine Parent Process ID**

```
wmic process get name, parentprocessid, processid | find "6624"
```

Output, remember we executed the malware using cmd:

![endpoint-security/images/Windows Process Analysis.png](/img/user/cards/blue-team/endpoint-security/images/Windows%20Process%20Analysis.png)
- additionally using the PPID we can add `wmic process get commandline` to know what arguments given.

---

Process Explorer is basically Task Manager and the above commands:

![Windows Process Analysisfile.png|450](/img/user/cards/blue-team/endpoint-security/images/Windows%20Process%20Analysisfile.png)
Interesting tab as well:

![Windows Process Analysis-1.png](/img/user/cards/blue-team/endpoint-security/images/Windows%20Process%20Analysis-1.png)
