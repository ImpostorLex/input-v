---
{"dg-publish":true,"permalink":"/cards/red-team/lay-off-the-land/","tags":["windows/red-team"]}
---

~ [[map-of-contents/post-compromise\|post-compromise]]
### Introduction
---
After successfully establishing _initial access_, it's time to learn more about the 'land' -- it's important to familiarize ourselves with the commonly used technologies, and security products.
#### Quick Summary
---

- Discover open and established ports and which devices in the network communicated with the (victim) machine.
- Identify if (victim) machine is part of  Active Directory domain.
- Enumerate Active Directory domain both unauthenticated and authenticated.
- Determining host security solutions

## Network Infrastructure
---

- _Network Segmentation (Broad concept)_ is the practice of dividing a large network into smaller, isolated segments commonly referred to as _subnets_ to improve security, performance, and manageability.
	- This include the following: VLANs, Firewalls, Subnets, Access Control Lists (ACLs), and Layer 3 switches or routes.
- _Virtual LAN (Specific Technique)_ a networking method to logically divide a large network into multiple, isolated domains using only one physical switch.

A sample of network segmentation:

![Lay off the Land.png|450](/img/user/cards/red-team/images/Lay%20off%20the%20Land.png)

A **Demilitarized Zone (DMZ)** is a network commonly sits between untrusted network such as the Internet and trusted network such as the internal network.

![Lay off the Land-1.png|450](/img/user/cards/red-team/images/Lay%20off%20the%20Land-1.png)

> [!question]+ DMZ use cases
> One good example of why DMZ is supposed a company offers File Transfer Protocol services, the company setups a DMZ zone using access control list to allow specific public IP address to access the FTP and then as for internal network, the company employee may access the FTP for multiple purposes such as file managing.
### Enumeration
---
The ever useful `netstat` this allows the attacker to view TCP and UDP ports and established connections:

```bash
netstat -na
```

![Lay off the Land-2.png](/img/user/cards/red-team/images/Lay%20off%20the%20Land-2.png)

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

```shell-session
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct
```

