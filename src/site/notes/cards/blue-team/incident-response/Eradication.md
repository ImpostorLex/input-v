---
{"dg-publish":true,"permalink":"/cards/blue-team/incident-response/eradication/"}
---

[[atlas/blue-team\|blue-team]]
### Introduction
---
Once the threat is [[cards/blue-team/incident-response/Containment\|contained]], we then now move on to fixing the root cause, such as determining it's behavior, analyzing IOCs, and more.

- [SANS Incident Response - Eradication form]([https://www.sans.org/media/score/incident-forms/IH-Eradication.pdf](https://www.sans.org/media/score/incident-forms/IH-Eradication.pdf))

- **Removing the attacker's system and network artifacts**.
- **Restoring Systems** 
	- **How far back into backups**? we should know this in the [[cards/blue-team/incident-response/Containment\|containment]] the stage.
	- **Recovery Time Object (RTO)** - the maximum acceptable time a system is down.
	- **Recovery Point Objective (RPO)**  - how much data we can afford to lose for the sake of business continuity.
- **Manually Removing Artifacts**
	- Malware
		- Disk
		- Memory
	- Backdoors
	- Network connections
	- Services
- **Improving Defenses**
	- Applying patches.
	- System [hardening (CIS Benchmarks).]([https://www.cisecurity.org/cis-benchmarks](https://www.cisecurity.org/cis-benchmarks))
- **Vulnerability Management**
	- Scanning the system or network for any related vulnerabilities.
	- [[cards/blue-team/threat-intelligence/Threat Intelligence\|Threat Intelligence]]


