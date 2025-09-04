---
{"dg-publish":true,"permalink":"/cards/blue-team/digital-forensics/digital-forensics/"}
---

[[atlas/blue-team\|blue-team]]
### Introduction
---
It is the process of identifying, and collecting digital evidence to support our claims.

- **transfer of material** between two objects colliding, such as fingerprint on the door knob.

## Investigation Process
---
The investigation process depends on the situation handed, different situation may require different process as well as consideration of legal standards.
#### Identification
---
- Confirm an incident or crime has taken place.
- Sources of identification
	- Victim Reports.
	- Detection and controls.
- Preparing to respond and collect evidence.
	- Locating and Preserving.
- Define scope and focus efforts accordingly.
#### Preservation
---
**note:** goal is to preserve integrity. [[cards/blue-team/digital-forensics/Order of Volatility\|Order of Volatility]]

- Ensuring evidence remains unaltered.
- Capture a snapshot of the device's current state.
- Identify relevant data to the case.
	- Prioritize volatile data such as the RAM.
- [[cards/blue-team/digital-forensics/Chain of Custody\|Chain of Custody]]
	- Document the handling of evidence.
	- Tracking everyone who's had possession of evidence.
- Secure the evidence.
	- Hash
#### Collection
---
**Note:** in collection it is important that we put the captured image onto our external usb or drive and never install forensic tools on the questioned device.

- Gather all of the identified evidence.
- Ensure legal authority (or it would be stealing.)
- [[cards/blue-team/digital-forensics/Order of Volatility\|Order of Volatility]].
- Filter out irrelevant information.
- Document everything.
#### Examination
---
- Analyze and interpret evidence to draw a conclusion.
- Maintain evidence integrity.
- Data discovery
	- Restore deleted or hidden data.
#### Analysis
---
- How does the evidence supports or refutes the hypothesis?
- Analyze, ask new questions, re-analyze.
- Link evidence to people, places, and things.
#### Presentation
---
- Communicate the results, ensure understood by all technical backgrounds.
- Executive Summary.
- Comprehensive Documentation
	- any analysis can follow along or easy to follow.
#### Decision
---
- Evaluating the result to determine if:
	- More investigation is required or not.