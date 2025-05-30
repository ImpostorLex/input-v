---
{"dg-publish":true,"permalink":"/cards/active-directory/active-directory-authenticated-enumeration/","tags":["ad/red-team"]}
---

[[map-of-contents/active-directory\|active-directory]]
### Introduction
---
Demonstrate the techniques used once we have access to an authenticated account, the learning objectives are:

- AS-REP Roasting
- Using the `net` command for enumeration among other commands such as `set`, `hostname`, and `systeminfo`.
- Enumeration using the ActiveDirectory PowerShell module
- Enumeration using PowerSploit’s PowerView module
- Enumeration with BloodHound

## Prerequisites
---
- [[cards/active-directory/Active Directory Enumeration\|Active Directory Enumeration]]
- [[cards/active-directory/Active Directory\|Active Directory]] fundamentals.

## AS-REP Roasting in Two Phases
---
Similarly to [[cards/active-directory/Kerberos\|Kerberoasting]], AS-REP Roasting dumps user account hashes that have Kerberos pre-authentication disabled - in Kerberoasting, these accounts do not need to be a service account - the only requirement is that the “Do not require Kerberos pre-authentication” flag (`UF_DONT_REQUIRE_PREAUTH`) is set on the user account.

In Kerboros authentication the user's hash encrypts a timestamp which then the [[Key Distribution Center\|Key Distribution Center]] decrypt's the hash to verify the user's identity, if pre-authentication is disabled, the KDC skips this verification step and returns an encrypted AS-REP blob without confirming the user’s identity, the blob can be captured and cracked offline.

1. Enumerate vulnerable user accounts, accounts that have Kerberos pre-authentication disabled - this allows accounts that have pre-auth request a Kerberos ticket (AS-REP) without proving their identity..

**Rubeus**

if pre-authentication is disabled, the KDC skips this verification step and returns an encrypted AS-REP blob without confirming the user’s identity

```C
Rubeus.exe asreproast
```

**Impacket's GetNPUsers.py** (Linux/Windows)

It requires a list of users indicated by the `users.txt` and then output the collected hash in `hashes.txt`

```C
GetNPUsers.py tryhackme.loc/ -dc-ip 10.211.12.10 -usersfile users.txt -format hashcat -outputfile hashes.txt -no-pass
```

Output:

![Pasted image 20250525080934.png|center](/img/user/cards/active-directory/images/Pasted%20image%2020250525080934.png)


2. Capture and crack the retrieved AS-REP hashes offline. 


**hashcat**

```C
hashcat -m 18200 hashes.txt wordlist.txt
```

- `-m 18200`: Specifies the AS-REP Kerberos hash cracking mode.
- `hashes.txt`: Contains collected hashes from vulnerable accounts.
- `wordlist.txt`: Your chosen dictionary of possible passwords.


![Pasted image 20250525081335.png](/img/user/cards/active-directory/images/Pasted%20image%2020250525081335.png)
#### Mitigations

- Enforce Kerberos pre-authentication for all user accounts
- Strong, complex passwords slow down offline cracking
- Monitor anomalous AS-REP requests on the KDC
## Manual Enumeration
---
This requires a Windows machine using CMD and Powershell commands. This will include exploring token privileges, identifying domain and local users (including service accounts), analyzing logged-in sessions, and collecting valuable data from environment variables and the Windows Registry.


This assumes a compromised Windows machine either with/without Graphical User Interface (GUI) but first we must know our environment: including knowing our user, host name machine, and more.

In our case, we already have a valid account - we can now logon to the account using either ssh (if available) or remote desktop protocol.

```C
ssh asrepuser1@10.211.12.20
```

`whoami`

![Pasted image 20250525081919.png](/img/user/cards/active-directory/images/Pasted%20image%2020250525081919.png)

`whoami /all`

Reveals account's Security Identifier (SID), group memberships, and account privileges.

![Pasted image 20250525082045.png](/img/user/cards/active-directory/images/Pasted%20image%2020250525082045.png)

Most interesting privileges:

- `SeImpersonatePrivilege` lets your account impersonate other logged-in users - allows to get SYSTEM shell via token manipulation.
- `SeAssignPrimaryTokenPrivilege`: This privilege permits a process to assign the primary token of another user to a new process. It is used in conjunction with the SeImpersonatePrivilege privilege.
- `SeBackupPrivilege`: This privilege lets users read any file on the system, ignoring file permissions. Consequently, attackers can use it to dump sensitive files like the SAM or SYSTEM hive.
- `SeRestorePrivilege`: This privilege grants the ability to write to any file or registry key without adhering to the set file permissions. Hence, it can be abused to overwrite critical system files or registry settings.
- `SeDebugPrivilege`: This privilege allows the account to attach a debugger to any process. As a result, the attacker can use this privilege to dump memory from LSASS and extract credentials or even inject malicious code into privileged processes.
#### System and Domain Information
---
To find out more about the system you're on, start with a few basic commands.

**1. `hostname`**  
This command shows the computer’s hostname. The name might give clues about the system's role. For example, names like **"dc"** may suggest a domain controller, while **"pc"** followed by numbers could be a workstation in a domain.

**2. `systeminfo`** _(requires admin privileges)_  
This command provides a lot of details — OS version, installed hotfixes, and domain or workgroup membership. You can filter it for specific info:

- `systeminfo | findstr /B "OS"` — shows the OS name and version.
- `systeminfo | findstr /B "Domain"` — shows the domain name if joined to Active Directory.

**3. `set`**  
This displays all environment variables. Useful ones include:

- `USERDOMAIN` — shows the domain the user belongs to. If the computer isn’t in a domain, this usually matches the computer name. In PowerShell, use `Get-ChildItem Env:` or `dir env:` instead of `set`.
#### Enumerating Users and Groups with NET commands (CMD)
---
At this stage, we should have a clear understanding of our **identity on the system** — including the **groups** we belong to, the **privileges** we hold, and whether the system (or user) is part of an **Active Directory domain** or operating in a standalone/workgroup environment.

`NET` is a suite of commands for viewing and managing networked resources, here are the important commands:

- `net user /domain` -- list all users in the domain.
- `net user` -- list all users locally.

The `net user <username> /domain` returns the following information: **full name**, **account status**, **information about password, group memberships, and last logon time**:

![Pasted image 20250525092818.png](/img/user/cards/active-directory/images/Pasted%20image%2020250525092818.png)
##### Listing domain groups
---
At this phase, we now know the list of active or inactive domain accounts, it's time to list all domain groups, using `net group /domain`:

![Pasted image 20250525092943.png](/img/user/cards/active-directory/images/Pasted%20image%2020250525092943.png)

Here are some worth noting groups:

- **Domain Admins** and **Administrators** can hold the keys to the whole Active Directory
- **Enterprise Admins** play a key role in a multi-domain forest
- **Server Operators** and **Backup Operators** are privileged built-in accounts that are worth inspecting
- Any group with “Admin” in its name (e.g., “SQL Admins”) could be worth targeting.

Display members of the group both user accounts and machine accounts since Domain Computers, and Domain Controllers can hold machine accounts:

```C
net group "<Group Name>" /domain
```

- `$` -- indicates a machine account at the very end of a name.
- Like the previous command to see the local groups we can use `net localgroup`.

**List all accounts including domain accounts that is part of a specific group**

```C
net localgroup Administrators
```

![Pasted image 20250525093759.png](/img/user/cards/active-directory/images/Pasted%20image%2020250525093759.png)
### Logged-on Users and Sessions
---
Identifying who else uses the machine, either via Remote Desktop Protocol (RDP), open sessions by users (Physically accessing the machine, network sessions , and more), service accounts running scheduled tasks, and more.

```C
query user // or quser
```

![Pasted image 20250525094125.png](/img/user/cards/active-directory/images/Pasted%20image%2020250525094125.png)

Having an administrator account logged on to the machine is huge, we can find their credentials or tokens in memory by dumping [[LSASS.exe\|LSASS.exe]] to get the password hash or Kerberos ticket. Or  the attacker can impersonate their session token provided their account has `SeImpersonatePrivilege`.

Additionally we can navigate to the `C:\Users\` directory to see users that logged in at least once in this machine.

**Analyzing system activity and users** (_Requires elevated privileges_)

- `tasklist` displays a list of currently running processes. You can use `tasklist /V` for verbose task information.
- `net session` lists the SMB sessions between the computer and other computers on the network. It requires administrator privileges to run.

#### Identifying Service Accounts
---
 Service accounts are local and domain accounts used by applications and services, usually have enough privileges to get the job one - most of the time service accounts have static passwords due to the hassle of changing it every time and the possibility of causing problem because of a simple expired password.
##### Search Using WMIC
---
Using `wmic` to display all services and the account for each service, additionally wmic provides a very long row of information to combat this we can specify specific columns:

```C
wmic service get Name,StartName
```

![Pasted image 20250525094857.png](/img/user/cards/active-directory/images/Pasted%20image%2020250525094857.png)

The **StartName** tells you **which account a service runs under**. This is important because services often run in the background with specific privileges.

**Common Built-in Accounts**

Most services run under standard system accounts like:

- `LocalSystem` — very high privileges on the local machine
- `NT AUTHORITY\LocalService` — limited local privileges
- `NT AUTHORITY\NetworkService` — limited local privileges but can access network resources
- `NT SERVICE\SomeServiceName` — a **virtual service account** created just for that service

Sometimes, a service will run under a **domain account**, like:

```C
CORP\svc-backup
```

Alternatively, if the command prompt is not available, we can achieve the same thing with PowerShell:

```C
Get-WmiObject Win32_Service | select Name, StartName
```
##### Searching Using SC
---
SC is a command-line program that communicates with services and the Service Control Manager.

`sc query state= all` will return all services on a Windows system; however, it requires administrator privileges to run.

The output is long so you might want to filter for keywords only using:

```C
sc query state= all | find "Keyword" // find "DHCP"
```

Then use `sc qc <service_name>` to learn more about the service:

![Pasted image 20250525095452.png](/img/user/cards/active-directory/images/Pasted%20image%2020250525095452.png)
#### Watching the Environment and Registry
---
Read this about [[cards/windows/Windows Registry\|Windows Registry]]
##### Saved Auto-Logon Credentials
---
Normally, when a system boots up, it waits for a user to **manually enter login credentials**. But in **misconfigured or test environments**, the system might be set to **auto-logon**, meaning credentials are stored and used automatically.

```C
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
```

- **`AutoAdminLogon`** — if set to `1`, auto-logon is enabled
- **`DefaultUserName`** — the user account that auto-logs in.
- **`DefaultDomainName`** — the domain (if any) for the user.
- **`DefaultPassword`** — the password saved in plaintext (if present).

Another registry hive to look out for that contains hashed password is `HKLM/Security/Cache` although it requires administrator access.
#### Installed Applications
---
You can run `reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall` to quickly get a list of installed applications. This list can be handy, especially if you find a server with known default credentials.
#### Searching the Registry
---
Search for a specific instance of a keyword such as:

```C
reg query HKLM /f "password" /t REG_SZ /s
```
#### Scheduled Tasks
---
You can list all scheduled tasks using `schtasks /query`. On a side note, you can use `schtasks` to create a new scheduled task using `/create` or run an existing scheduled task with `/run`.
## Enumeration with BloodHound
---
The most effective Active Directory enumeration tool up to date developed initially by SpecterOps. It is used by both attackers and defenders to proactively identify misconfigurations, excessive privileges, and excessive pathways.

### The Two-Stage Attack Model with BloodHound
---
**BloodHound** introduced a powerful two-step way to attack Active Directory environments:

**Stage 1: Enumeration**

Attackers use tools like **SharpHound** or **BloodHound-python** to collect data about the AD environment. This includes:

- User sessions
- Group memberships
- Access Control Lists (ACLs)
- Delegation settings

Even if defenders detect and stop this early, attackers now have a rich offline dataset to analyze.

**Stage 2: Targeted Attack**

Using BloodHound offline, attackers analyze the data to find **precise, efficient paths** to high-value targets like Domain Admins.

When they return to the network, they can move laterally and escalate privileges quickly — often in just minutes, faster than defenders can react.
#### SharpHound
---
SharpHound is specifically the data collection component of BloodHound. It gathers Active Directory (AD) information, which BloodHound then visualises in attack graphs.

 It enumerates key AD elements such as:

- Group memberships
- Session data
- Access Control Lists (ACLs)
- Domain trusts
- Privileged relationships (like local administrator rights)
##### Types
---

- **`SharpHound.exe`**: This is a Windows executable designed for standard enumeration on domain-joined Windows machines. Due to its versatility and robust functionality, it is currently the recommended method.
- **`AzureHound.ps1`**: A PowerShell script focused specifically on Azure Entra ID environments. It enables enumerating cloud-specific configurations and identities seamlessly into hybrid AD scenarios.
- **`SharpHound.ps1` (Deprecated)**: Previously a PowerShell variant used for stealth operations, it is particularly useful for loading scripts directly into memory to avoid antivirus detection. However, recent releases have discontinued support, favouring the executable and Python-based approaches.
#### BloodHound
---
Besides SharpHound, there’s **BloodHound.py**, a Python-based collector ideal for Linux or Python-friendly environments. It can enumerate AD domains using credentials, NTLM hashes, or Kerberos tickets, and outputs JSON or ZIP files compatible with BloodHound’s GUI.

**Note:** BloodHound and SharpHound versions should match, as older SharpHound data may not import into newer BloodHound versions. Also, BloodHound.py isn’t officially supported by the BloodHound team.
### Executing SharpHound
---
We can run SharpHound on a domain-joined Windows machine; Although can be blocked by Windows Defender:

```C
C:\> .\SharpHound.exe --CollectionMethods All --Domain tryhackme.loc --ExcludeDCs
```

- **`--CollectionMethods All`**: Specifies that all data collection methods should be used
- **`--Domain tryhackme.loc`**: Targets the specified domain for data collection
- **`--ExcludeDCs`**: Excludes domain controllers from the collection process to reduce detection risk
### BloodHound.py
---
You can use a Linux system if you don’t have access to a domain-joined Windows machine. In this case, we will be running the `bloodhound-python`

```C
bloodhound-python -u asrepuser1 -p qwerty123! -d tryhackme.loc -ns 10.211.12.10 -c All --zip
```

- **`-u username`**: Specifies the username for authentication
- **`-p password`**: Specifies the password for authentication
- **`-d tryhackme.loc`**: Targets the specified domain for data collection
- **`-ns`**: Specifies the IP address of a DNS server
- **`-c All`**: Uses all available collection methods
- **`--zip`**: Compresses the output into a ZIP archive for easy import into BloodHound

##### Operational Security Considerations
---
When conducting assessments, be aware that data collection tools like **SharpHound** and **BloodHound.py** may trigger security alerts. To minimise detection:

- Use the **`--ExcludeDCs`** flag to avoid querying domain controllers
- Employ stealthier collection methods, such as **`DCOnly`**, to limit interactions with sensitive systems
- Run collectors from systems with appropriate antivirus exclusions or non-domain-joined machines using the `runas` command with the `/netonly` flag to authenticate without joining the domain

Graphing the gathered data in BloodHound CE now as a web application:

By running SharpHound or bloodhound-python with the `--zip` flag:

1. Go to the **Administration** tab in the left-hand navigation menu
2. Scroll to the **File Ingest** section
3. Upload your ZIP file directly through the browser

Click on the **Explore** tab to view the visual Active Directory graph.

- **Nodes**: Represent users, computers, groups, etc.
- **Edges**: Represent relationships and permissions between nodes.

At the search bar find 'ASREPUSER1' user account and click the **node** to view it's properties:

![Pasted image 20250525103056.png|450](/img/user/cards/active-directory/images/Pasted%20image%2020250525103056.png)

**Node Information Breakdown**

- **Object information** – summary details of the object, such as name, type, and domain
- **Sessions** – active logon sessions associated with the object
- **Member of** – AD groups the object belongs to
- **Local admin privileges** – machines where the object has local administrator rights
- **Execution privileges** – rights such as RDP or equivalent permissions
- **Outbound object control** – rights the object has over other objects
- **Inbound object control** – rights other objects have over this object

To use BloodHound’s built-in queries:

1. Click **Cypher** in the top menu
2. Click the **folder icon** to browse prebuilt queries (e.g., “All Domain Admins”)

![Pasted image 20250525103150.png](/img/user/cards/active-directory/images/Pasted%20image%2020250525103150.png)

## Enumeration With PowerShell's Active Directory and PowerView Modules
---
The `ActiveDirectory` module is available on domain controllers. For other workstations and servers, you need to download Remote Server Administration Tools (RSAT) for Windows. Alternatively, certain repositories make the ActiveDirectory module available for easy installation without installing the whole RSAT.


`Get-Module -ListAvailable ActiveDirectory` in PowerShell. If it is available, or import it using `Import-Module ActiveDirectory` assuming it is already installed.
### User Enumeration
---
Retrieve all user with:

```C
Get-ADUser -Filter *
```

 Get any user’s details by running `Get-ADUser -Identity <username>`; if you want to list all this user’s properties, you can expand your command to `Get-ADUser -Identity <username> -Properties *`

Specific interesting columns only:

```C
Get-ADUser -Identity Administrator -Properties LastLogonDate,MemberOf,Title,Description,PwdLastSet
```
 
 Find accounts with `admin` in them, you can issue `Get-ADUser -Filter "Name -like '*admin*'"`.
### Group Enumeration
----
Get all groups by running and created in Active Directory:

```C
Get-ADGroup -Filter *
```

You can get all members of a group using `Get-ADGroupMember -Identity "Group Name"`. For example, `Get-ADGroupMember -Identity "Remote Management Users"` will show you accounts that belong to the Remote Management Users group, while `Get-ADGroupMember -Identity "Domain Admins"` will reveal accounts with Domain Admins privileges.

### Computer Enumeration
---

```C
Get-ADComputer -Filter *
```

- `Get-ADComputer -Filter * | Select Name, OperatingSystem` will only display the computer name and operating system.
- `Get-ADDefaultDomainPasswordPolicy` will reveal the password policy enforced in the domain.
## Enum with PowerView
---
It is like an evolution of tools such as `net user` and `net group` to use it we must first` Import-Module .\PowerView.ps1` should be no error displayed

- `Get-DomainUser`
- `Get-DomainUser *admin*` display usernames with admin.
#### Group Enumeration
---
List of all groups created in Active Directory:

- `Get-DomainGroup`
- `Get-DomainGroup "*admin*"` shows group with admin on their name.
### Computer Enumeration
---
????

```C
Get-DomainComputer
```

- `Get-DomainUser -AdminCount` will return the list of domain users that have administrator privileges.
- `Get-DomainUser -SPN` lists the accounts with non-null service principal names (SPN). These can be considered for Kerberoasting attacks.

Additionally with all the previous command - we can use `Get-Object` to filter out for key values:

```
Get-DomainGroup "*admin*" | Select-Object "samaccountname"
```