---
{"dg-publish":true,"permalink":"/cards/blue-team/detection-engineering/tactical-detection/","tags":["sunday"]}
---

[[Detection Engineering Map\|Detection Engineering Map]]
### Introduction 
---
The importance of having the correct mindset in alerting and detecting threats, IOAs, IOCs, and etc rather than just simply relying on default ruleset provided by their security tools.
## Unique Threat Intel
---
Indicator of Compromise (IOCs) from previous intrusion are good example of Threat Intel including these IOCs to our detection mechanism will help spot re-intrusion from specific adversary.

![Tactical Detection.png](/img/user/cards/blue-team/detection-engineering/images/Tactical%20Detection.png)

Logging these IOCs in a spreadsheet allows for better collaboration, also makes scoping of the incident more effective that will most likely lead to more IOCs being found. (Now a great time to add them to our detection mechanism such as Yara or Sigma.)
## Public Generated IOCs
---
If there is currently a zero day vulnerability that our organization is susceptible, we can rely to reports from organizations or communities that encounters or currently studying it.

Once detected they will most likely have a Sigma rule to detect the zero-day vulnerability, and we can use tools such as [Uncoder](https://uncoder.io/) to translate this Sigma rules into our SIEM of choice.

![Tactical Detection-1.png](/img/user/cards/blue-team/detection-engineering/images/Tactical%20Detection-1.png)
### Tripwires
---
One way to create immediate impact is by leveraging specific data, usually access to it is controlled by the IT team such as a **sensitive files**, in this way we can set alert for access, modification, and deletion of sensitive files.

Or the use of **honeypots**, they serve no legitimate business purpose, so any activities concerning them should raise a red flag, and next is **hidden files and folders** they aren't meant to be seen by normal end users, this work best with dealing with crawlers like a worm for detecting intrusions.

One of the easiest tripwire is using **local security policy** and enabling audit object access policy then select files (ideally inside a folder so you can easily manage multiple files) and use EventID 4663.
### Purple Teaming
---
The idea of purple teaming is to test your defenses against an attack such as you want to understand how a certain vulnerability works, and what it looks like on the logs, or you simply want to know the extent of your visibility on your environment.

Consequently, reflect on the following questions:

- What are the attacker techniques that you did?
- Which ones did you detect?
- Which ones flew under the radar?

## Conclusion
---




