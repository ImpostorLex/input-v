---
{"dg-publish":true,"permalink":"/soc-workbooks-and-lookups/"}
---

[[]]
### Introduction
---
Imagine the following:

You are having a night shift and looking into an alert saying that G.Baker logged into the HQ-FINFS-02 server. Then, the user downloaded the "Financial Report US 2024.xlsx" file from there and shared it with R.Lund. To correctly triage the alert and understand if the activity is expected, you will have to find answers to many questions:

- Who is G.Baker? What are their working hours and role in the company?
- What is the purpose and location of HQ-FINFS-02? Who can access it?
- Why could R.Lund need access to the corporate financial records?
## Identity Inventory
---
Is a catalogue or a list of corporate employees (user accounts), services (machine accounts), and their details such as privileges, contacts, and roles within the company it can also be called **assets**.

**Source of Identities:**
- Active Directory
- Custom IT solutions such as CSV or Excel sheets.
- SSO providers such as Okta, Google Workspace

In our case:

**Employee**

| **Full Name** |         |                      | **Role**                |            |                  |
| ------------- | ------- | -------------------- | ----------------------- | ---------- | ---------------- |
| Gregory Baker | G.Baker | g.baker@tryhatme.thm | Chief Financial Officer | Europe, UK | VPN, HQ, FINANCE |
| Raymond Lund  | R.Lund  | r.lund@tryhatme.thm  | US Financial Adviser    | US, Texas  | VPN, FINANCE     |
**Asset**

| **hostname** |               |              |                     |            | **Purpose**                       |
| ------------ | ------------- | ------------ | ------------------- | ---------- | --------------------------------- |
| HQ-FINFS-02  | UK Datacenter | 172.16.15.89 | Windows Server 2022 | Central IT | File server for financial records |
Making our case legitimate.
### Network Diagrams
---
It is a visual schema presenting existing locations, subnets, and their connections.

![SOC Workbooks and Lookups.png](/img/user/SOC%20Workbooks%20and%20Lookups.png)
#### Workbooks Theory
---
Now that you have enough context thanks to asset inventory and network diagram, now your next task is to make a verdict whether the activity is true positive or false positive.

_Workbooks_ are specific step by step activities to take on specific encounters, junior analyst most of the time are required to follow it specifically due to their lack of experience.

Example of a workbook:
![SOC Workbooks and Lookups-1.png|500](/img/user/SOC%20Workbooks%20and%20Lookups-1.png)

1. **Enrichment**: Use Threat Intelligence and identity inventory to get information about the affected user
2. **Investigation**: Using the gathered data and SIEM logs, make your verdict if the login is expected
3. **Escalation**: Escalate the alert to L2 or communicate the login with the user if necessary

