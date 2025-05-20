---
{"dg-publish":true,"permalink":"/cards/blue-team/soc/soc-metrics-and-objectives/"}
---

[[]]
### Introduction
---
Like any other departments, measuring the effiiciency of the SOC team through different indicators and metrics is vital and shows also potential consequences of ignoring them.
## Core Metrics
---
The SOC team performs its purpose by developing, receiving, and triaging alerts. The L1 analyst's role is to analyse alerts that are true positives then reports to the higher level (L2 Analyst).

| Metric                | Formula                                  | Measures                     |
| --------------------- | ---------------------------------------- | ---------------------------- |
| Alerts Count          | `AC = Total Count of Alerts Received`    | Overall load of SOC analysts |
| False Positive Rate   | `FPR = False Positives / Total Alerts`   | Level of noise in the alerts |
| Alert Escalation Rate | `AER = Escalated Alerts / Total Alerts`  | Experience of L1 analysts    |
| Threat Detection Rate | `TDR = Detected Threats / Total Threats` | Reliability of the SOC team  |

- **Alerts Counts:** the ideal metric value depends on the company size but the general value is **5 to 30 alerts per day per L1 analyst is a good metric**. Too many alerts goes unresolved is overwhelming and prone to missing real threats, too low count of alerts may indicate a issue in the SIEM or lack of visibility allowing real threats go unseen.

- **False Positive Rates:** an **80% or greater false positive rate** is a serious problem and may lead analyst to _become less vigilant and more likely to miss the threat, and treat all alerts as 'spam'_.  An 0% false positive rate is unreachable.

- **Alert Escalation rate:** Level 2 Analyst rely on L1 Analyst to filter out the noise and only escalate the actionable, and threatening alerts. L1 analyst should ask for senior support against alerts they don't fully understand. ideal metric is below **50%, or even better below 20%**.

- **Threat Detection Rate:** Imagine that out of six attacks for 2025, your SOC team detected and prevented four attacks, missed the fifth because of the broken detection rule, and the sixth because one of the L1 analysts misclassified the breach as False Positive. The resulting metric is TDR = 4 / 6 = 67%, and this is a very bad result. Ideal detection metric rate should be **100%** since every missed detection have devastating consequence.

## Triage Metrics
---
It's important to know that an _alert_ won't stop the breach, it's important to timely receive the alert, triage it, and respond to the attack before the attackers achieve their goals. The requirements to ensure a quick detection and remediation of the threat are commonly grouped into a **Service Level Agreement (SLA)** - a document signed between the internal/external SOC team and its company management:

- **Mean Time to Detect:** Is the average time it takes for an organization to identify a security threat, or a technical problem. (5 minutes)
- **Mean Time to Acknowledge** is the average time it takes for the service provider (in this case, the L1 Analyst) to take action after the initial alert is generated. (10 minutes)
- **Mean Time to Response (MTTR)** is the average time between the initial alert and response to it (such as malware removal, password resset, or system restore). (60 minutes)

![SOC Metrics and Objectives.png](/img/user/cards/blue-team/soc/images/SOC%20Metrics%20and%20Objectives.png)

> [!question]- Consider the scenario
> Imagine a scenario where an employee was lured into running data stealer malware.  
>   1. The SOC team received the "Connection to Redline Stealer C2" alert after **12** minutes.  
>   2. One of the L1 analysts on shift moved the alert to In Progress **10** minutes later.  
>   3. After **6** minutes, the alert was escalated to L2, who spent **35** minutes cleaning the malware.
> 
> `Answer: 12, 10, 51`
> 
> Explanation:
> 
> - `Mean Time to Detect (MTTD)` is 12 minutes.
> - `Mean Time to Acknowledge (MTTA)` is 10 minutes because it took the analyst 10 minutes to take action.
> - `Mean Time to Response (MTTR)` is 51 minutes because 12 (MTTD) + (MTTA) 10 (L1 delay) + 6 (before escalation or triaging) + 35 (L2 cleanup) = 51.
### Improving Metrics
---
Metrics exist to make the SOC team more efficient and, therefore to make attacks less successful, second the metric is used to evaluate your performance, and good results lead to career growth.

|Issue|Recommendations|
|---|---|
|False Positive Rate  <br>over 80%|**Your team receives too much noise in the alerts. Try to:**  <br>  <br>1. Exclude trusted activities like system updates from your EDR or SIEM detection rules  <br>2. Consider automating alert triage for most common alerts using SOAR or custom scripts|
|Mean Time to Detect  <br>over 30 min|**Your team detects a threat with a high delay. Try to:**  <br>  <br>1. Contact SOC engineers to make the detection rules run faster or with a higher rate  <br>2. Check if SIEM logs are collected in real-time, without a 10-minute delay|
|Mean Time to Acknowledge  <br>over 30 min|**L1 analysts start alert triage with a high delay. Try to:**  <br>  <br>1. Ensure the analysts are notified in real-time when a new alert appears  <br>2. Try to evenly distribute alerts in the queue between the analysts on shift|
|Mean Time to Respond  <br>over 4 hours|**SOC team can't stop the breach in time. Try to:**  <br>  <br>1. As L1, make everything possible to quickly escalate the threats to L2  <br>2. Ensure your team has documented what to do during different attack scenarios|


