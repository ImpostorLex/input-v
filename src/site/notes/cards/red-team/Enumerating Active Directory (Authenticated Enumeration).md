---
{"dg-publish":true,"permalink":"/cards/red-team/enumerating-active-directory-authenticated-enumeration/","tags":["red-team/ad"]}
---

~ [[atlas/red-team\|red-team]] | ~ [[cards/active-directory/active-directory\|active-directory]]
### Introduction 
---
Once we successfully obtained our first set of AD credentials; you can start enumerating various details about the AD setup and structure with authenticated access, even super low-privileged access.

![cards/red-team/images/Enumerating Active Directory.png|500](/img/user/cards/red-team/images/Enumerating%20Active%20Directory.png)

[Figure 1.0]: Once an attack path shown by the enumeration phase has been exploited, enumeration is again performed from this new privileged position.

#### Key Topics
---

- [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)#Credential Injection\|How to inject AD credentials into memory using runas.exe to authenticate without a domain-joined host.]]

- [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)#Enumeration through Command Prompt\|Steps to verify credentials, configure DNS, and access SYSVOL from non-domain-joined systems.]]

- [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)#IP vs Hostnames\|Explains why hostname-based connections use Kerberos while IP-based ones force NTLM authentication, and when each is useful for stealth.]]

- [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)#Using Injected Credentials\|How injected credentials enable authentication to networked apps like MSSQL and NTLM-secured web portals.]]

- [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)#Enumeration through Microsoft Management Console\|Using MMC and RSAT AD Snap-ins with runas to graphically enumerate users, groups, and OUs.]]

- [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)#Users and Computers\|Exploring AD organizational units, users, groups, and systems visually using MMC.]]

- [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)#Enumeration through Command Prompt (net commands)\|Using net commands to list users, groups, and domain password policies for additional spraying or structure discovery.]]

- [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)#PowerShell\|Leveraging AD-RSAT PowerShell cmdlets for deeper queries and selective enumeration of AD objects, groups, and users.]]

- [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)#Bloodhound\|Collecting and visualizing AD relationships and attack paths using Sharphound and Bloodhound.]]

- [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)#Session Data Only\|Rerunning Sharphound with the Session collection method for updated session data during ongoing engagements.]]

- [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)#Conclusion\|Summarizes the post-breach enumeration process using both built-in and advanced tools for domain mapping.]]

## Prerequisites

- [[cards/active-directory/active-directory\|active-directory]] | [[cards/active-directory/Active Directory\|Active Directory]]
- Need to know: [[cards/red-team/Breaching Active Directory\|Breaching Active Directory]]

## Credential Injection
---
You can do so much Active Directory enumeration from a Kali machine but if you want to do in-depth enumeration and even exploitation you need a **Windows machine**. One of these built-in tools, called the `runas.exe` binary.
### What is Runas?
---
Runas is the best solution when you have/found Active Directory credentials but nowher to log in with them.

In a security assessments, often times you will have network access and have just discovered AD credentials but have no means or privileges to a create a new domain-joined machine.

To inject the credentials into memory, format `username:password`:

```C
runas.exe /netonly /user:<domain>\<username> cmd.exe // Password prompt later
```

- **/netonly** - Since we are not domain-joined, we want to load the credentials for network authentication but not authenticate against a domain controller. So commands executed locally on the computer will run in the context of your standard Windows account, but any network connections will occur using the account specified here.

	- "Run this program under my local account, but _when this process makes outbound network requests_ (e.g., SMB, RPC) use the credentials I supply for network authentication." 
	- The credentials are stored in the process’ session (in-memory) and will be used for any network connections that support passing credentials. So it's important that we inject the correct credentials.

- **/user** - Here, we provide the details of the domain and the username. It is always a safe bet to use the Fully Qualified Domain Name (FQDN) instead of just the NetBIOS name of the domain since this will help with resolution.

- **cmd.exe** - This is the program we want to execute once the credentials are injected. This can be changed to anything, but the safest bet is cmd.exe since you can then use that to launch whatever you want, with the credentials injected.

**Note:** If you use your own Windows machine, If you launch an elevated cmd before `runas`, local commands will run elevated; network authentication still uses injected credentials.

## Enumeration through Command Prompt
---
Before proceeding, the next step is only needed when you use your **own Windows machine** (Non-domain joined). On a domain joined laptop those things are automatic.

After providing the password (previously from `runas` command), a new command prompt window will open. Now you still need to verify your credentials, the best way is listing the `SYSVOL`. Any AD account, no matter how low-privileged, can read the contents of the `SYSVOL` directory.

Before listing [[cards/active-directory/SYSVOL\|SYSVOL]], you need to configure DNS. Sometimes DNS will be automatically configured through DHCP or the VPN connection. Your safest bet for a DNS server is the domain controller:

```powershell
$dnsip = "<DC IP>"
$index = Get-NetAdapter -Name 'Ethernet' | Select-Object -ExpandProperty 'ifIndex'
Set-DnsClientServerAddress -InterfaceIndex $index -ServerAddresses $dnsip
```

Replace **'Ethernet'** with whatever the interface is in the network. You can verify that DNS is working by running the following:

```PowerShell
nslookup za.tryhackme.com
```

This should resolve to the DC IP since this is where the FQDN is being hosted. Now that DNS is working, you can finally test your credentials using the following command to force a network-based listing:

```PowerShell
dir \\za.tryhackme.com\SYSVOL\
```

### IP vs Hostnames
---

**Question:** _Is there a difference between_ _`dir \\za.tryhackme.com\SYSVOL` and `dir \\<DC IP>\SYSVOL`_ _and why the big fuss about DNS?_

When you provide the hostname, network authentication will attempt first to perform Kerberos authentication since it uses hostnames embedded in the tickets, if you provide the IP instead, you can force the authentication type to be NTLM. This is worth noting for being stealthy during a Red team assessment. In some organization they will be monitoring for OverPass- and Pass-The-Hash Attacks. Forcing NTLM authentication is a good trick for this.

### Using Injected Credentials
---
After successfully injecting credentials into memory. With the `/netonly` option, all network communication will use these injected credentials for authentication including all applications executed from the command prompt Window.

Have you ever had a case where an MS SQL database used Windows Authentication, and you were not domain-joined? Start MS SQL Studio from that command prompt; even though it shows your local username, click Log In, and it will use the AD credentials in the background to authenticate! You can even use this to authenticate to web applications that use NTLM Authentication.

## Enumeration through Microsoft Management Console
---
**Note:** THMJMP1 can be seen as a jump host into this environment, simulating a foothold that you have achieved. 

Read: [[Microsoft Management Console (MMC)\|Microsoft Management Console (MMC)]].

Enumeration using MMC with the Remote Server Administration Tools (RSAT) AD Snap-ins though it requires installation:

1. Press **Start**
2. Search **"Apps & Features"** and press enter
3. Click **Manage Optional Features**
4. Click **Add a feature**
5. Search for **"RSAT"**
6. Select "**RSAT: Active Directory Domain Services and Lightweight Directory Tools"** and click **Install**

You can start MMC by using the Windows Start button, searching run, and typing in MMC. If you just run MMC normally, it would not work as our computer is not domain-joined, and our local account cannot be used to authenticate to the domain.

![Enumerating Active Directory-8.jpg](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-8.jpg)

This is where the Runas window from the previous task comes into play. In that window, you can start MMC, which will ensure that all MMC network connections will use our injected AD credentials.  

In MMC, you can now attach the AD RSAT Snap-In (Do this for domain-joined machine as well):

1. Click **File** -> **Add/Remove Snap-in**
2. Select and **Add** all three Active Directory Snap-ins (It should starts with"Active Directory .....")
3. Click through any errors and warnings  

4. Right-click on **Active Directory Domains and Trusts** and select **Change Forest**
5. Enter _za.tryhackme.com_ as the **Root domain** and Click **OK**
6. Right-click on **Active Directory Sites and Services** and select **Change Forest**
7. Enter _za.tryhackme.com_ as the **Root domain** and Click OK
8. Right-click on **Active Directory Users and Computers** and select **Change Domain**
9. Enter _za.tryhackme.com_ as the **Domain** and Click **OK**
10. Right-click on **Active Directory Users and Computers** in the left-hand pane  

11. Click on **View** -> **Advanced Features**  
    
If everything up to this point worked correctly, your MMC should now be pointed to, and authenticated against, the target Domain:

![Enumerating Active Directory-7.jpg|450](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-7.jpg)

- If machine is domain-joined RSAT should be preinstalled and can run MMC normally no `runas` required from previous task.
- if machine is not domain-joined: Install RSAT, then use `runas /netonly /user:DOMAIN\user mmc.exe` (or start an elevated CMD from your existing `runas` window and launch `mmc`) so MMC’s **network** calls authenticate with the injected AD credentials.
- If your machine is domain‑joined but you logged in with a local account (or a different domain account without rights), `runas /netonly` can still help to force MMC to use other AD creds for network authentication.
### Users and Computers
---
Looking at the Active Directory structure. Expand the snap-in and expand the za domain to see the Initial Organisational Unit (OU) structure:

![Enumerating Active Directory-3.jpg|400](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-3.jpg)

In the **People** directory, you can see that the users are divided according to the department OUs. Clicking each 'directory' shows it's users:

![Enumerating Active Directory-4.jpg|450](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-4.jpg)

Clicking on any of these users will allow us to review all of their properties and attributes. You can also see what groups they are a member of:

![Enumerating Active Directory-5.jpg](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-5.jpg)

You can also use MMC to find hosts in the environment. If you click on either Servers or Workstations, the list of domain-joined machines will be displayed:

![Enumerating Active Directory-6.jpg|450](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-6.jpg)

If you had the relevant permissions, you could also use MMC to directly make changes to AD, such as changing the user's password or adding an account to a specific group. 

**Benefits**

- The GUI provides an excellent method to gain a holistic view of the AD environment.
- Rapid searching of different AD objects can be performed.  
- It provides a direct method to view specific updates of AD objects.
- If we have sufficient privileges, we can directly update existing AD objects or add new ones.

**Drawbacks**

- The GUI requires RDP access to the machine where it is executed.
- Although searching for an object is fast, gathering AD wide properties or attributes cannot be performed.

## Enumeration through Command Prompt
---
The CMD has a built-in command that we can use to enumerate information about AD. The `net` command is a handy tool to enumerate information about the local system and AD.
### Users

You can use the `net` command to list all users in the AD domain by using the `user` sub-option:

```C
net user /domain
```

This is useful for determining the size of the domain:

![Enumerating Active Directory-2.jpg|500](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-2.jpg)
[Figure 6.0: Output of `net user /domain` command]:

Enumerate a single user account:

```C
net user zoe.marshall /domain
```

![Enumerating Active Directory-1.jpg|450](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-1.jpg)
[Figure 7.0: Info about specific user]:

**Note:** If the user is only part of a small number of AD groups, this command will be able to show us group memberships. However, usually, after more than ten group memberships, the command will fail to list them all.
### Groups

Enumerating groups in the domain:

```C
net group /domain
```

![Enumerating Active Directory.jpg|500](/img/user/cards/red-team/images/Enumerating%20Active%20Directory.jpg)
[Figure 8.0: Info about groups in the domain]:

You could also enumerate more details such as membership to a group by specifying the group in the same command:

```C
C:\>net group "Tier 1 Admins" /domain
The request will be processed at a domain controller for domain za.tryhackme.com

Group name     Tier 1 Admins
Comment

Members

-------------------------------------------------------------------------------
t1_arthur.tyler          t1_gary.moss             t1_henry.miller
t1_jill.wallis           t1_joel.stephenson       t1_marian.yates
t1_rosie.bryant
The command completed successfully.
```

### Password Policy

Enumerate the password policy of the domain:

```c
C:\>net accounts /domain
The request will be processed at a domain controller for domain za.tryhackme.com

Force user logoff how long after time expires?:       Never
Minimum password age (days):                          0
Maximum password age (days):                          Unlimited
Minimum password length:                              0
Length of password history maintained:                None
Lockout threshold:                                    Never
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        PRIMARY
The command completed successfully.
```

You cab use `net accounts /domain` to capture domain password policy (min/max age, lockout thresholds, length of password history) to tune password-spray timing.

This is great for additional password spraying attacks, guess what single passwords you should use and how many attacks can you run before risking account lockouts. However it's important to note blind password spraying can cause account lock outs since we don't know how many recent failed attempts the target already did.

More info about the net command [here](https://docs.microsoft.com/en-us/troubleshoot/windows-server/networking/net-commands-on-operating-systems)

**Benefits**

- No additional or external tooling is required, and these simple commands are often not monitored for by the Blue team.
- No GUI needed.
- VBScript and other macro languages that are often used for phishing payloads support these commands natively so they can be used to enumerate initial information regarding the AD domain before more specific payloads are crafted.

**Drawbacks**

- The `net` commands must be executed from a domain-joined machine. If the machine is not domain-joined, it will default to the WORKGROUP domain.
- The `net` commands may not show all information. 

## PowerShell
---
It has all the standard functionaly of Command prompt and it provides cmdlets (pronounced command-lets), which are .NET classes to perform a specific function. You can write our own cmdlets.

Installing the AD-RSAT provides 50+ cmdlets.

**Enumerate AD users:**

```PowerShell
Get-ADUser -Identity gordon.stevens -Server za.tryhackme.com -Properties *
```

- Identity - The account name that we are enumerating
- Properties - Which properties associated with the account will be shown, * will show all properties
- Server - Since we are not domain-joined, we have to use this parameter to point it to our domain controller

The `-Filter` allows more control over enumeration and use the `Format-Table` to display the results elegantly.

```PowerShell
PS C:\> Get-ADUser -Filter 'Name -like "*stevens"' -Server za.tryhackme.com | Format-Table Name,SamAccountName -A

Name             SamAccountName
----             --------------
chloe.stevens    chloe.stevens
samantha.stevens samantha.stevens
```

**Enumerate AD groups:** (remember you can use `-Properties *`)

```PowerShell
Get-ADGroup -Identity Administrators -Server za.tryhackme.com
```

**Enumerate AD group membership:**

```PowerShell
Get-ADGroupMember -Identity Administrators -Server za.tryhackme.com
```

Queries and return members of the AD group named **Administrators.**

**Enumerate AD Objects:**

For example, if we are looking for all AD objects that were change after a specific date:

```PowerShell
$ChangeDate = New-Object DateTime(2022, 02, 28, 12, 00, 00)
Get-ADObject -Filter 'whenChanged -gt $ChangeDate' -includeDeletedObjects -Server za.tryhackme.com
```

Performing password spraying attack buy avoiding user with already bad attempts:

```PowerShell
Get-ADObject -Filter 'badPwdCount -gt 0' -Server za.tryhackme.com
```

**Retrieve additional information about the specific domain:**

```PowerShell
Get-ADDomain -Server za.tryhackme.com
```

**Altering AD Objects:**

The great thing about the AD-RSAT cmdlets is that some even allow you to create new or alter existing AD objects. However, our focus for this network is on enumeration. Creating new objects or altering existing ones would be considered AD exploitation

Forcing password change:

```PowerShell
Set-ADAccountPassword -Identity gordon.stevens -Server za.tryhackme.com -OldPassword (ConvertTo-SecureString -AsPlaintext "old" -force) -NewPassword (ConvertTo-SecureString -AsPlainText "new" -Force)
```

## Bloodhound
---
The most powerful AD enumeration tool up to date. 
### Sharphound
---
The enumeration tool of Bloodhound that can then be visualized in Bloodhound while Bloodhound is used to display the actual graphs.

- **Sharphound.ps1:** This version is good to use with RATs since the script can be loaded directly into memory. Though latest release of Sharphound has stopped releasing Powershell script version.
- **Sharphound.exe**
- **Azurehound.ps1:** For Azure (Microsoft Cloud Computing Services) instances. Bloodhund can ingest data enumerated from Azure.

When using these collector scripts on an assessment, **there is a high likelihood that these files will be detected as malware** and raise an alert to the blue team. This is again where our Windows machine that is non-domain-joined can assist. You can use the `runas` command from the previous task and point Sharphound to a Domain Controller.

```C
Sharphound.exe --CollectionMethods <Methods> --Domain za.tryhackme.com --ExcludeDCs
```

- **CollectionMethods** - Determines what kind of data Sharphound would collect. The most common options are `Default` or `All`. Use _Session collection_ method to speed up process when running again.
- **Domain** - Here, we specify the domain we want to enumerate. In some instances, you may want to enumerate a parent or other domain that has trust with your existing domain
- **ExcludeDCs** -This will instruct Sharphound not to touch domain controllers, which reduces the likelihood that the Sharphound run will raise an alert.

After running Sharphound, it will result into a `.zip` file ready to be ingested in Bloodhound. To extract the zip file:

```PowerShell
scp <AD Username>@THMJMP1.za.tryhackme.com:C:/Users/<AD Username>/Documents/<Sharphound ZIP> .
```

Upload to Bloodhound by drag and dropping.

**Searching for a user:**

![Enumerating Active Directory-16.png](/img/user/cards/red-team/Enumerating%20Active%20Directory-16.png)

Clicking the numbers beside each information:

![Enumerating Active Directory-17.png|500](/img/user/cards/red-team/Enumerating%20Active%20Directory-17.png)

The icons are called nodes, and the lines are called edges and taking a look at the **"Analysis** tab these are the queries made by the Bloodhound developers:

![Enumerating Active Directory-19.png](/img/user/red/Enumerating%20Active%20Directory-19.png)

There is an AD user account with the username of **T0_TINUS.GREEN**, that is a member of the group **Tier 0 ADMINS**. But, this group is a nested group into the **DOMAIN ADMINS** group, meaning all users that are part of the **Tier 0 ADMINS** group are effectively DAs.

Since the **ADMINISTRATOR** account is a built-in account, we would likely focus on the user account instead.

**Enumerating attack path:**

![Enumerating Active Directory-20.png](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-20.png)

Then a **"Start Node"** and **"Target Node:**

![Enumerating Active Directory-21.png](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-21.png)

**Note:** It will show "No Results founds", if there is no attack path found but it may also due to a Bloodhound/Sharphound mismatch version.

It shows that one of the **T1 ADMINS, ACCOUNT,**  broke the tiering model by using their credentials to authenticate to **THMJMP1**, which is a workstation. It also shows that any user that is part of the **DOMAIN USERS** group, including our AD account, has the ability to RDP into this host.

We could do something like the following to exploit this path:

1. Use our AD credentials to RDP into **THMJMP1**.
2. Look for a privilege escalation vector on the host that would provide us with Administrative access.
3. Using Administrative access, we can use credential harvesting techniques and tools such as Mimikatz.
4. Since the T1 Admin has an active session on **THMJMP1**, our credential harvesting would provide us with the NTLM hash of the associated account.
### Session Data Only
---
The structure of AD does not change very often in large organisations. There may be a couple of new employees, but the overall structure of OUs, Groups, Users, and permission will remain the same.

However, the one thing that does change constantly is active sessions and LogOn events. A good approach is to execute Sharphound with the "All" collection method at the start of your assessment and then execute Sharphound at least twice a day using the "Session" collection method. This will provide you with new session data and ensure that these runs are faster since they do not enumerate the entire AD structure again.


### Questions and Problems
---
## Conclusion


