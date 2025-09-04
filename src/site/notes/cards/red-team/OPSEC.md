---
{"dg-publish":true,"permalink":"/cards/red-team/opsec/","tags":["red-team"]}
---

~ [[atlas/red-team\|red-team]]
### Introduction
---
A process on securing the plan and execution of sensitive activities by identifying, controlling, and protecting unclassified evidence.

OPSEC has five steps:

1. Identify critical information
2. Analyse threats
3. Analyse vulnerabilities
4. Assess risks
5. Apply appropriate countermeasures

In a red teamer's point of view, the blue teamers are the adversary we don't want them to know that we are planning on attacking their infrastructure and vice versa. Frameworks such as the Cyber Kill Chain and MITRE ATT&CK helps defenders identify the objectives of the adversary.

Each interaction made by red team leaves a 'dot' that dot can be connected by the blue team to other dots.
#### Key Topics
---

- [[#Critical Information Identification|Details the types of information (client, tools, IPs, domains) that must be protected to avoid mission compromise.]]
    
- [[#Threat Analysis|Covers identifying adversaries, their goals, and their methods—defining threats based on intent and capability.]]
    
- [[#Vulnerability Analysis|Explains OPSEC-specific vulnerabilities—how leaked information can be used by blue teams to disrupt red team activities.]]
    
- [[#Risk Assessment|Assesses likelihood and impact of events, and how adversaries might exploit vulnerabilities—helps prioritize risks.]]
    
- [[#Counter Measures|Outlines how to prevent, deceive, or deny adversary access to critical information using techniques like alternate IPs.]]

- [[#Operating Procedures for Red Team OPSEC|Basic Setup for Red Team operations.]]

## Critical Information Identification
---
These are information regardless whether it is technical or not technical that once obtained by the blue team would put a 'stop', degrade, or make the execution of the red team's mission tough.


- **Client Information** -- it should be a need-to-know basis as it could jeopardise the red team's mission. The Principle of Least Privilege (PoLP) should be applied.
- **Red team information** -- identities, activities, plans, capabilities, and limitations.
- **Tactics, Techniques, and Procedures** that your team is going to use.
- OS, cloud hosting provider, or C2 framework utilised by your team. Let’s say that your team uses [Pentoo](https://pentoo.github.io/) for penetration testing, and the defender knows this. Consequently, they can keep an eye for logs exposing the OS as Pentoo.
- **Public IP address**
- **Domain names**
- **Hosted websites**

## Threat Analysis
---
After identifying critical information that must be protected, it's time to analyze our threats:

1. Who is the adversary?
2. What are the adversary’s goals?
3. What tactics, techniques, and procedures does the adversary use?
4. What critical information has the adversary obtained, if any?

We consider any adversary with the intent and capability to take actions that would prevent us from completing our operation as a threat:

```C
threat = adversary + intent + capability
```

Becareful of data leaks as this could damage the reputation of the client such as client's customer data and as well as reveals the red team's parent organization

## Vulnerability Analysis
---
It is different from cybersecurity vulnerability, it is called an **_opsec vulnerability_**, it is when an adversary obtains critical information, analyse the findings, and act in a way that would affect your plans.

For example:

You used the same IP address to scan the target's network, host a phishing website, and used metasploit framework to attempt to exploit a certain software vulnerability, once detected by the blue teamers they could easily block your IP address and effectively stopping all three operations.
## Risk Assessment
---
In red team OPSEC, **risk assessment** means evaluating:

- **How likely** a vulnerability will be discovered by the blue team
- **How damaging** it would be if exploited

Once risks are understood, we decide if countermeasures are needed, based on:

1. **Effectiveness** – How well does it reduce the risk?
2. **Cost vs. Impact** – Is it worth the effort compared to the damage it prevents?
3. **Detection Risk** – Could the countermeasure itself expose your operation?

**Example 1: Shared Public IP**

- **Risk:** High if blue team uses SIEM (they can correlate scanning, phishing, and exploits)
    
- **Countermeasure:** Use separate IPs for each activity
    
- **Trade-off:** More infrastructure needed, but much harder to track

## Counter Measures
---
Providing countermeasures to prevent an adversary from detecting critical information, provide an alternative intrepretation of critical information (fake data | deception), or deny the adversary's collection system.

Moving back to the first example:

Using the same IP address for the said three activities, the counter measure should be easy: use different IP address for each activities.


If dot(s) is unavoidable don't leave clues that can be associated with other dot(s) or create dot(s) that can be associated with other dot(s).
## Operating Procedures for Red Team OPSEC
---

### Local workstation setup
---

1. Ideally virtual machines that can be easily be cloned or clean snapshot - no information from previous clients whatsover.- 
	- Should be easily modified/updated without needing of full rebuild.
	- Have a checklist for your build and configuration process
2. Change the hostname to imitate normal devices from the client's network - no obvious hacking names.
	- Remote: localhost, DESKTOP, PC
	- On-site: PRINTER, deskjet, LEXMARK

In red team operations, changing the domain name helps avoid detection when connecting to the target’s VPN or simulating internal systems.

### Key Points:

- **VPN Security Checks**: Many VPNs inspect system details like domain name, hostname, and user credentials.
    
- **Detection Risk**: If your system isn't joined to the expected domain (e.g., ACMECORP.local), the blue team can easily flag it as suspicious.
    
- **Payload Testing**: Some payloads or malware are configured to only execute on systems with a specific domain. Matching the domain name helps test these accurately.
    
- **EDR/AV Checks**: VPNs may also check for specific endpoint protection software that is domain-bound.


**Changing the default User-Agent:**

In `~/.bashrc` or `~/.zshrc`:

```C
export AGENT="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36" 
alias curl="curl -A '$AGENT'" 
alias wget="wget -U '$AGENT'" alias nmap="nmap --script-args=\"http.useragent='$AGENT'\""
```

This effectively changes the default-user agent used by tools to a not so obvious user-agent.

Another example is:

```C
wpscan --ua "$AGENT" --url 127.0.0.1
```

- Additionally, change the default user agent in firefox browser setting as well.


https://www.blackhillsinfosec.com/wp-content/uploads/2021/03/SLIDES_OPSECFundamentalsRemoteRedTeams-1.pdf
### Source IP address
---

1. Avoid using the same IP address for different activities.
2. Physical location: the client is located in Los Angeles but the red team accessed the organization's network through VPN but is located in Atlanta that could easily raise suspicions, enabling the blue team to contact the user for 'impossible travel' or account compromised.
3. **Service Provider:** sure you have three different IP address but the problem is all of them are from the same provider such as Linode then the blue team will simply increase their monitoring to any IP address resolving to Linode.

#### Countermeasures
---

1. Never use the same IP address for two activities that you don't want to be associated with each other.
2. Login from an IP address in the same region as the user (not the organization as VPN or work from home is possible).
3. Avoid known-suspicious IPs: TOR, proxies or any cloud computing IP address that was recently flagged by tools such as VirusTotal, Cisco talos intelligence.

## Third Party Services
---

- Ensure that your third party service does not expose your registration information.
- Assess whether **red team actions are likely to violate TOS** 
	- **Modified Identity**: Using fake names, emails, phone numbers, or billing info can breach TOS.
	- **Payment Info**: Fake or reused credit card data is often explicitly prohibited.
	- **IP Spoofing**: Masking or rotating IPs to bypass limits may be seen as abuse.
- Assess whether use of the **same account** across multiple projects to avoid leaking red team or it's customer's information
- **Avoid typosquatting target domain name, this includes subdomain** as experienced analyst could easily use tools such as dnstwister to detect this stuff.

![Pasted image 20250603103039.png](/img/user/cards/red-team/images/Pasted%20image%2020250603103039.png)

- Use private registration/WHOIS privacy
- One red team action per domain name (same concept as source IPs).
- No default or self-signed certificates - easily flagged and blocked. Avoid Let's Encrypt as it is most common in lot's of hacking tutorials.
- **Avoid data leaks in certificate transparency logs**

## Payloads and Network Services
---


- Don't expose services to the Internet that are not required, use SSH port forwarding for access from the red team
- Change default settings on hacking tools that listen on an Internet-facing port.

#### Cobalt Strike Team Server Study

- **Indicator**: TCP Port `50050` – Default Cobalt Strike Team Server port  
- **OPSEC Tip**: Don’t expose services to the Internet that aren’t required  
- **Mitigation**: Use SSH port forwarding for red team access
#### DNS Behavior

- **Indicator**: Cobalt Strike DNS server responds with IP `0.0.0.0`
#### Default Settings in Hacking Tools

- **OPSEC Tip**: Change all default settings for tools listening on Internet-facing ports  
- **Indicator**:  
  - Web root returns `404 Not Found`, no content  
  - `Content-Type: text/plain`  
  - Default response when nothing is hosted at `/` or when no redirector is used
#### Service Fingerprinting

- **Indicator**: JA3S (TLS server fingerprinting) reveals unique tool-based signatures  
- **OPSEC Tip**: Use common web services like **Apache** or **Nginx** to proxy or redirect HTTP/S traffic to backend servers running hacking tools
#### Executable Payloads – Metadata Risks

- Often used to gather names, usernames, and organizational details of the target  
- Can also **reveal red team infrastructure or operator artifacts** if not properly sanitized  
- **Common phishing payloads**:  
  - Microsoft Office documents  
  - PDF files  
  - Other document-based exploits
