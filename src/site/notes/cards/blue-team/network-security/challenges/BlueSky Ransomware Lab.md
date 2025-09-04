---
{"dg-publish":true,"permalink":"/cards/blue-team/network-security/challenges/blue-sky-ransomware-lab/"}
---

~ [[atlas/blue-team\|blue-team]] 

Dealing with a total of 4797 packet capture:
![BlueSky Ransomware Lab.png](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab.png)
At protocol hierarchy, we are dealing with DNS, ICMP, SSDP, and HTTP(s).

![BlueSky Ransomware Lab-1.png|300](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab-1.png)
Interestingly, the packet captures public IP address and the one who seems to be communicating with multilple endpoint is: **87.96.21.81:**

![BlueSky Ransomware Lab-2.png](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab-2.png)
Immediately at first glance **21.81** made a DNS query to **g.live.com** and **config.edge.skype.com**:

![BlueSky Ransomware Lab-3.png](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab-3.png)
**Investigating http**
**21.81** made a lot of suspicious GET request to **21.84** such as `Invoke-PowerDump.ps1`, `Invoke-SMBExec.ps1`, `Extracted_hosts.txt`, and `javaw.exe` to name a few.
![BlueSky Ransomware Lab-4.png](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab-4.png)
All of the `.ps1` files have specific purpose - `checking.ps1` most notably downloads `.ps1` that deletes it's traces, disable Windows Defender, and check for the current user's privileges.

`del.ps1` disable process viewer such as Task Manager, Process Hacker and many more:

![BlueSky Ransomware Lab-5.png](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab-5.png)
My initial guess is **21.84** is either compromised and then move laterally to **21.81** then **21.84** host various tools, viewing the ICMP packet shows that **21.84** perform a port scan against **21.81**:

![BlueSky Ransomware Lab-6.png](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab-6.png)
Based on the previous finding the attacker did manage to move to or at least compromised the **21.81** machine, at the `.evtx` file let's look for login events:

Unfortunately no events found related to log on.

Moving back to the HTTP GET request, analyzing the `ichigo-lite.ps1` content, we can see this:

![BlueSky Ransomware Lab-7.png](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab-7.png)
Back at the event viewer, lets filter out for 403, 400, and 600 as they show powershell related logs:

![BlueSky Ransomware Lab-8.png](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab-8.png)
Unfortunately I did not find anything related but moving back at the **protocol hierarchy** we can see this:

![BlueSky Ransomware Lab-9.png](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab-9.png)
> The **TDS protocol** (Tabular Data Stream protocol) is a **communication protocol** used by **Microsoft SQL Server** and **Sybase** to transfer data between a client application and the database server

At login, we can see one username being logged in which is **sa** and it's password:

![BlueSky Ransomware Lab-10.png](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab-10.png)
Then viewing the slightly formatted SQL query:

![BlueSky Ransomware Lab-11.png](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab-11.png)
The `xp_cmdshell` allows for the SQL server to execute malicious commands.

Filtering for powershell events in event viewer using 400, 403, and 600: 

> Process injection is often used by attackers to escalate privileges within a system. What process did the attacker inject the C2 into to gain administrative privileges?

![BlueSky Ransomware Lab-12.png](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab-12.png)

Then after the privilege escalation the attacker downloaded:

```C
http://87.96.21.84/checking.ps1
```

It checks for security group which is:

```
S-1-5-32-544
```

The attacker creates a cleaner using `schtasks.exe`:

![BlueSky Ransomware Lab-13.png](/img/user/cards/blue-team/network-security/images/BlueSky%20Ransomware%20Lab-13.png)
extracted_hosts.txt

145 mins or 2 hours and 20 mins with hints.



