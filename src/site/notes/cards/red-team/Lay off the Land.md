---
{"dg-publish":true,"permalink":"/cards/red-team/lay-off-the-land/","tags":["red-team/windows"]}
---

~ [[map-of-contents/post-compromise\|post-compromise]]
### Introduction
---
After successfully establishing _initial access_, it's time to learn more about the 'land' -- it's important to familiarize ourselves with the commonly used technologies, and security products.
#### Key Topics
---

- [[#Network Infrastructure|Covers VLANs, subnets, DMZs, and other network segmentation techniques.]]
    
- [[#Enumeration|Basic post-compromise host-level network enumeration using netstat and arp.]]
    
- [[#Active Directory|Verifying domain membership and linking to AD enumeration techniques.]]
    
- [[#Host Security Solutions|Enumerate AV, Defender, firewall, Sysmon, and log settings on the host.]]
    
- [[#Application and Services|Discover installed apps, services, hidden files, and internal network services.]]
## Network Infrastructure
---

- _Network Segmentation (Broad concept)_ is the practice of dividing a large network into smaller, isolated segments commonly referred to as _subnets_ to improve security, performance, and manageability.
	- This include the following: VLANs, Firewalls, Subnets, Access Control Lists (ACLs), and Layer 3 switches or routes.
- _Virtual LAN (Specific Technique)_ a networking method to logically divide a large network into multiple, isolated domains using only one physical switch.

A sample of network segmentation:

![cards/red-team/images/Lay off the Land.png|450](/img/user/cards/red-team/images/Lay%20off%20the%20Land.png)

A **Demilitarized Zone (DMZ)** is a network commonly sits between untrusted network such as the Internet and trusted network such as the internal network.

![cards/red-team/images/Lay off the Land-1.png|450](/img/user/cards/red-team/images/Lay%20off%20the%20Land-1.png)

> [!question]+ DMZ use cases
> One good example of why DMZ is supposed a company offers File Transfer Protocol services, the company setups a DMZ zone using access control list to allow specific public IP address to access the FTP and then as for internal network, the company employee may access the FTP for multiple purposes such as file managing.
### Enumeration
---
The ever useful `netstat` this allows the attacker to view TCP and UDP ports and established connections:

```bash
netstat -na
```

![cards/red-team/images/Lay off the Land-2.png](/img/user/cards/red-team/images/Lay%20off%20the%20Land-2.png)

Next up is, `arp -a` this displays the IP address and the physical address that communicated with (victim) machine within the network, this is useful for moving laterally we don't have to use any network enumeration tool to determine live hosts.

![Lay off the Land-3.png](/img/user/cards/red-team/images/Lay%20off%20the%20Land-3.png)

However, most of the time it will never show all live host as the `arp` table is connection oriented basis.
## Active Directory
---

Read first [[cards/active-directory/Active Directory\|Active Directory]] and then determine if machine is part of a domain:

```C
systeminfo | findstr Domain
```

![Lay off the Land-4.png](/img/user/cards/red-team/images/Lay%20off%20the%20Land-4.png)

Then for enumerating users, groups, services, and more see [[cards/active-directory/Active Directory Enumeration\|Active Directory Enumeration]] and [[cards/active-directory/Active Directory Authenticated Enumeration\|Active Directory Authenticated Enumeration]].
## Host Security Solutions
---
Determining security solutions in place allow us to decrease our chances of detection as all security solutions are not perfect and have known evasion or bypasses techniques.

Display Antivirus product:

```C
wmic /namespace:\\root\securitycenter2 path antivirusproduct
```

Alternative using Powershell:

```C
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct
```

**Note:** that Windows servers may not have `SecurityCenter2` namespace.

**Microsoft Defender**

- **Active mode** - run as primary antivirus software on the machine provides protection and remediation.
- **Passive** - run as secondary antivirus software, usually when a third party antivirus software is installed, it scans files and detect threats but does not provided remediation.
- **Disabled**

Checking service state of Windows Defender using Powershell:

```C
Get-Service WinDefend
```

View the status of Windows Defender:

```C
Get-MpComputerStatus | select RealTimeProtectionEnabled
```

Additionally, we can check for threats that have been detected by Defender using the command below:

```C
Get-MpThreat
```


**Host-based Firewall**
A modern host-based firewall uses multiple levels of analyzing traffic, including packet analysis, while establishing the connection.

Show status of Firewall:

```shell-session
Get-NetFirewallProfile | Format-Table Name, Enabled
```

Output:

```C
Name    Enabled
----    -------
Domain     True // Is enabled when machine is connected to a domain
Private    True // Is enabled on trusted home or private networks
Public     True // Firewall is active on public or untrusted networks.
```

Disabling one or more than one Firewall profile (requires administrator) and viewing it's status:

```C
Set-NetFirewallProfile -Profile Domain, Public, Private -Enabled False
Get-NetFirewallProfile | Format-Table Name, Enabled
```

Testing inbound connection against specific IP address and ports:

```C
Test-NetConnection -ComputerName 127.0.0.1 -Port 80 // Change port and IP address.
```

**Security Event Logging and Monitoring**

List all available logs from the local machine:

```C
Get-EventLog -List
```

It also includes what application and services are installed:

![Lay off the Land2.png](/img/user/cards/red-team/images/Lay%20off%20the%20Land2.png)

**Sysmon**

Look for a process or sevice that has been named "Sysmon"

```C
Get-Process | Where-Object { $_.ProcessName -eq "Sysmon" }
```

Alternatively:

```shell-session
Get-Service | where-object {$_.DisplayName -like "*sysm*"}
```

Additionally, you can use `reg query <sysmon_path>` to check for sysmon and most **importantly**, is to know what they are 'logging' or monitoring for.

Identify configuration file of Sysmon:

```C
findstr /si '<ProcessCreate onmatch="exclude">' C:\tools\*
```

## Application and Services
---
Identify the following:

- Installed applications  
- Services and processes
- Sharing files and printers  
- Internal services: DNS and local web applications

**Installed Applications**

Show application's name and version, useful for finding vulnerable software to exploit:

```C
wmic product get name, version
```

Look for hidden directories, text strings, and backup files:

```C
Get-ChildItem -Hidden -Path C:\Users\kkidd\Desktop\
```

**Services and Processes**

Windows services enable the system administrator to create long-running executable applications in our own Windows sessions. Sometimes Windows services have misconfiguration permissions, which escalates the current user access level of permissions.

**Sharing files and printers**

Printers sometimes have misconfigured access permision.

**Internal services: DNS, local web application, etc**

View interesting running services:

```C
net start
```

Output:

![2Lay off the Land.png](/img/user/cards/red-team/images/2Lay%20off%20the%20Land.png)

In this example, we want to learn more about **THM-Demo** found in the image above:

```C
wmic service where "name like 'THM Demo'" get Name,PathName
```

Alternatively, if it does not work:

```C
wmic service where "name='THM Demo'" get Name,PathName
```

Output:

![Lay off the Land3.png](/img/user/cards/red-team/images/Lay%20off%20the%20Land3.png)

Now, we know the filename and it's path, determine it's id using:

```C
Get-Process -Name thm-demo
```

Then once we have it's Process ID, let see if it's providing a network service by listing the listening ports within the system:

```C
netstat -noa | findstr "LISTENING" | findstr "3212"
```

**Enumerating DNS**

**List all DNS records** by attempting a **zone transfer**


```C
nslookup.exe
```

We should get a prompt after that let's provide the DNS server a target machine:

```C
server 10.10.10.253
```

Output:

![Lay off the Land5.png](/img/user/cards/red-team/images/Lay%20off%20the%20Land5.png)

Here is what to look for:

![Lay off the Land5-1.png](/img/user/cards/red-team/images/Lay%20off%20the%20Land5-1.png)


