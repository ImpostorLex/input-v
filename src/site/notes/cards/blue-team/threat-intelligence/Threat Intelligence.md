---
{"dg-publish":true,"permalink":"/cards/blue-team/threat-intelligence/threat-intelligence/"}
---

~ [[atlas/blue-team\|blue-team]]
## Blue-Team Context
---
- Threat Intelligence is the process of analyzing and applying information about potential threats to an organization.  
- The Blue Team’s role: understand actor behavior, anticipate adversary moves, and defend accordingly.  
- **Attribution:** the faster we identify the threat actor, the faster we understand their motivation and behavior.  

## Key Areas of Focus
---
### Adversary and Actor Profiling
---

- Identify threat actor: who, where, why — motivations, capabilities, infrastructure. 
- **Reference:** [[cards/blue-team/threat-intelligence/Diamond Model of Intrusion Analysis\|Diamond Model of Intrusion Analysis]]  
- Use profiling to anticipate TTPs and prepare defenses.  

### Methods, Techniques, and Indicators
---

- Understand adversary tactics, techniques, and procedures (TTPs).  
- Identify and monitor Indicators of Compromise (IOCs): domains, IPs, hashes, and artifacts.  
- References: [[cards/blue-team/threat-intelligence/Cyber Kill Chain\|Cyber Kill Chain]], [[cards/blue-team/threat-intelligence/Pyramid of Pain\|Pyramid of Pain]]  
- Apply these models to prioritize intelligence and detection efforts.  

### Threat Intelligence Sharing and Collaboration
---

- Use sharing ecosystems such as [[cards/blue-team/threat-intelligence/Malware Information Sharing Platform\|Malware Information Sharing Platform]] (MISP).  
- Identify **Producers** (collect, analyze, share intel) and **Consumers** (use intel in defenses).  
- Reference: [[cards/blue-team/threat-intelligence/Intel-Driven Adversary Emulation Plans\|Intel-Driven Adversary Emulation Plans]]  
- Transform collected intelligence into adversary emulation and red-teaming exercises.  

### Operationalization in SOC and Defense
---

- **Apply threat intelligence to day-to-day SOC workflows.**  
- Feed IOCs and behavioral indicators into SIEM and XDR tools.  
- References: [[cards/blue-team/threat-intelligence/Threat Intelligence for SOC\|Threat Intelligence for SOC]], [[cards/blue-team/threat-intelligence/IP and Domain Threat Intel\|Guide for enriching IP and Domain Threat Intelligence]]  
- Use collected data for:  
	  - Intelligence-driven prevention (block malicious IPs/domains).  
	  - Intelligence-driven detection (detect residual or recurring indicators).  
	  - Intelligence-driven triage and enrichment (see [[cards/blue-team/threat-intelligence/IP and Domain Threat Intel\|IP and Domain Threat Intel]] for full process).  

### Feedback and Continuous Improvement
---

- Conduct incident reviews: did the intelligence predict, detect, or miss the threat?  
- Update models, sharing protocols, and detection logic.  
- Maintain a continuous loop between collection, analysis, action, and refinement.  

## Reference Notes
---
- [[cards/blue-team/threat-intelligence/Diamond Model of Intrusion Analysis\|Diamond Model of Intrusion Analysis]]  
- [[cards/blue-team/threat-intelligence/Malware Information Sharing Platform\|Malware Information Sharing Platform]]  
- [[cards/blue-team/threat-intelligence/Intel-Driven Adversary Emulation Plans\|Intel-Driven Adversary Emulation Plans]]  
- [[cards/blue-team/threat-intelligence/Threat Intelligence for SOC\|Threat Intelligence for SOC]]  
- [[cards/blue-team/threat-intelligence/Cyber Kill Chain\|Cyber Kill Chain]]  
- [[cards/blue-team/threat-intelligence/Pyramid of Pain\|Pyramid of Pain]]  
- [[cards/blue-team/threat-intelligence/IP and Domain Threat Intel\|IP and Domain Threat Intel]]  & [[cards/blue-team/threat-intelligence/Ref IP and Domain Threat\|a shorter version for reference]]

## Quick Usage Guide
---
- When **planning** your intelligence effort, start with [[cards/blue-team/threat-intelligence/Diamond Model of Intrusion Analysis\|Diamond Model of Intrusion Analysis]].  
- When **implementing** or **defending**, go to [[cards/blue-team/threat-intelligence/Threat Intelligence for SOC\|Threat Intelligence for SOC]].  
- For **intel sharing**, use [[cards/blue-team/threat-intelligence/Malware Information Sharing Platform\|Malware Information Sharing Platform]].  
- To **improve and iterate**, review [[cards/blue-team/threat-intelligence/Intel-Driven Adversary Emulation Plans\|Intel-Driven Adversary Emulation Plans]].  





