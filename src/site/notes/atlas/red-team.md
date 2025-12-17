---
{"dg-publish":true,"permalink":"/atlas/red-team/","tags":["map"]}
---

~ [[atlas/map\|map]]

**A map can exist inside a map.**

**Note:** only use tags if the list is overwhelming.

> [!info]- TAGLESS RED TEAM NOTES
>  | Title                                                                                                                          | Last Modified                |
> | ------------------------------------------------------------------------------------------------------------------------------ | ---------------------------- |
> | [[cards/red-team/Persistence through SID History\|TECH - T1134.005 Access Token Manipulation: SID-History Injection]]       | 10:05 AM - December 17, 2025 |
> | [[cards/red-team/Persistence through Tickets\|TECH - T1558.001 Golden Ticket & T1558.002 Silver Ticket]]                    | 10:07 AM - December 17, 2025 |
> | [[cards/red-team/Persistence through Credentials\|TECH - T1003.006 OS Credential Dumping: DCSync]]                          | 8:22 PM - December 16, 2025  |
> | [[cards/red-team/Persistence through Group Membership\|TECH - T1098.001 Account Manipulation: Additional AD Group]]         | 10:13 AM - December 17, 2025 |
> | [[cards/red-team/Exploiting Domain Trusts\|TECH - T1558..001 Exploiting Domain Trusts]]                                     | 8:36 PM - December 13, 2025  |
> | [[cards/red-team/Exploiting Certificates\|TECH - T1649 Steal or Forge Authentication Certificates]]                         | 5:55 PM - December 12, 2025  |
> | [[cards/red-team/images/Exploiting GPOs\|TECH - T1484.001 Exploiting GPOs]]                                                 | 5:03 PM - December 12, 2025  |
> | [[cards/red-team/Exploiting Automated Relays\|TECH - T1557.01  Exploiting Automated Relays]]                                | 8:02 PM - November 28, 2025  |
> | [[cards/red-team/Exploiting Kerberos Delegation\|TECH - T1550.003 Exploiting Kerberos Delegation]]                          | 6:49 PM - November 28, 2025  |
> | [[cards/red-team/Exploiting Permission Delegation\|TECH - T1098 Exploiting Permission Delegation]]                          | 10:50 AM - November 20, 2025 |
> | [[cards/red-team/Port Forwarding\|TECH – T1090 Proxy / Port Forwarding & Pivoting]]                                         | 2:46 PM - November 16, 2025  |
> | [[cards/red-team/Abusing User Behavior\|TECH – T1574.008 Hijack Execution Flow]]                                            | 2:45 PM - November 16, 2025  |
> | [[cards/red-team/Use of Alternate Authentication Material\|TECH – T1550 Use of Alternate Authentication Material]]          | 7:14 PM - November 28, 2025  |
> | [[cards/red-team/Moving Laterally using WMI\|TECH – T1047 Windows Management Instrumentation (WMI-Based Lateral Movement)]] | 3:49 PM - November 16, 2025  |
> | [[cards/red-team/Spawning Process Remotely\|TECH – T1021 Remote Services]]                                                  | 2:32 PM - November 16, 2025  |
> | [[cards/red-team/Tricking VirusTotal or any Scanning Agent\|Tricking VirusTotal or any Scanning Agent]]                     | 6:29 PM - June 06, 2025      |
> 
{ .block-language-dataview}

> [!warning]- HOST EVASION
>  | Title                                                                                                                      | Last Modified                |
> | -------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
> | [[cards/windows/Windows Internals\|Windows Internals for Red Teaming: Processes, Threads, Memory, DLLs]]                | 6:49 PM - August 26, 2025    |
> | [[cards/red-team/Process Hollowing\|Process Hollowing]]                                                                 | 6:46 PM - July 15, 2025      |
> | [[cards/red-team/Abusing Process Components\|Abusing Process Components]]                                               | 6:56 PM - July 12, 2025      |
> | [[cards/red-team/Abusing Windows Internals\|hiding and executing code, evading detection by abusing windows internals]] | 6:56 PM - July 12, 2025      |
> | [[cards/red-team/Evading Antivirus - Shellcode\|Evading Antivirus - Shellcode]]                                         | 6:54 PM - July 12, 2025      |
> | [[cards/red-team/How Antivirus works and gets bypassed\|How Antivirus works and gets bypassed]]                         | 6:54 PM - July 12, 2025      |
> | [[cards/red-team/Staged and Stageless\|staged versus stageless payloads]]                                               | 4:06 PM - August 08, 2025    |
> | [[cards/red-team/Abusing DLLs\|Abusing DLLs]]                                                                           | 6:56 PM - July 12, 2025      |
> | [[cards/red-team/Generating a simple shellcode\|Understanding how to create a basic shellcode]]                         | 6:55 PM - July 12, 2025      |
> | [[cards/red-team/Shellcode Encoding and Encrypting\|Shellcode Encoding and Encrypting]]                                 | 7:50 PM - July 25, 2025      |
> | [[cards/red-team/Packers Against Disk-Based AV Detection\|Packers]]                                                     | 6:55 PM - July 12, 2025      |
> | [[cards/red-team/Binders - malware development\|Binders - malware development]]                                         | 6:55 PM - July 12, 2025      |
> | [[cards/red-team/Obfuscation Principles\|Hide malicious functions and create unique code]]                              | 6:54 PM - July 12, 2025      |
> | [[cards/red-team/Signature Evasion\|Signature Evasion]]                                                                 | 10:39 AM - August 22, 2025   |
> | [[cards/red-team/Bypassing UAC\|Bypassing UAC]]                                                                         | 9:06 PM - August 02, 2025    |
> | [[cards/red-team/Runtime Detection Evasion\|Runtime Detection Evasion]]                                                 | 10:34 AM - August 08, 2025   |
> | [[cards/red-team/Evading Logging and Monitoring\|Evading Logging and Monitoring]]                                       | 10:37 AM - August 22, 2025   |
> | [[cards/red-team/Living off the Land\|Living off the Land]]                                                             | 4:12 PM - August 20, 2025    |
> | [[cards/red-team/Network Security Solutions\|Network Security Evasion]]                                                 | 4:59 PM - September 02, 2025 |
> | [[cards/red-team/Firewall Evasion Techniques\|Firewall Evasion Techniques]]                                             | 6:10 PM - September 11, 2025 |
> | [[cards/red-team/Sandbox Evasion\|Sandbox Evasion]]                                                                     | 10:27 AM - October 03, 2025  |
> 
{ .block-language-dataview}

> [!warning]- THREAT EMULATION
>  | Title                                                    | Last Modified           |
> | -------------------------------------------------------- | ----------------------- |
> | [[cards/red-team/Threat Emulation\|Threat Emulation]] | 7:00 PM - July 12, 2025 |
> 
{ .block-language-dataview}

 > [!warning]- INITIAL ACCESS
>  | Title                                                                      | Last Modified           |
> | -------------------------------------------------------------------------- | ----------------------- |
> | [[cards/red-team/Reconnaissance - Red Team\|Reconnaissance - Red Team]] | 7:02 PM - July 12, 2025 |
> | [[cards/red-team/Phishing\|Phishing]]                                   | 7:03 PM - July 12, 2025 |
> 
{ .block-language-dataview}

 > [!warning]- POST COMPROMISE
>  | Title                                                    | Last Modified           |
> | -------------------------------------------------------- | ----------------------- |
> | [[cards/red-team/Lay off the Land\|Lay off the Land]] | 7:04 PM - July 12, 2025 |
> 
{ .block-language-dataview}

> [!warning]- RED TEAM FUNDAMENTALS
>  | Title                                                        | Last Modified               |
> | ------------------------------------------------------------ | --------------------------- |
> | [[cards/red-team/OPSEC\|Operation Security]]              | 7:05 PM - July 12, 2025     |
> | [[cards/red-team/Input Manipulation\|Input Manipulation]] | 6:59 PM - November 26, 2025 |
> 
{ .block-language-dataview}

 >[!warning]- ACTIVE DIRECTORY
>  | Note                                                                                                                                     |
> | ---------------------------------------------------------------------------------------------------------------------------------------- |
> | [[cards/red-team/Reconnaissance - Red Team\|Reconnaissance - Red Team]]                                                               |
> | [[cards/active-directory/Active Directory Enumeration\|Active Directory Enumeration]]                                                 |
> | [[cards/active-directory/Active Directory Authenticated Enumeration\|Active Directory Authenticated Enumeration]]                     |
> | [[cards/red-team/Breaching Active Directory\|Breaching Active Directory]]                                                             |
> | [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)\|Enumerating Active Directory (Authenticated Enumeration)]] |
> | [[cards/red-team/lateral movement and pivoting\|TA0008 - Lateral Movement and Pivoting]]                                              |
> | [[cards/red-team/Exploiting Active Directory\|Exploiting Active Directory]]                                                           |
> 
{ .block-language-dataview}
>[[cards/active-directory/Group Policy Objects#Red Teaming\|Group Policy Objects#Red Teaming]] has a cool way to attack AD with GPO

#### Others
---

> [!NOTE]+ red teaming related concepts/terminologies
>  | File                                                                                         | is_concept |
> | -------------------------------------------------------------------------------------------- | ---------- |
> | [[cards/red-team/AMSI (Anti-Malware Scan Interface)\|AMSI (Anti-Malware Scan Interface)]] | true       |
> | [[cards/red-team/shell code injection\|shell code injection]]                             | true       |
> 
{ .block-language-dataview}

> [!bug]- red team related resources e.g codeblocks and more mentioned in notes.
>  | Title                                                                                              | Last Modified              |
> | -------------------------------------------------------------------------------------------------- | -------------------------- |
> | [[cards/red-team/Runtime Detection Evasion Code Blocks\|Runtime Detection Evasion Code Blocks]] | 10:29 AM - August 08, 2025 |
> | [[cards/red-team/Signature Evasion Code Blocks\|Signature Evasion Code Blocks]]                 | 6:33 PM - July 24, 2025    |
> | [[cards/red-team/Process Hollowing - Basic Example\|Process Hollowing - Basic Example]]         | 6:50 PM - July 15, 2025    |
> | [[cards/red-team/Obfuscation Principles Object Name Code\|Obfuscation Principles code blocks]]  | 9:39 AM - July 11, 2025    |
> 
{ .block-language-dataview}



See also [[atlas/windows\|windows]] and [[atlas/linux\|linux]]