---
{"dg-publish":true,"permalink":"/cards/blue-team/network-security/intrusion-detection-and-prevention-system/"}
---

[[map-of-contents/blue-team\|blue-team]]

- **Intrusion Detection System** are software or programs meant to alert based on signature, policy violation, rules, or behavioral monitoring and analysis (**REACTIVE**).
	- Out-of-band traffic does not need to sit directly in traffic flow, it only requires a copy.
- **Intrusion Prevention System** are programs meant to take action once a condition is satisfied such as blocking and dropping traffic. (**PROACTIVE)**
	- Sits between traffic as the goal is to prevent malicious traffic by inspecting the packet's payload, may affect network performance

- It has two versions one is at a network level and one is at a host level.
- **Detection Methods**
	- **Signature Based** compares traffic against known patterns
	- **Behavior Based** identify anomalies that is based on pre-established baseline.
		- It has better chance of detecting unknown threats however prone to false positives must have a great baseline.
	- **Rule-based detection** highly customizable as it depends on organization policies - [[cards/blue-team/network-security/Snort\|Snort]]

