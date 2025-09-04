---
{"dg-publish":true,"permalink":"/cards/blue-team/threat-intelligence/diamond-model-of-intrusion-analysis/"}
---

[[atlas/blue-team\|blue-team]]

The goal is to determine the adversary using the other 'points' in the model:

![Diamond Model of Intrusion Analysis-1.png|450](/img/user/cards/blue-team/threat-intelligence/images/Diamond%20Model%20of%20Intrusion%20Analysis-1.png)
- **Capability** 
	- How good their malware is?, did they develop it on their own?, do they have their own zero day?
- **Infrastructure** any technical element used to support and manage the intrusion.
	- IP, email addresses, domain names, c2 and more.
	- **A mean to maintain or deliver their capability**.
- **Victim**
	- **Victim Personae** specific organization or sectors, non-key and key individuals.
	- **Victim Assets** actual hardware, software, endpoints, and more.
## Stuxnet
---

The adversary here is unknown but by looking at the other 'points':

![Diamond Model of Intrusion Analysis-2.png|400](/img/user/cards/blue-team/threat-intelligence/images/Diamond%20Model%20of%20Intrusion%20Analysis-2.png)
- Who have the capability to do the following such as finding 4 zero days?
- How did the infected usb drive got delivered to air-gapped industrial control system and tight security?
- Who would target Iran's nuclear program?

- **Answers**
	- Industrial control systems are expensive, the malware should evade detection, their must be a lot of security researchers involve finding 4 Zero days exploit, so the actor must be **rich** and a lot of **manpower**.
	- The likeliness of the USB drive getting picked on the ground and being inserted to the system is zero. the **threat actor must have delivered him/herself** bypassing security checks, in this the threat actor must have a lot of experience in infiltrating enemy bases. 
	- Why Iran? and why their nuclear program? simple threat to national safety.
	- Conclusion? the USA .