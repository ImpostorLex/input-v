---
{"dg-publish":true,"permalink":"/cards/blue-team/threat-intelligence/threat-intelligence-for-soc/"}
---

[[cards/blue-team/threat-intelligence/Threat Intelligence\|Threat Intelligence]]
### Introduction
---
As the threat actors evolves so should the defenders, the defenders should apply known indicators from reliable sources in to their Security Operations pipeline, and determine unknown adversaries.

- _Producers_ -- are large group (usually) that collects, gathers, and share threat intelligence to consumers.
- _Consumers_ -- are organizations or individuals that consumes threat intelligence produced by producers.
## Hunting and Consuming IOCs
---
[Uncoder.io](https://uncoder.io/) is an online tool that transforms Sigma rules, IOC lists, and other platform query syntaxes into custom hunting queries prepared for execution in SIEM and XDR

In this example, Elastic will be our Security Information & Event Management (SIEM) and a list of IOCs:

```C
117[.]213[.]7[.]8
119[.]180[.]220[.]224
144[.]202[.]127[.]44
119[.]180[.]220[.]224
221[.]15[.]94[.]231
```

Then at the uncoder website:

![Threat Intelligence for SOC.png](/img/user/cards/blue-team/threat-intelligence/images/Threat%20Intelligence%20for%20SOC.png)
It's that easy to create a query based on Indicators of Compromise (IOCs) and paste it to our SIEM.

## Intelligence-driven Prevention
---

1. IP blocking via Firewall (for both ingress and egress).
2. Domain Blocking through Email Gateways.
3. DNS sinkhole is a security measure that prevents user redirecting to known malicious domains.

## Intelligence-driven Detection
---
You already blocked them that's _prevention_, now you need to **detect** them if anyone tries to connect to the blocked IPs, domains, and URLs.

Instead of manually creating detection rules for each **Indicator of Compromise (IOC)** (which can become cumbersome and difficult to manage), it’s more effective to **update a blocklist** with known IOCs and **investigate** when these are **detected**.

The following sigma rule is used to detect any DNS resolving to `0.0.0.0` which is configured as a DNS Sinkhole:

```
title: DNS Sinkhole
author: TryHackMe User
description: Sigma rule for sinkholed DNS queries 
logsource:
 category: dns
detection:
 select_sinkholed:
   dns.resolved_ip:
     - '0.0.0.0'
 condition: select_sinkholed
falsepositives:
 - Unknown
status: experimental
level: medium
tags:
 - dns
 - filebeat
```

Then back at uncoder.io:

![Threat Intelligence for SOC-1.png](/img/user/cards/blue-team/threat-intelligence/images/Threat%20Intelligence%20for%20SOC-1.png)

Then navigate to _elastalert_ which is a command-line based tool that accepts a Elastalert query and then alerts every finding, it also has the ability to alert analyst through common communication medium:

1. `cd ~/elastalert/rules`
2. Create a `.yaml` file then paste the output in uncoder.io.
3. Execute the command starting from X time:

```C
elastalert --start 2023-02-16T00:00:00 --verbose 2>&1 | tee output.txt
``````

