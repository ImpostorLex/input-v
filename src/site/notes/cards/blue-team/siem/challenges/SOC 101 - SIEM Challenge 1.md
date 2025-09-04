---
{"dg-publish":true,"permalink":"/cards/blue-team/siem/challenges/soc-101-siem-challenge-1/"}
---

~ [[atlas/blue-team\|blue-team]]

- File reference: `04_SIEM/01_Log_Analysis/Challenges/challenge.log`
- The log file is semi structured.
- Source: Apache web server.
- First timestamp:
- Last timestamp: 18/Jul/2024:15:59:25 -0400


```bash
ls -lh access.log && wc access.log
```

There are 343 possible events with a file size of 71749 bytes:
![SOC 101 - SIEM Challenge 1.png](/img/user/cards/blue-team/siem/images/SOC%20101%20-%20SIEM%20Challenge%201.png)

```bash
cut -d " " -f 1 challenge.log | sort| uniq -c | sort -nr
```

Extracting IP address and counting the number of occurrences each, shows three:
![SOC 101 - SIEM Challenge 1-1.png](/img/user/cards/blue-team/siem/images/SOC%20101%20-%20SIEM%20Challenge%201-1.png)
Investigating the top 'talker' first as uploading the three IP address in VirusTotal shows that **182.87.64.64** is flagged as malicious.

```bash
grep "182.87.64.64" challenge.log | wc
```

It shows that 23 of the said number of events belongs to this IP address.

```bash
grep "182.87.64.64" challenge.log | cut -d "\"" -f 6 access.log | uniq -c
```

Nothing out of the ordinary when extracting user agent:

![SOC 101 - SIEM Challenge 1-2.png](/img/user/cards/blue-team/siem/images/SOC%20101%20-%20SIEM%20Challenge%201-2.png)
```bash
grep "182.87.64.64" challenge.log| cut -d "\"" -f 2 | uniq -c
```

Extracting requested URLs by the suspicious IP address, shows that the attacker performs SQL Injection:

![SOC 101 - SIEM Challenge 1-3.png](/img/user/cards/blue-team/siem/images/SOC%20101%20-%20SIEM%20Challenge%201-3.png)
```bash
cat challenge.log | cut -d "\"" -f 6 | sort | uniq -c
```

Found a suspicious mispelled user-agent string:

![SOC 101 - SIEM Challenge 1-4.png](/img/user/cards/blue-team/siem/images/SOC%20101%20-%20SIEM%20Challenge%201-4.png)
Output:
![SOC 101 - SIEM Challenge 1-5.png](/img/user/cards/blue-team/siem/images/SOC%20101%20-%20SIEM%20Challenge%201-5.png)