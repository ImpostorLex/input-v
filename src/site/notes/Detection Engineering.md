---
{"dg-publish":true,"permalink":"/detection-engineering/","tags":["sunday","template"]}
---

[[map-of-contents/blue-team\|blue-team]]
### Introduction 
---
It's a **continuous process** of identifying threats, detecting them through rules, and processes, and fine-tune the rules and processes as the cybersecurity landscape changes.

There are two types of detection:

1. **Environment-based** it focuses on changes in an environment based on configuration and baseline that have been defined.
2. **Threat-based** focuses on elements that is associated with the adversary, these are tactics, tools, and artefacts.

## Detection Engineering Methodologies
---

1. **Detection Gap Analysis** -- Identifying key areas in our organization's environment where threat detection can be improved, this process is also known as **threat modelling**, and it can be done in two ways.
	- Reactive -- Assesing the most recent incidents, taking notes of the lesson learnt from the attacks, and **identifying missed areas of detection.**
	- Proactive -- Using the [[cards/blue-team/threat-hunting/Threat Hunting\|MITRE ATT&CK]] framework and various intelligence sources to map out potential areas of attack and various TTPs that adversary may use against our organization.

2. **Datasource Identification and Log Collection** -- Once we understand the potential risks and threat actors our organization may face, the next step is to **identify the data sources** that can help us detect and respond to those risks. This involves reviewing what logs we currently collect (like firewall logs, endpoint logs, cloud service logs, etc.) and determining which important logs or data sources are missing.

3. **Baseline creation** -- Before collecting information about adversaries, and their TTPs, it's important that security analyst knows **what normal behaviour** looks within the organization and what is not, this process requires the participation of all departments.

4. **Log Collection** -- Once **baseline** and sources of **internal data** have been identified and prioritized - a centralised system may aggregate all logs using network sensors for network data and services such as Sysmon to collect host data.

5. **Rule Writing to Deployment, and Tuning** -- Detection rules will need to be written and tested against the data sources. 
	- In the deployment stage the rules should be assessed, modified and updated.
### Frameworks
---

- [[cards/blue-team/threat-hunting/Threat Hunting\|MITRE ATT&CK]]
- [[cards/blue-team/threat-intelligence/Pyramid of Pain\|Pyramid of Pain]] -- detect TTPs cause a lot of pain for adversary.
- [[cards/red-team/Cyber Kill Chain\|Cyber Kill Chain]]

## Constructing Effective Detection and Alerts
---
The [Alerting and Detection Strategy (ADS) framework](https://github.com/palantir/alerting-detection-strategy-framework) has a strict flow to follow before publishing the detection rules into production:

1. **Goal:** Describes the intended reasons for setting up the alert and the type of behaviour that needs to be detected.
2. **Categorization:**  Mapping the detection to the MITRE ATT&CK framework to provide analyst with information on the TTPs for investigation.
3. **Strategy Abstract:** Basically a high level summary **how** will this detection rule work, what data sources will it use, and how to reduce false positive.
4. **Technical Context**: Provides detailed information and background needed for a responder to understand all components of the alert. The goal is to create a **self-contained reference** for a responder to make a judgement call.
5. **Blind Spots and Assumptions:** Describes any issues identified where suspicious activities may not trigger the strategy. Assumptions and blind spots help clarify ways the ADS may fail or be bypassed by an adversary.
6. **False Positives:** Outlines occurrences where alerts may be triggered due to misconfigurations or non-malicious activities within the environment. This makes it easy to configure your SIEM to limit alert generation to only targetted threats when pushed to production.
7. **Validation:** outline all the steps required to produce a true-positive.
	- Develop a plan that will produce a true-positive alert.
	- Document everything.
	- Test and trigger the alert in the testing environment.
	- Validate the **strategy** that triggered the alert.
8. **Priority:**  Explains how seriously it should be treated once detected based on defined **criteria**.
9. **Response:** Provides details of how to triage and investigate a detection alert. 
### Sample ADS
---

|ADS Stage|Description|
|---|---|
|**Goal**|Detect when PowerShell is loaded into an unusual host process.|
|**Categorisation**|[Execution/Powershell](https://attack.mitre.org/wiki/Technique/T1086)|
|**Strategy Abstract**|- Monitor module loads via endpoint tools.<br>- Assess the process that loads PowerShell DLL.<br>- Alert on unusual PowerShell host processes.|
|**Technical Context**|Powershell, built on the .NET framework, is a command-line shell and scripting language for performing system management and automation. It is a DLL entitled **system.management.automation.dll** but also may exist as a native image or through the process **powershell.exe.**<br><br>Attackers leverage Powershell as it provides a high-level interface to interact with the OS without requiring the development of functionality in C, C# or .NET. Sophisticated adversaries may opt for the OPSEC-friendly method of injecting Powershell into non-native hosts, commonly identified as [unmanaged PowerShell.](https://github.com/leechristensen/UnmanagedPowerShell)|
|**Blind Spots & Assumptions**|- Endpoint tools are running correctly.<br>- Endpoint logs are reported and forwarded to the SIEM.<br>- SIEM is indexing endpoint logs successfully.|
|**False Positives**|A legitimate Powershell host is used and not suppressed via a whitelist.|
|**Priority**|Medium|
|**Validation**|Perform the executions:<br><br>`Copy-Item C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Destination C:\windows\temp\unusual-powershell-host-process-test.exe -Force` <br><br>`Start-Process C:\windows\temp\unusual-powershell-host-process-test.exe -ArgumentList '-NoProfile','-NonInteractive','-Windowstyle Hidden','-Command {Get-Date}'` <br><br>`Remove-Item 'C:\windows\temp\unusual-powershell-host-process-test.exe' -Force -ErrorAction SilentlyContinue`|
|**Response**|- Compare suspect PowerShell host against whitelist entries.<br>- Check the digital signature of the binary.<br>- Identify the execution behaviour of the binary.|


#### Detection Maturity Level Model
---
It's a way to measure how good your organization is at **using threat intel to actually detect and respond to cyber threats** — not just collecting cool info, but doing something with it.

- It’s not about how much intel you have — it’s about how well you **use** it to catch bad guys.
- You can’t respond to threats if you don’t even know they’re happening — **detection comes first**.

| Level   | Name                   | What It Means (Simple Version)                                                                 |
|---------|------------------------|-----------------------------------------------------------------------------------------------|
| DML-0   | None                   | No detection at all. You’re flying blind.                                                     |
| DML-1   | Atomic Indicators      | Basic intel feeds. Just matching IPs or domains from blocklists.                             |
| DML-2   | Host & Network Artefacts | Chasing breadcrumbs. Looking for clues like logs, IOCs, or file hashes after an attack.     |
| DML-3   | Tools                  | Catching their tools. Spotting hacking tools either when downloaded or when running.         |
| DML-4   | Procedures             | Recognizing patterns. Detecting attack routines (e.g., scanning before data theft).          |
| DML-5   | Techniques             | Spotting their technique. Recognizing specific threat actor behaviors.                       |
| DML-6   | Tactics                | Seeing the tactic. Not the exact tool, but the type of behavior (e.g., privilege escalation).|
| DML-7   | Strategy               | Understanding their strategy. You know why they’re doing what they’re doing.                 |
| DML-8   | Goals                  | Guessing their goals. Predicting their endgame (e.g., steal secrets), often based on patterns



## Conclusion


