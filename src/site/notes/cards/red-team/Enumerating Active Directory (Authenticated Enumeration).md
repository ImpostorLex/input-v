---
{"dg-publish":true,"permalink":"/cards/red-team/enumerating-active-directory-authenticated-enumeration/","tags":["red-team/ad"]}
---

~ [[cards/active-directory/Active Directory\|Active Directory]] | [[atlas/red-team\|red-team]]
## Summary
---
**What you will find:** Comprehensive overview of Active Directory enumeration techniques after obtaining initial credentials, covering credential injection, built-in Windows tools, PowerShell cmdlets, and automated enumeration frameworks.

**Primary purpose:** Guide post-breach enumeration of AD structure, users, groups, permissions, and attack paths using both native Windows utilities and specialized tools.

**Expect to see:**

- Credential injection techniques without domain-joined hosts
- DNS configuration and SYSVOL verification methods
- Graphical and command-line AD object enumeration
- Password policy discovery for spray attacks
- Automated attack path mapping with BloodHound

## Key Concept - Post-Breach Enumeration Cycle
---
Once you successfully obtain your first set of AD credentials, you can start enumerating various details about the AD setup and structure with authenticated access, even super low-privileged access.

![cards/red-team/images/Enumerating Active Directory.png|500](/img/user/cards/red-team/images/Enumerating%20Active%20Directory.png)

**Enumeration is cyclical:** Once an attack path shown by the enumeration phase has been exploited, enumeration is performed again from this new privileged position to discover further attack paths.

**Progressive privilege escalation:**

1. Initial foothold with low-privileged credentials
2. Enumerate AD structure and relationships
3. Identify attack path to higher privileges
4. Exploit path and gain elevated access
5. Re-enumerate from new position
6. Repeat until objectives achieved

## Knowledge Base
---
### Core Techniques
---

- [[cards/red-team/Credential Injection via Runas\|TECH - T1078.002 - Active Directory Credential Injection via Runas]]
- [[cards/red-team/MMC RSAT Enumeration\|TECH - T1087.002 - Active Directory MMC RSAT Enumeration]]
- [[Net Commands Enumeration\|TECH - T1087.002 - Active Directory Net Commands Enumeration]]
- [[PowerShell AD-RSAT Enumeration\|TECH - T1087.002 - Active Directory PowerShell AD-RSAT Enumeration]]
- [[BloodHound AD Enumeration\|TECH - T1087.002 - Active Directory BloodHound Enumeration]]

### Prerequisites
---

- [[cards/active-directory/Active Directory\|Active Directory]] - Understanding of AD structure and concepts
- [[cards/red-team/Breaching Active Directory\|Breaching Active Directory]] - How to obtain initial AD credentials
- [[cards/active-directory/SYSVOL\|SYSVOL]] - Shared folder containing Group Policy and scripts
- [[Kerberos Authentication\|Kerberos Authentication]] - Understanding ticket-based authentication vs NTLM
- [[Microsoft Management Console (MMC)\|Microsoft Management Console (MMC)]] - Windows administrative tool framework

### Tools Commonly Used
---

- runas.exe - Windows built-in credential injection
- MMC with AD-RSAT Snap-ins - Graphical AD management
- net.exe - Built-in Windows networking commands
- PowerShell AD-RSAT Module - Advanced AD querying cmdlets
- BloodHound + SharpHound - Automated AD relationship mapping and attack path identification

### Detection & Defense References
---

- [[DET - T1078.002 - Credential Injection Detection\|DET - T1078.002 - Credential Injection Detection]]
- [[DET - T1087.002 - AD Enumeration Detection\|DET - T1087.002 - AD Enumeration Detection]]

## Case Notes / Labs
---

- Links to THM, HTB, or internal labs demonstrating these techniques