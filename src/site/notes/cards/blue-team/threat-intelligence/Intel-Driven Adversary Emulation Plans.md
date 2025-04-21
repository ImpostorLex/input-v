---
{"dg-publish":true,"permalink":"/cards/blue-team/threat-intelligence/intel-driven-adversary-emulation-plans/"}
---

[[cards/red-team/MITRE ATT&CK\|MITRE ATT&CK]]
### Introduction
---
Showcase how to use Adversary Emulation Plans from MITRE to validate your defenses and run evaluations to see how your defense tools stack up against today's threats.

- _adversary emulation_ is a kind of red team engagement that takes known threats to an organization and reproduces them by blending in threat intelligence to define what actions and behaviors the red team uses. 
	- The goal is to test their network and defenses by modelling their behavior to adversary.
	- Different from pentesting - the goal is to emulate the adversary TTPs.

https://medium.com/mitre-engenuity/introducing-the-all-new-adversary-emulation-plan-library-234b1d543f6b
## Creating Adversary Emulation Plans
---
1. **Gather Threat Intelligence --** Select an adversary based on the threats to your organization, and analyze intelligence about what the adversary has done. Document their behavior: what they go after, and whether they do _smash and grab or low and slow._
2. **Extract Techniques --** In the same way you mapped your red team operations to ATT&CK techniques, map the intel you have to specific techniques/
3. **Analyze and organize --** Now you have a bunch of intel about the adversary and how they operate, diagram that information into their operational flow in a way that’s easy to create specific plans from.
4. **Develop tools and procedures --** Figure out how to implement the behavior including:
	- How did the threat group use this technique?
	- Did the group vary which technique used based on the environment context?
	- What tools can we use to replicate these TTPs?
5. **Emulate the adversary --** The red team should closely work with the blue team to gain a deep understanding of where gaps are in the blue team’s visibility and why they exist
### Tools to emulate
---
- Atomic Red Team
- Caldera
- Endgame RTA
- Uber Metta


