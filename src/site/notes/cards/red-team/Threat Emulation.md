---
{"dg-publish":true,"permalink":"/cards/red-team/threat-emulation/","tags":["red-team/threat-emulation"]}
---

~ [[map-of-contents/red-team\|red-team]]
### Introduction 
---
It is a act of intentionally performing real world attacks to better improve the organization security posture and response capabilities.

- **Threat simulation** have predefined and automated attack patterns - think it like this: goat simulator then the simulator should simulate goat behaviors while emulation simulates what the attacker expects: **UNKNOWN.**
## Prerequisites

- MITRE ATT&CK 
## Emulation Methodologies

- The ATT&CK navigator allows user to know what specific techniques each threat group uses.
- Atomic Red Team is a library of emulations to test security defences.
### Emulation Process
---
Assume the following: VASEPY Corp is a multi-billion dollar U.S. retail establishment.

1. **Define Emulation Objectives** - it should be clearly defined, specific, and measurable. 
	- In case of our company it should be protection against credit card fruad and ransomeware attacks.
2. **Research Adversary TTPs** the goal is to accurately model the target adversary and identify behaviours that can be tested based on the set environment.
	 1. **Information Gathering** Avoid instances of non-concerning threat groups such as APT41 that deals with cyber espionage. **Start with internal sources**, such as system administrators, network owners, and the cyber defence team and see if they seen about threats.
	 2. **Selecting the Adversary**:
		 -  Relevance: It should be selected to the engagement objectives and company's goal.
		 - Available CTI: It should have enough trustworthy resources around the TTPs performed by the threat group.
		 - TTP Complexity: Complex TTP may require some time - identify if existing tools can handle the emulation or it may require custom.
		 - Available Resources.
	3. **Selecting the Emulated TTPs** tools such as the ATT&CK navigator is useful here then back to CTI to learn more about the TTP and actual execution.
	4. **Construct TTP Outline** how the emulation will be executed, stating the scope and rules of engagement, sources, and how TTPs will be implemented during the exercise.

![Threat Emulation.png](/img/user/cards/red-team/images/Threat%20Emulation.png)
#### 3. Planning the Threat Emulation Engagement
---
Since threat emulation involves conducting real world attacks, issues may occur such as disclosure of private data, data loss, and unplanned system downtime.
##### 3.1 Threat Emulation Plans
---
A collection of resources used to organize and step by step execution of instructions for every TTPs.

The plan should consist of:

- **Engagement Objectives** discussed in 1.
- **Scope** departments, users, and devices which emulation activities are permitted.
- **Schedule** when the activities will occur and when is the due date.
- **Rules of Engagement** it is a list of specific activities in pentest/redteam/threat emulation project so attacker and client are on the same page.
- **Permission to Execute** explicit written consent for the emulation activities.
- **Communication plan** how information is sent and received from both parties and what communication channel?
#### 4. Conducting the Emulation
---
This is where we actually carry out the TTPs identified in the research phase and issues should be addressed immediately.
##### 4.1 Planning the Deployment
---
The use of ATT&CK to understand the TTPs and use the Navigator to map them out combined with the CTI resources, in our case for initial access how FIN7 used Windows document lures to execute their campaign.
##### 4.2 Implementation of TTP and 4.3 Detections & Mitigations
---
Deplyoment of actual TPPs, in our case the initial access payload for FIN7 is created and delivered through spear-phishing email **and** the defence team must find ways to detect and mitigate against emulated TTPs.

#### 5. Observe Results and 6. Document & Report Findings
---
The observing team (usually the blue team) must identify artefacts that came from the emulation activity **and** once the results have been obtained, the team must document and report the finding - the document should cover the exercise procedure, what was executed, the impact faced and recommendations. 
 

## Conclusion


