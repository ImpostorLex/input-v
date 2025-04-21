---
{"dg-publish":true,"permalink":"/simmering/using-smart-screen-logs-to-find-evidence-of-execution-and-user-activity-analysis/","tags":["sunday","template"]}
---

[[cards/blue-team/digital-forensics/Digital Forensics\|Digital Forensics]]

source: https://www.hackthebox.com/blog/smartscreen-logs-evidence-execution
### Introduction 
---
In digital forensics or any forensics, the more evidence you have the stronger your case is. Although there are [[map-of-contents/blue-team\|Windows Artifact]] such as RecentDocs registry key, jumplists, and ShimCache that provides proof of execution and user activity, it can't hurt the investigation and it might provide more insights.
#### Microsoft Defender SmartScreen
---
It is a feature/tool that comes with the Windows Operating System that is designed to protect users from potential malicious actions such as visiting a phishing website, or downloading a malware.

- It looks for suspicious indicators in the website such as behavior, if there is then it shows a **warning**.
- Lookup the visited website in a dynamic list of phishing sites and malicious software sites, if the website exist in the list, it shows the user a warning that **the site might be malicious**.
- It checks the downloaded files against a list of software or programs that is known to be **unsafe**, if it matches it shows a warning that **it might be malicious.**
- Checks the downloaded files against well known and frequently downloaded files, if the downloaded file is not on the list, a warning is given advising **caution**.
##### Limitation
---
- The logging is disabled by default, can be enabled using:
	- `wevtutil sl Microsoft-Windows-SmartScreen/Debug /e:true`
- The logging is Graphical User Interface (GUI) dependent such as the way we commonly use our machine through our devices or using Remote Desktop Protocol (RDP) anything run through PowerShell or CMD will not be recorded.

## Prerequisites

- Windows Machine.
- The order of volatility.
## Finding Execution of Evidence

There are many Windows artifacts that can confirm executable executions, these are Prefetch, LNK files, RecentDocs, UserAssists, BAM, and more but SmartScreen stands out by:

1. Logs are written in real time.
2. Easy to analyze or well-formatted.
3. It can easily be fed to any SIEM without additional tooling.

First, open cmd in administrator mode:
```C
wevtutil sl Microsoft-Windows-SmartScreen/Debug /e:true
```

![Using SmartScreen logs to find evidence of execution and user activity analysis.png](/img/user/+%20simmering/Using%20SmartScreen%20logs%20to%20find%20evidence%20of%20execution%20and%20user%20activity%20analysis.png)

Open Event Viewer, then go to: **Applications and Services Logs -> Microsoft -> Windows -> Smartscreen-> Debug**

![Using SmartScreen logs to find evidence of execution and user activity analysis-1.png](/img/user/+%20simmering/Using%20SmartScreen%20logs%20to%20find%20evidence%20of%20execution%20and%20user%20activity%20analysis-1.png)
Open `cmd.exe` to create a log:



If it does not work try increasing the maximum log size, by right-clicking and choosing properties:

![Using SmartScreen logs to find evidence of execution and user activity analysis-2.png](/img/user/+%20simmering/Using%20SmartScreen%20logs%20to%20find%20evidence%20of%20execution%20and%20user%20activity%20analysis-2.png)






### Advices, Examples/Case Studies, Questions and Problems
---

## Conclusion 
---


