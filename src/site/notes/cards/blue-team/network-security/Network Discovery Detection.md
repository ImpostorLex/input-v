---
{"dg-publish":true,"permalink":"/cards/blue-team/network-security/network-discovery-detection/","tags":["blue-team/network-analysis"]}
---

~ [[atlas/Network-Traffic Analysis\|Network-Traffic Analysis]] | ~ [[atlas/blue-team\|blue-team]]
### Introduction 
---
The first stage of an attack: gathering information about the target, usually organization's assets that are open to the publicly accessible Internet (this is known as _attack surface_).

Attackers:

- What assets can be accessed by the attacker? 
- What are the IP addresses, ports, OS, and services running on these assets?
- What versions of services are running? Does any service have a vulnerability that can be exploited?

Defenders:

- What assets can be accessed by the attacker? 
- What are the IP addresses, ports, OS, and services running on these assets?
- What versions of services are running? Does any service have a vulnerability that can be exploited?

The problem is differentiating the **good from the bad** as there are known multiple research organization, web crawlers, search engines, and more., that also perform discovery actions to map the resources present on the Internet.

Defenders often use these techniques:

- Allowlist known internal and benign external scanners, ensuring no alerts are triggered on those sources.

- Integrate Threat Intelligence with detection use cases and flag scanning activity only from known malicious or suspicious sources.

- Since the previous point has a chance of missing some malicious scanning activity, some teams use the Threat Intelligence to raise the severity of the alerts instead of only triggering alerts on them. In addition, they add some generic use cases to alert on scanning behavior.

#### Key Topics
---

## Prerequisites

- Linux Fundamentals
- Network Security Fundamentals

## 1. External vs Internal Scanning
---
### External Scanning
---
When you encounter a scanning activity outside from your organization network, scanning machines inside the network (mainly these are the public facing assets) analyzing the external source IP indicates that the attacker is still on the Reconnaissance phase of the MITRE ATT&CK lifecylce, the attacker does not have a foothold inside the network yet.

![Network Discovery Detection.png|450](/img/user/cards/blue-team/network-security/images/Network%20Discovery%20Detection.png)

This should be considered as a low-severity type of scanning, in response as a defender you can block the offending IP address on the Firewall. However, the attacker might come back with a different IP address.

### Internal Scanning Activity
---
This is internal-to-internal scanning activity. The attacker has already established a foothold and has progressed to the Discovery phase of the MITRE ATT&CK and is now gearing up for lateral movement.

![Network Discovery Detection-1.png|450](/img/user/cards/blue-team/network-security/images/Network%20Discovery%20Detection-1.png)

This should be considered **high severity** but ensure this is not some authorized activity, in this case defenders should escalate this alert and initiate a Incident Response process, these requires deeper investigation and root cause analysis.

## 2. Horizontal vs Vertical Scanning
---
Once the attackers know what hosts are present on the network, they want to identify what ports are open on these hosts. This is called a **port scan**.

**Horizontal Scanning:**

This is where the attacker scan for the same port number over multiple endpoints/IP addresses. With the goal of exploiting a specific port. An example is WannaCry ransomware, which spread through the network using an SMBv1 vulnerability and scanned for machines with port 445.

![Network Discovery Detection-2.png|450](/img/user/cards/blue-team/network-security/images/Network%20Discovery%20Detection-2.png)

Detection:

- Same source IP address, a single destination port, but multiple destination IP address.

**Vertical Scanning:**

Is the opposite of horizontal scanning, this is where attacker scans one endpoint across multiple ports. This is done if the attacker thinks this endpoint is a high value target or the only asset publicly accessible from the Internet.

![Network Discovery Detection-3.png|450](/img/user/cards/blue-team/network-security/images/Network%20Discovery%20Detection-3.png)
## 3. The Mechanics of Scanning
---
Now that we understand the different types of network discovery scans that can be run

**Ping Sweep:**

This is one of the most basic network scanning techniques. Ping sweeps are generally used to identify hosts present (and online) on a network. This scan is run by sending an Internet Control Message Protocol (ICMP) packet to the host. If the host is online, it will reply with an ICMP packet of its own. Often blocked by security controls.

**TCP SYN Scans**

**UDP Scan**

203.0.113.25 > 192.168.230.145
### Questions and Problems
---
## Conclusion


