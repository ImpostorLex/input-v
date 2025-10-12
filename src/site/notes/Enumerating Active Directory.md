---
{"dg-publish":true,"permalink":"/enumerating-active-directory/"}
---

~ [[atlas/red-team\|red-team]] | ~ [[atlas/active-directory\|active-directory]]
### Introduction 
---
Once we successfully obtained our first set of AD credentials; you can start enumerating various details about the AD setup and structure with authenticated access, even super low-privileged access.

![Enumerating Active Directory.png|500](/img/user/Enumerating%20Active%20Directory.png)

[Figure 1.0]: Once an attack path shown by the enumeration phase has been exploited, enumeration is again performed from this new privileged position.

#### Key Topics
---

## Prerequisites

- [[atlas/active-directory\|active-directory]] | [[cards/active-directory/Active Directory\|Active Directory]]
- Need to know: [[cards/red-team/Breaching Active Directory\|Breaching Active Directory]]

## Credential Injection
---
Let's first read:

> _"If you know the enemy and know yourself, you need not fear the results of a hundred battles. If you know yourself but not the enemy, for every victory gained you will also suffer defeat."_ - Sun Tzu, Art of War.

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

**Note:** If you use your own Windows machine, you should make sure that you run your first Command Prompt as Administrator. This will inject an Administrator token into CMD. If you run tools that require local Administrative privileges from your Runas spawned CMD, the token will already be available. This does not give you administrative privileges on the network, but will ensure that any local commands you execute, will execute with administrative privileges.

## Enumeration through Command Prompt
---
Before proceeding, the next step is only needed when you use your own Windows machine (Non-domain joined). On a domain joined laptop those things are automatic.

After providing the password (previously from `runas` command), a new command prompt window will open. Now you still need to verify your credentials, the best way is listing the `SYSVOL`. Any AD account, no matter how low-privileged, can read the contents of the `SYSVOL` directory.



Before listing SYSVOL, you need to configure DNS. Sometimes DNS will be automatically configured through DHCP or the VPN connection. Your safest bet for a DNS server is the domain controller:

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

There is a difference and it boils down to the authentication method being used. When you provide the hostname, network authentication will attempt first to perform Kerberos authentication since it uses hostnames embedded in the tickets, if you provide the IP instead, you can force the authentication type to be NTLM. This is worth noting for being stealthy during a Red team assessment. In some organization they will  be monitoring for OverPass- and Pass-The-Hash Attacks. Forcing NTLM authentication is a good trick for this.

### Using Injected Credentials
---
After successfully injecting credentials into memory. With the `/netonly` option, all network communication will use these injected credentials for authentication including all applications executed from the command prompt Window.

Have you ever had a case where an MS SQL database used Windows Authentication, and you were not domain-joined? Start MS SQL Studio from that command prompt; even though it shows your local username, click Log In, and it will use the AD credentials in the background to authenticate! You can even use this to authenticate to web applications that use NTLM Authentication.

## Enumeration through Microsoft Management Console
---
Note: THMJMP1 can be seen as a jump host into this environment, simulating a foothold that you have achieved. Jump hosts are often targeted by the red team since they provide access to a new network segment. A **domain‑joined** machine

Read: [[Microsoft Management Console (MMC)\|Microsoft Management Console (MMC)]].

Enumeration using MMC with the Remote Server Administration Tools (RSAT) AD Snap-ins though it requires installation:

1. Press **Start**
2. Search **"Apps & Features"** and press enter
3. Click **Manage Optional Features**
4. Click **Add a feature**
5. Search for **"RSAT"**
6. Select "**RSAT: Active Directory Domain Services and Lightweight Directory Tools"** and click **Install**

You can start MMC by using the Windows Start button, searching run, and typing in MMC. If you just run MMC normally, it would not work as our computer is not domain-joined, and our local account cannot be used to authenticate to the domain.

![Enumerating Active Directory-8.jpg](/img/user/Enumerating%20Active%20Directory-8.jpg)

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

![Enumerating Active Directory-7.jpg|450](/img/user/Enumerating%20Active%20Directory-7.jpg)

- If machine is domain-joined RSAT should be preinstalled and can run MMC normally no `runas` required from previous task.
- if machine is not domain-joined: Install RSAT, then use `runas /netonly /user:DOMAIN\user mmc.exe` (or start an elevated CMD from your existing `runas` window and launch `mmc`) so MMC’s **network** calls authenticate with the injected AD credentials.
- If your machine is domain‑joined but you logged in with a local account (or a different domain account without rights), `runas /netonly` can still help to force MMC to use other AD creds for network authentication.

### Users and Computers
---
Looking at the Active Directory structure. Expand the snap-in and expand the za domain to see the Initial Organisational Unit (OU) structure:

![Enumerating Active Directory-3.jpg|400](/img/user/Enumerating%20Active%20Directory-3.jpg)

In the **People** directory, you can see that the users are divided according to the department OUs. Clicking each 'directory' shows it's users:

![Enumerating Active Directory-4.jpg|450](/img/user/Enumerating%20Active%20Directory-4.jpg)

Clicking on any of these users will allow us to review all of their properties and attributes. You can also see what groups they are a member of:

![Enumerating Active Directory-5.jpg](/img/user/Enumerating%20Active%20Directory-5.jpg)

You can also use MMC to find hosts in the environment. If you click on either Servers or Workstations, the list of domain-joined machines will be displayed:

![Enumerating Active Directory-6.jpg|450](/img/user/Enumerating%20Active%20Directory-6.jpg)

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
The command prompt the every technical including all types of hacker best friend. CMD has a built-in command that we can use to enumerate information about AD. The `net` command is a handy tool to enumerate information about the local system and AD.
### Users

You can use the `net` command to list all users in the AD domain by using the `user` sub-option:

```C
net user /domain
```

This is useful for determining the size of the domain:

![Enumerating Active Directory-2.jpg|500](/img/user/Enumerating%20Active%20Directory-2.jpg)
[Figure 6.0: Output of `net user /domain` command]:

Enumerate a single user account:

```C
net user zoe.marshall /domain
```

![Enumerating Active Directory-1.jpg|450](/img/user/Enumerating%20Active%20Directory-1.jpg)
[Figure 7.0: Info about specific user]:

**Note:** If the user is only part of a small number of AD groups, this command will be able to show us group memberships. However, usually, after more than ten group memberships, the command will fail to list them all.
### Groups

Enumerating groups in the domain:

```C
net group /domain
```

![Enumerating Active Directory.jpg|500](/img/user/Enumerating%20Active%20Directory.jpg)
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

This will provide us with helpful information such as:

- Length of password history kept. Meaning how many unique passwords must the user provide before they can reuse an old password.
- The lockout threshold for incorrect password attempts and for how long the account will be locked.
- The minimum length of the password.
- The maximum age that passwords are allowed to reach indicating if passwords have to be rotated at a regular interval.

This is great for additional password spraying attacks, guess what single passwords you should use and how many attacks can you run before risking account lockouts. However it's important to note blind password spraying can cause account lock outs since we don't know how many recent failed attempts the target already did.

More info about the net command [here](https://docs.microsoft.com/en-us/troubleshoot/windows-server/networking/net-commands-on-operating-systems)

**Benefits**

- No additional or external tooling is required, and these simple commands are often not monitored for by the Blue team.
- No GUI needed.
- VBScript and other macro languages that are often used for phishing payloads support these commands natively so they can be used to enumerate initial information regarding the AD domain before more specific payloads are crafted.

**Drawbacks**

- The `net` commands must be executed from a domain-joined machine. If the machine is not domain-joined, it will default to the WORKGROUP domain.
- The `net` commands may not show all information. 

### Questions and Problems
---
## Conclusion


