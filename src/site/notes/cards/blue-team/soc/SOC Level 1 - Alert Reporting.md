---
{"dg-publish":true,"permalink":"/cards/blue-team/soc/soc-level-1-alert-reporting/"}
---

[[atlas/blue-team\|blue-team]]
### Introduction
---
During or after alert triage, L1 analysts may be uncertain about how to classify the alert, requiring senior support or information from the system owner.
## Pre-requiste
---
- [[cards/blue-team/soc/SOC L1 Triage\|SOC L1 Triage]]

## Alert Funneling
---
It all starts with Level 1 analyst receiving an alert:

![SOC Level 1 - Alert Reporting.png|450](/img/user/cards/blue-team/soc/images/SOC%20Level%201%20-%20Alert%20Reporting.png)
## Reporting Guide
---
Raw Security Information and Event Management (SIEM) logs are stored for 3 - 12 months, but alerts are kept indefinetly so it's **important** to keep all context inside the alert.

- **Who**: Which user logs in, runs the command, or downloads the file
- **What**: What exact action or event sequence was performed
- **When**: When exactly did the suspicious activity start and ended
- **Where**: Which device, IP, or website was involved in the alert
- **Why**: The most important W, the reasoning for your final verdict

![SOC Level 1 - Alert Reporting-1.png|450](/img/user/cards/blue-team/soc/images/SOC%20Level%201%20-%20Alert%20Reporting-1.png)
### Escalation Guide
---
1. The alert is an indicator of a major cyberattack requiring deeper investigation or DFIR
2. Remediation actions like malware removal, host isolation, or password reset are required
3. Communication with customers, partners, management, or law enforcement agencies is required
4. You just do not fully understand the alert and need some help from more senior analysts
### Crisis Communication
---
- **You need to escalate an urgent, critical alert, but L2 is unavailable and does not respond for 30 minutes.**  
	- Ensure you know where to find emergency contacts. First, try to call L2, then L3, and finally your manager.
    
- **The alert about Slack/Teams account compromise requires you to validate the login with the affected user.**  
    - Do not contact the user through the breached chat - use alternative contact methods like a phone call.
    
- **You receive an overwhelming number of alerts during a short period of time, some of which are critical.**  
    - Prioritise the alerts according to the workflow, but inform your L2 on shift about the situation.
    
- **After a few days, you realise that you misclassified the alert and likely missed a malicious action.**  
    - Immediately reach out to your L2 explaining your concerns. Threat actors can be silent for weeks before impact.
    
- **You can not complete the alert triage since the SIEM logs are not parsed correctly or are not searchable.**  
    - Do not skip the alert - investigate what you can and report the issue to your L2 on shift or SOC engineer.