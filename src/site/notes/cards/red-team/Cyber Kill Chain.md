---
{"dg-publish":true,"permalink":"/cards/red-team/cyber-kill-chain/","tags":["sunday","template"]}
---

[[map-of-contents/blue-team\|blue-team]]
### Introduction 
---
A model that shows the whole cyber attack process opposed to [[cards/red-team/MITRE ATT&CK\|MITRE ATT&CK]] where it models specific behaviors of certain threat groups.
## Reconnaissance

- Obtaining information about the target, the more information the attacker obtain the more _attack surfaces_(All possible threat vectors) opens up.
	- **Passive** refers to the collection of information without interacting with the target such as web archives.
	- **Active** is the opposite, it interacts with the target to gain information such as analyzing web request.

**Defenders should ideally**
- Detect information disclosure with external pentesting.
- Remove sensitive information available to the Internet.
### Weaponization
---
- threat actors crafts or uses a specific tool that is appropriate based on the gathered information about the target.

**Defenders should ideally**
- Regular identification for security vulnerabilities.
#### Delivery
---
- threat actor executes the cyber attack he/she prepared.

**Defenders should ideally**
- Always be skeptical and validate URL, programs to be downloaded not only for defenders.
- User training on information security.
- Monitors logs from SIEM.

#### Exploitation
---
- The stage where the threat actor ensures the delivered malicious content gets executed

**Defenders should ideally**
- Training employees.
- Regular automated vulnerability scanning and monitoring reports
- pentest on assets.
##### Installation
---
- Attempts to maintain persistence on target system because the vulnerability may be patched or inaccessible due to detection.

**Defenders should ideally**
- Perform network security monitoring.
- Restrict access to critical files.
- Monitor admin activities.
- Malicious process.

##### Command and Control (c2)
---
- Threat actor managed to take control of a machine and in this step the threat actor sends command to the victim machine to carry out malicious activities.

**Defenders should ideally**
- block C2 IP address or if possible domain name.
- C2 makes a lot of noise due to c2 sends command and next is  victim sends output of the command back to c2.
##### Action on Objectives
---
- threat actor carries out any desired operations on the system, this could be ransomware, attack other organization using the compromised machine, sell information, and more.

**Defenders should ideally**
- monitor for suspicious traffic
- at this stage restrict network access to outside
- restrict access to sensitive information.
- use data loss protection products.
## Conclusion




