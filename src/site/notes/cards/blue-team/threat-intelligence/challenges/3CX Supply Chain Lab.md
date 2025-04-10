---
{"dg-publish":true,"permalink":"/cards/blue-team/threat-intelligence/challenges/3-cx-supply-chain-lab/"}
---

[[cards/blue-team/threat-intelligence/Threat Intelligence\|Threat Intelligence]]

> Understanding the scope of the attack and identifying which versions exhibit malicious behavior is crucial for making informed decisions if these compromised versions are present in the organization. How many versions of 3CX **running on Windows** have been flagged as malware?

At anyrun.io
![3CX Supply Chain Lab.png](/img/user/cards/blue-team/threat-intelligence/images/3CX%20Supply%20Chain%20Lab.png)

> Determining the age of the malware can help assess the extent of the compromise and track the evolution of malware families and variants. What's the UTC creation time of the `.msi` malware?

![3CX Supply Chain Lab-1.png](/img/user/cards/blue-team/threat-intelligence/images/3CX%20Supply%20Chain%20Lab-1.png)

> Executable files (`.exe`) are frequently used as primary or secondary malware payloads, while dynamic link libraries (`.dll`) often load malicious code or enhance malware functionality. Analyzing files deposited by the Microsoft Software Installer (`.msi`) is crucial for identifying malicious files and investigating their full potential. Which malicious DLLs were dropped by the `.msi` file?


Info found here from report [here](https://www.zscaler.com/blogs/security-research/3cx-supply-chain-attack-campaign)

![3CX Supply Chain Lab-2.png](/img/user/cards/blue-team/threat-intelligence/images/3CX%20Supply%20Chain%20Lab-2.png)
Search term for google "malware_name report"

> Recognizing the persistence techniques used in this incident is essential for current mitigation strategies and future defense improvements. What is the MITRE sub-technique ID employed by the `.msi` files to load the malicious DLL?

Behaviors of VirusTotal:

![3CX Supply Chain Lab-3.png](/img/user/cards/blue-team/threat-intelligence/images/3CX%20Supply%20Chain%20Lab-3.png)
> Recognizing the malware type (`threat category`) is essential to your investigation, as it can offer valuable insight into the possible malicious actions you'll be examining. What is the threat category of the two malicious DLLs?


Hash of the .dll then VirusTotal

![3CX Supply Chain Lab-4.png](/img/user/cards/blue-team/threat-intelligence/images/3CX%20Supply%20Chain%20Lab-4.png)

> As a threat intelligence analyst conducting dynamic analysis, it's vital to understand how malware can evade detection in virtualized environments or analysis systems. This knowledge will help you effectively mitigate or address these evasive tactics. What is the MITRE ID for the virtualization/sandbox evasion techniques used by the two malicious DLLs?

Found on VirusTotal

> When conducting malware analysis and reverse engineering, understanding anti-analysis techniques is vital to avoid wasting time. Which hypervisor is targeted by the anti-analysis techniques in the `ffmpeg.dll` file?

VirusTotal's behavior's capabilities category:

![3CX Supply Chain Lab-5.png](/img/user/cards/blue-team/threat-intelligence/images/3CX%20Supply%20Chain%20Lab-5.png)

40 mins

