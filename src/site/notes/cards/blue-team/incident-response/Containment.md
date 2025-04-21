---
{"dg-publish":true,"permalink":"/cards/blue-team/incident-response/containment/"}
---

[[map-of-contents/blue-team\|blue-team]]
### Introduction
---
The stage where we identified a legitimate security incident, the goal now is to limit the potential impact or damage.

- [SANS - Forms and Questions to answer during containment](https://www.sans.org/media/score/incident-forms/IH-Containment.pdf)

- **Short-term Containment** temporary stop.
	- Isolating through network either physically or through controls such as using EDR.
		- Move the infected machine into a VLAN or a honey net to simulate a fake network.
	- Network filters and rules through Firewall and IPS/IDS.
	- Killing processes.
	- However, **the above should be a calculative risk,** as soon the attacker knows it, it might come back with a much more complex technique or think about the forensic side if we shutdown the device we might lost all volatile data..
- **Business Impact** 
	- The goal is to minimize business disruption as much as possible. 
	- Data owners and stakeholders have the best understanding on what is critical so it is important to work with them.
- **Evidence Collection / System Backup**
	- Creating forensic images including volatile memory.
	- Documentation: Chain of Custody, Analysis.
- **Long-term Containment** after succesfully stopping the bleeding.
	- Ensuring the attackers can't do it again.
	- Returning the environment into a stable condition.
	- Removing the attacker's access.
		- Deleting user accounts.
		- Resetting credentials.
		- Disable remote access, and removing backdoors.
	- **More monitoring and detection** ensure that the attackers won't resurface or there is no more signs of life.
	- **Patching and hardening** not only the compromised machine but to other as well that possibly have the same configurations.

```C
taskkill /pid <pid>
```

