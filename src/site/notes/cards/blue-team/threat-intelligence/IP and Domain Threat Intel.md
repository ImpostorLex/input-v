---
{"dg-publish":true,"permalink":"/cards/blue-team/threat-intelligence/ip-and-domain-threat-intel/"}
---

~ [[atlas/blue-team\|blue-team]]
### Introduction
---
Security Operations runbooks still revolves around the process `verify -> enrich -> decide`, but the enrichment phase is different if we are working with a lone IP address or domain, we rely on geolocation, ASNs, open-service footprints, and passive DNS to learn whether a connection is normal (or routine SaaS traffic) or an adversary foothold.

For this demonstration:

> The SOC has flagged two suspicious domains in phishing emails and three IP addresses in outbound proxy logs. You are tasked with triaging all seven artefacts, enriching them with context, and recommending actions with expiry.

- advanced-ip-sccanner[.]com
- 166[.]1[.]160[.]118
- 64[.]31[.]63[.]194
- 69[.]197[.]185[.]26
- 85[.]188[.]1[.]133

Quick reference here for [[cards/blue-team/IP and Domain Threat Intelligence References\|analyis]].
#### Key Topics
---
- [[cards/blue-team/threat-intelligence/IP and Domain Threat Intel#Prerequisite\|Covers DNS basics, ASNs, and how networks are structured — foundational knowledge before enrichment.]]
    
- [[cards/blue-team/threat-intelligence/IP and Domain Threat Intel#IP Building Blocks\|Explains DNS records (A, NS, MX, TXT, SOA, TTL), attacker abuse like fast flux/CDN abuse, and how analysts triage domains.]]
    
- [[cards/blue-team/threat-intelligence/IP and Domain Threat Intel#IP Enrichment: Geolocation and ASN\|How to use RDAP, ASN context, and geolocation to classify IPs (hosting, residential, CDN, cloud).]]
    
- [[cards/blue-team/threat-intelligence/IP and Domain Threat Intel#Services and Certificates\|Using Shodan, TLS certificate analysis, and pivots to assess exposed services and potential attacker infrastructure.]]
    
- [[cards/blue-team/threat-intelligence/IP and Domain Threat Intel#Reputation Checks and Passive DNS\|Leveraging VirusTotal, Cisco Talos, IP2Proxy, passive DNS, and CT logs to assess IOC history and risk.]]
    
- [[cards/blue-team/threat-intelligence/IP and Domain Threat Intel#Operational Integration\|Guidelines for safe SOC actions — hostname vs IP blocking, expiry, cloud/CDN pitfalls, and legal/provider considerations.]]


## Prerequisite
---

- [[cards/web/How the web works\|How the web works]] - you can read the _Domain Name System_ part here but you can read the whole as this still DOMAIN:IP base.
- _Autonomous System Number_ - a identifier assigned to a network that routes traffic on the Internet (number).
- Autonomous System - A collection of IP address ( a network) run by the same organization that has it's own routing policy.
	
	- Google's Network
	- Cloudflare's Network.
	- Internet Service Provider's Network.
	

## IP Building Blocks
---
The [[cards/web/How the web works\|Domain Name System]] is a mechanism that converts human-firendly names such as google[.]com into IP addresses that machines understand, With the high level importance of DNS for the Internet, this makes a favourite attack vector for the attackers.

Adversaries rapidly register, configure, and abandon domains to stay ahead of defences. Your job as analysts is to turn a raw domain into a contextual artefact: who owns it, what IPs it resolves to, how often it changes, and whether it behaves more like a normal content delivery network (CDN) or a throwaway setup.

**Core DNS Records for Triage**

- **A / AAAA Records**: Map the domain to IPv4 and IPv6 addresses. If you see several A records that hop between different networks, raise suspicion of _rapid rotation_. In practice, copy the A record from [nslookup.io](https://www.nslookup.io) or [dnschecker.org](https://dnschecker.org) and follow with pasting the IP into VirusTotal for a quick read.

	- Normally, if you query DNS multiple times it will resolve into one IP address, **unless** the DNS server you previously queried for is not the best 'path' e.g the original DNS server is near Australia but it was down or maintenance for some reason then it will choose the next nearest alternative.
	- **Suspiciously**, if the domain address maps to multiple different IP address that belongs to **different providers or network**, this suggest that the IPs are changing quickly across unrelated infrastructure therefore tools such as `nslookup` and `dnscheckers` are important.
	- **Attacker perspective**: Malicious actors may point A records to **compromised machines** that act as temporary proxies. This technique, known as **fast flux**, hides the real command-and-control server and makes it harder for defenders to block the malicious infrastructure.

- **NS Records**: Identify the nameservers (authorative nameserver) controlling the domain. Unusual or recently changed NS entries can mark fresh set up. **Note the provider's name** rather than focusing on low level details

- **MX Records**: Define which servers handle email. Attackers may configure MX records to deliver phishing campaigns directly. If the alert relates to web browsing, just record whether MX exists.

- **TXT Records**: Store [[cards/blue-team/phishing/Sender Policy Framework (SPF)\|Sender Policy Framework (SPF)]] and [[cards/blue-team/phishing/DomainKeys Identified Mail (DKIM)\|DomainKeys Identified Mail (DKIM)]] rules or verification tags. Poorly configured or absent SPF can increase risk in mail cases. see [[cards/blue-team/phishing/Phishing\|Phishing Analysis]] notes for more info. 

- **SOA (Start of Authority) record**: It defines the **primary source of truth** for that domain’s DNS: who controls the domain’s DNS and whether it’s stable, new, or being tampered with.

- **TTL (Time To Live)**: Tells resolvers how long to cache answers. Very low TTLs, seconds or minutes, can point to frequent changes, and should be treated as clues.
### Attack Techniques Using DNS
---

- **Fast Flux Hosting**: Adversaries rotate many IPs quickly with short cache times to avoid simple blocks. **Identify a domain that resolves to changing IPs within a short period and across different providers.**

- **CDN Abuse**: If the A record points to a **major CDN ASN** then **IP rotation alone isn’t suspicious**, it’s normal CDN behavior.
	
	- **Reputation** - Is the domain itself flagged as malicious? (Even if it's behind a CDN, it could be phishing or any malicious activity)
	- **Ownership** - Does the domain belong to an organization that makes sense for that CDN usage? (e.g. `paypal.com` on Cloudflare is normal, but `login-paypa1.com` on Cloudflare is suspicious).

- **Typosquatting** such as paypa1[.]com or micros0ft[.]com
- **Internationalised Domain Names**: look-alike domains abuse via Unicodes.

**So you analyst should do:**

- Record A, NS, MX, TXT, SOA, and TTL values in a single page view.
- **Basic Ownership Check:** Use WHOIs to note registrat, creation date, and contract pattern.
- **Assesses patterns:** benign CDN activity or indicates malicious throwaway domain.

## IP Enrichment: Geolocation and ASN
---
An IP address is ambiguous by itself: it could belong to a compromised router, a shared CDN edge, or a cloud service used by thousands of tenants. Without enrichment, we risk blocking legitimate traffic or ignoring real command-and-control server.

### RDAP (Registration Data Access Protocol)

Is the modern replacement for `whois`, It’s used to query authoritative databases run by the **Regional Internet Registries (RIRs)** to find out **who owns an IP address or domain**.

A query would yield:

- **The organization that owns it** (e.g., Google, Cloudflare, an ISP, or a small hosting company).
- **Contact info** (abuse contacts, technical contacts).
- **Registration details** (allocation date, status).
- **Netblock size** (the range of IPs assigned).

![IP and Domain Threat Intel.png|400](/img/user/cards/blue-team/threat-intelligence/images/IP%20and%20Domain%20Threat%20Intel.png)

If our alert points to suspicious traffic, we should pivot to the domain or certificate to narrow the scope.

**Domain investigation:**

- Reputation checks such as VirusTotal, AbuseIPDB, and more.
- **Passive DNS**: see what other IPs the domain has pointed to (e.g, fast flux).
- **whois/rdap**: see registrar, creation date, ownership: new, recently registered domains = higher risk.

**Certificate investigation:**

- **Issuer**: Free certs such as Let's Encrypt are common but very short lived certs or odd issuers (assume that the domain is pretending to be big company Google they never or most likely never going to use Let's Encrypt) can be a red flag.
- **Validity dates**: Recently issued or expiring soon might suggest throwaway infrastructure. (can be viewed in the lock icon -> more info -> view certificate)
- **Certificate Transparency logs**: find other domains using the same certificate fingerprint. (can be found the same way as the above) then go to crt[.]sh and paste the following hash

![IP and Domain Threat Intel-1.png](/img/user/cards/blue-team/threat-intelligence/images/IP%20and%20Domain%20Threat%20Intel-1.png)

### Autonomous Systems and Heuristics
---
Think of the Internet as **lots of different networks stitched together**. Each big network is called an Autonomous System. Every AS is run by one **organization** (e.g., Amazon, Vodafone, Facebook).

- **Hosting ASNs**

    - These belong to hosting companies that sell servers to anyone.
        
    - They have **lots of small IP ranges, rented out to diverse customers**.
        
    - Attackers love these because they can easily rent a VPS and host malware.
        
    - Example: OVH, DigitalOcean.
        
    - Suspicious traffic here -> **likely an attacker’s rented box**.
        
- **Residential ISP ASNs**
    
    - These belong to consumer Internet providers (Vodafone, Comcast, etc.).
        
    - IP ranges are **huge**, covering millions of home users.
        
    - Suspicious traffic here usually means a **compromised customer device/router** — not a malicious hosting service.
        
    - Blocking the whole ASN would cut off millions of innocent users.
        
- **Cloud/CDN ASNs**
    
    - These belong to cloud providers (AWS, Azure, GCP) or CDNs (Cloudflare, Akamai).
        
    - They have **massive global infrastructure**.
        
    - Attackers love cloud ASNs for **temporary C2 servers, phishing sites, or scanners**.
        
    - Blocking the entire ASN would be catastrophic — you’d break half the Internet.
        
    - Instead, analysts scope down to:
        
        - The **specific domain** (FQDN).
            
        - Or a **narrow IP range (CIDR)**.

### Geolocation
---
Geolocation should be treated as a **hint**, not a fact:

1. **Country mismatches**

	- Cloud/CDN providers often **register their IP ranges in one country** but **serve traffic worldwide**.
	- Result: The GeoIP may say "United States" even though the packets came from Australia.

2. **City-level is unreliable**: databases often guess city locations based on where an ISP _registered_ an IP, not where it’s actually used.

Best practice - use two or more sources for IP geolocation then note the discrepancy (if any).

**So you as analyst, should do:**

- Start with **RDAP**.
- **Add ASN Context**: with bgpview[.]io or ipinfo[.]io.
- **Check Geolocation**: Capture country from at least two sources. Record mismatches.
- **Look for rDNS Patterns**: Reverse DNS can hint at hosting type. Do not base decision solely on rDNS.
- **Consult Internal Logs**: Has this IP appeared in the last 30 days? If yes, in what context?
- **Classify Role**: Hosting, residential, CDN, or cloud. Record reasoning.

## Services and Certificates
---
Exposed services provide information on the system's intent and potential blast radius if abused.

**Shodan**

A powerful reconnaissance tool for IP address analysis. It does this by indexing Internet-connected devices and services, it provids detailed information about open ports, running services, and system configurations

![IP and Domain Threat Intel-3.png|450](/img/user/cards/blue-team/threat-intelligence/images/IP%20and%20Domain%20Threat%20Intel-3.png)
From our search:

- **Open Ports**: This is the first fingerprint of exposure.
- **Service Banners**: These provide hints on server types and frameworks used. Additionally, software versions and cookies in RDP/HTTP banners can inform us about operator markers.
### TLS Certificates as Infrastructure Clues
---
[[cards/blue-team/threat-intelligence/IP and Domain Threat Intel#IP Enrichment Geolocation and ASN\|Section discussed here]]:

- **Issuer:** This field provides details on who signed the certificate. For example, Let's Encrypt is a common but neutral vendor. A self-signed certificate may be a sign of a hastily deployed system.
- **Validity Period:** Short-lived certificates of up to 90 days are normal for usage. Analysts must look for bursts of reissued certificates and investigatesuspected phishing infrastructure.
- **Subject Alternative Names:** This provides details on the domains covered by thecertificate.

At crt[.]sh
![IP and Domain Threat Intel-4.png](/img/user/cards/blue-team/threat-intelligence/images/IP%20and%20Domain%20Threat%20Intel-4.png)

**Pivots and Mapping**

Tools like Censys.io allow analysts to pivot by finding siblings with the same certificate (again previously discussed [[cards/blue-team/threat-intelligence/IP and Domain Threat Intel#IP Enrichment Geolocation and ASN\|here]]) - [69.197[.]185[.]26]

**You should do as analyst:**

- **Open Services/Banner:** Identify exposed services and possible misconfigurations.
- **Review TLS certificates:** record issuer, SANs, and validity period
- **Pivot:** Utilise the certificate or banner artefacts to uncover related infrastructure. (needto know more - a real example)
- **Assess blast radius:** 
	- RDP/SSH on residential ASN -> shows a likelihood of a compromised endpoint
	- TLS with many unrelated SANs on CDN ASN -> shared infrastructure, avoid IP block.
	- Self-signed TLS on small ranges -> shows likelihood of attacker panels or proxies.

hxxps://www[.]shodan[.]io/host/85.188.1[.]133

## Reputation Checks and Passive DNS
---
By this stage of enrichment, we have uncovered two major intelligence avenues: “Who owns this IP/domain?” and “What services does it expose?”. The next question is what this IP/domain/indicator has been doing over time. It should also be **considered how dynamic IPs and domains are** given they could be used for phishing for a week then taken down and then the provider re-assigns the IP address to another tenant/user.

Reputation services and passive DNS analysis provide us with a way of adding that value.

**Reputation Services:**

VirusTotal but Cisco Talos Intelligence should be also used which provides frequently updated web and email reputation scores and category labels

**Talos Dashboard:**

By default, the dashboard presents an overview of email traffic across numerous countries, with indicators of whether the emails are legitimate, spam, or malware. Any of these markers will produce more information associated with IP and hostname addresses, daily volume, and type.

![IP and Domain Threat Intel-5.png|450](/img/user/cards/blue-team/threat-intelligence/images/IP%20and%20Domain%20Threat%20Intel-5.png)

**I2PROXY**

IP2Proxy is a tool/database that helps you figure out if an IP address is coming from:

- A VPN
- A proxy server
- A Tor exit node (part of the Tor anonymity network

**PassiveDNS**

Passive DNS adds time context in domain enrichment, providing a historical record of how domains resolved over time. The key signals to look at include:

- **First Seen/Last Seen:** These tell us if a domain is new or long-lived.
- **Number of IPs in Time Window:** A high churn over days would suggest flux or agile hosting.
- **ASN Spread**: If IPs belong to many unrelated ASNs, this should be marked as suspicious, while those limited to one ASN should be marked as stable or belonging to a CDN mapping.

**Beyond using Passive DNS**

- **Certificate Transparency (CT) Logs**: These logs show certificate issuance history. They are useful for detecting sudden bursts of domains registered under phishing themes.
- **Wayback Machine:** Reveals historical website content. A domain that hosted a blog for years but switched to a phishing kit last week is high-risk.

**So you as analyst, should do:**

- **Check VirusTotal:** Record detection ratio, First Seen, Last Seen, and any community notes.
- **Check Cisco Talos:** Record reputation score and category, noting any changes in the last 30 days.
- **Check IP2Proxy:** Flag if VPN/proxy/Tor; adjust severity accordingly.
- **Check Passive DNS:** Record First Seen, Last Seen, number of IPs in the last 7 days, and ASN spread.
- **Check CT Logs:** Note certificate bursts, suspicious SANs.
- **Cross-Reference with Wayback:** Identify content shifts (benign -> phishing).
- **Decision:** Block, monitor, or close, with expiry tied to observed activity.
## Operational Integration
---
Its very important to turn intelligence into safe action. Again IP and Domain is very
dynamic, one day it could be used for phishing and then the next day it is used for
legitimate purposes, additionally blocking legitimate service provider such as (for example sake) Google drive as it was hosting a malicious executable, blocking drive[.]com would be catastrophic, the safe action would be is reporting it to Google or blocking the **HASH of the EXECUTABLE**.

**Safe Integration Patterns:**

1. Prefer hostnames over IP addresses when possible
2. If you must block an IP, make it specific
3. Always set an expiration on blocks cuz bro IP/DOMAINs or any infrastructure gets recycled.
4. Document your reasoning.

**Geofencing Cautions:**

Never, treat only as an enrichment to raise priority. UNLESS the risk decision has been reviewed with the business will FUNCTION NORMALLY after country blocking.

**Cloud and Large Provider Pitfalls:**

Poviders such as Amazon and Microsoft reuse IPs across many customers and services. Only take action at a domain (e.g malicious.hehe) or path level (e.g
malicious[.]hehe/verymalicius.exe), it is also important to consider vendor abuse processes here. For example: Google Drive, it's legitimate but can be abused by hosting malicious files, ask business does company use Google Drive in any way, shape, and form? (Bad business example but you get the idea)

**Legal and Provider Considerations:**

Knowing the provider and country informs whether evidence preservation or rapid
takedown is achievable. Some providers have strong abuse desks and comply quickly with requests. Others are slower or operate under legal frameworks that make urgent actions difficult.



