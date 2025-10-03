---
{"dg-publish":true,"permalink":"/cards/red-team/sandbox-evasion/","tags":["red-team/host-evasion"]}
---

~ [[atlas/red-team\|red-team]]
### Introduction 
---
This is part of the concept **Defense in Depth** wherein one layer of protection fails another layer stands up. Sanbox is a process of analysing a file via static and dynamic analysis to determine whether the file is malicious or not.

#### Key Topics
---

## Prerequisites

- [[atlas/malware\|malware]]
- [[cards/windows/Windows Internals\|Windows Internals]]
- [[cards/red-team/Abusing Windows Internals\|Abusing Windows Internals]]

## Sleeping through Sandboxes
---
Sandboxes are often limited to a time constraint to prevent the overallocation of resources, which may increase the Sandboxes queue drastically. For example: if we know the sandbox will only run for _five minutes_, we can implement a sleep timer that sleeps for five minutes before executing our malicious payload.

Another method is to do complex, compute-heavy math though it may take more or less time to do based on the system's hardware, for example, calculating the **Fibonacci sequence** up to a given number.

It is recommended to develop your own sleep function as there are multiple sandboxes that counters or alters built-in sleep functions:

- [https://evasions.checkpoint.com/src/Evasions/techniques/timing.html](https://evasions.checkpoint.com/src/Evasions/techniques/timing.html)  
- [https://www.joesecurity.org/blog/660946897093663167](https://www.joesecurity.org/blog/660946897093663167)

## Checking System Information
---
As mentioned before, sandboxes often run on limited resource and this is a great way to evade sandboxes, one of the popular sandbox out there is _any.run_, it only allocates 1 CPU core and 4GB of RAM per virtual machine.

Most workstations in a network typically have 2-8 CPU cores, 8-32GB of RAM, and 256GB-1TB+ of drive space. This is dependent on the organization but generally you can expect more than 2 CPU cores per system and more than 4GB of RAM.

Things to query or filter for, **again** context is the key:

- Storage Medium Serial Number
- PC Hostname
- BIOS/UEFI Version/Serial Number
- Windows Product Key/OS Version
- Network Adapter Information
- Virtualization Checks
- Current Signed in User
- and much more!

For example, the sandbox hostname does not align with the organization endpoint naming convention, or this machine is using Windows 10 wherein all company devices at Windows 10.
## Querying Network Information
---
Many malware families try to determine **what environment they’re running in** before doing anything noisy. One of the easiest signals is: **is the machine joined to an Active Directory (AD) domain?**

- If it's not domain joined > exit quietly.
- If it's domain joined > run the malicious payload.

There are many objects that you can query:

- **Computers**
- **User accounts**
- **Last User Login(s)** human activity evidence
- **Groups**
- **Domain Admins**
- **Enterprise Admins**
- **Domain Controllers** are there DCs reachable on the network?
- **Service Accounts** are their service accounts and scheduled jobs? this is common in an enterprise infrastructure.
- **DNS Servers**

However these are common network things that analyst look for, alternatively these are low-risk check you can run locally:

- Inspect environment variables (Windows CMD):
    - `set` → look for `LOGONSERVER`, `USERDOMAIN`, etc.
        
    - `echo %LOGONSERVER%`
        
- Basic identity info:
    - `whoami` (shows domain\username when joined)
        
    - `systeminfo` (on Windows, often includes `Domain: <domain>` line)
        
- Network neighbors you can see without sweeping the network:
    - Check DNS suffix / DNS server configured on the machine (shows corporate DNS in many environments)


### Questions and Problems
---
## Conclusion


