---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/email-analysis/"}
---

[[cards/blue-team/phishing/Phishing\|Phishing]]
### Methodolgy
---
- **Initial Triage**
	- **Quickly assess and prioritize**: taking notes of the obvious.
	- Potential phishing against the CEO versus a low-level employee so which first? the CEO of course.
- [[cards/blue-team/phishing/Headers to look out for\|Header and Sender Examination]] 
	- Analyzing metadata such as: IP address, [[cards/blue-team/phishing/Email#Introduction\|MTAs]], and more.
	- Goal is to identify true origin and check authenticty.
- [[cards/blue-team/phishing/Content Examination\|Content Examination]]
	- Social Engineering flags
	- Double check `content-transfer-encoding` header
- [[cards/blue-team/phishing/URL Analysis\|Web and URL Examination]]
	- VirusTotal, CiscoTalos
	- urlscan.io, hybridanalysis
	- urlhaus for malware analysis link
- [[cards/blue-team/phishing/Attachment Examination\|Attachment Examination]]
	- hybridanalysis, VirusTotal's hash
	- **Keep in mind the things you upload - check company policy first.**
- **Context Examination**
	- Identify patterns: recent or current incidents.
	- Is this a recurring sender email address?
- **Defense Measures**
	- [[cards/blue-team/phishing/Reactive Phishing Defense\|Reactive Phishing Defense]] and [[cards/blue-team/phishing/Proactive Phishing Defense\|Proactive Phishing Defense]] action.
	- Communicate with users and stakeholders.
- **Documentation and Reporting**
	- All of the steps taken literally.
	- [[Challenges TimeKeeping#Phishing\|Challenges]]

## Questions
---
> What about Message-ID that make uses of proton server to send but has a custom domain? such as alex@proexample[.]com




