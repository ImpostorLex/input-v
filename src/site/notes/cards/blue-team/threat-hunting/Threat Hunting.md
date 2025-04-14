---
{"dg-publish":true,"permalink":"/cards/blue-team/threat-hunting/threat-hunting/"}
---

[[Threat Hunting Map\|Threat Hunting Map]]
### Introduction 
---
It is actively looking for signs of malicious activity guided by [[cards/blue-team/threat-intelligence/Threat Intelligence\|Threat Intelligence]], it is different from Incident Response (IR), It usually starts with a alert or notification then it is triaged (assess, prioritize, and addressing), analyzed, and build enough evidence to consider it as an indicent that needs to be responded.

Incident Response = Reactive
Threat Hunting = Proactive

![Threat Hunting.png|450](/img/user/cards/blue-team/threat-hunting/images/Threat%20Hunting.png)

**Figure 1** shows the synergry of both Incident Response and Threat Hunting when it comes to strengthening security posture.
## Mindset

The process should start with this question and followed by:

1. What do we want to hunt for?

	- Identify what is relevant to our company such as what threat group or APT attacks company similar to ours?

2. We want to arm ourselves with critical information that will let us know more about the threat(s) that we may be dealing with.

	- Source from threat intelligence providers such as [[MISP\|MISP]] or paid that can tailor intelligence unique for your environments.
	
3. Use **Attack Signatures** and **Indicators of Compromise (IOCs)** to define identifiers for target and compare against historical data or from reports.

4. **Patterns of Activity:** after identifying relevant threat actors, it's time to characterise their behaviour through patterns of activity that they most likely going to make.

5. **Undetectable to Detectable** once we successfully profiled our threats, it's time to translate them into **detection mechanism** since we don't want to hunt for the same threats.
### Setting What We Want To Hunt with ATT&CK
---
The [ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/) for threat hunting, the end result:

![Threat Hunting-1.png](/img/user/cards/blue-team/threat-hunting/images/Threat%20Hunting-1.png)

Let's assume we want to hunt for ransomware, we got WannaCry, Stuxnet, and Conficker as our target:

![Threat Hunting-2.png](/img/user/cards/blue-team/threat-hunting/images/Threat%20Hunting-2.png)

1. Create New Layer
2. Choose Enterprise

![Threat Hunting-3.png](/img/user/cards/blue-team/threat-hunting/images/Threat%20Hunting-3.png)

Click the magnifying glass to open the side panel and search for WannaCry, after clicking the 'Select All' - to better visualize under "technique controls" select background colour and set **arbitrary score** of 1 for this layer, the control for the score is beside the background color:

![Threat Hunting-4.png](/img/user/cards/blue-team/threat-hunting/images/Threat%20Hunting-4.png)

Do the same for other malwares by performing the same way we did for WannaCry = 1, Stuxnet = 2, Conficker = 4:
### Visualizing them all together
---
![Threat Hunting-5.png](/img/user/cards/blue-team/threat-hunting/images/Threat%20Hunting-5.png)

Select "**Create Layer from other layers**" and under domain choose the latest version, and then put **"a+b+c"** on the **"score expression"** and skip other options and we should end up with this:

![Threat Hunting-6.png](/img/user/cards/blue-team/threat-hunting/images/Threat%20Hunting-6.png)

A deep **red colour** means that the type of attack is common to all of them, while the other lighter colours mean it’s either one or any combination of two threat actors.

By hovering to each attack we can see score, and based on the score, this mean that Stuxnet and Conficker uses this technique:

![Threat Hunting-7.png](/img/user/cards/blue-team/threat-hunting/images/Threat%20Hunting-7.png)

### When do we stop hunting?
---
In Capture The Flag, we know that if we prepare enough there is a flag waiting for us but in Threat hunting if we do everything right, it is still possible to not find anything at all.

#### Conclusion
---
It's the proactive way of finding threats in our organization leveraging resources such as MITRE ATT&CK framework for mapping out attacker's tactics, techniques, and procedures (TTPs) rather than waiting for an alert or incident to take action which is done in **incident response**, however even done rigth there is chance we may not find anything at all. (which is good in a sense.)