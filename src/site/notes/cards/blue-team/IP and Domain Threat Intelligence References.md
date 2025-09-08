---
{"dg-publish":true,"permalink":"/cards/blue-team/ip-and-domain-threat-intelligence-references/","tags":["blue-team"]}
---

~ [[cards/blue-team/threat-intelligence/IP and Domain Threat Intel\|IP and Domain Threat Intel]]
### Introduction
---
It's important you know the [[cards/blue-team/threat-intelligence/IP and Domain Threat Intel\|WHY]] first before proceeding:

## DNS Building Blocks
---

- `nslookup <domain>` or [dnschecker.org](https://dnschecker.org) → Get **A, NS, MX, TXT, SOA, TTL**.
- [whois.domaintools.com](https://whois.domaintools.com) or `whois <domain>` → Registrar, creation date, ownership.
- Compare IP churn: repeat `nslookup` — note if IPs belong to **different ASNs** → possible fast flux.
- Typosquatting check: visually inspect domain (`paypal.com` vs `paypa1.com`).
- For Unicode lookalikes → [punycode converter](https://www.punycoder.com/).
## IP Enrichment (RDAP, ASN, Geo)
---
- RDAP: [rdap.org](https://rdap.org) or `whois <ip>` → org, allocation date, netblock size.
- ASN context: [bgpview.io](https://bgpview.io) or [ipinfo.io](https://ipinfo.io).
- Geolocation: [iplocation.net](https://iplocation.net) (use 2+ sources) → note mismatches.
- rDNS: `dig -x <ip>` → can hint at hosting type (cloud, ISP, residential).
- Logs: Search internal SIEM/Proxy for last 30 days activity.
- Classify: Hosting, residential ISP, CDN, or cloud.

## Services & Certificates
---
- Shodan: `https://www.shodan.io/host/<IP>` → open ports, banners.
- Censys: [search.censys.io](https://search.censys.io) → certificates, banners, related infra.
- TLS certificate check: Browser padlock → “View Certificate.”
    - Issuer → Let's Encrypt/self-signed/suspicious.
    - SANs → multiple unrelated = shared infra.
    - Validity period → very short = throwaway.
- Pivot: Use certificate fingerprint in [crt.sh](https://crt.sh).
## Reputation & Historical Context
---
- VirusTotal: [virustotal.com](https://virustotal.com) → detection ratio, first/last seen.
- Cisco Talos: [talosintelligence.com](https://talosintelligence.com) → web/email reputation.
- IP2Proxy: [ip2proxy.com](https://www.ip2proxy.com/) → VPN/proxy/Tor exit.
- Passive DNS: [securitytrails.com](https://securitytrails.com) or [dnsdb.info](https://dnsdb.info) → first/last seen, churn, ASN spread.
- CT logs: [crt.sh](https://crt.sh) → certificate issuance history.
- Wayback Machine: [web.archive.org](https://web.archive.org) → historical content.

## Operational Integration (Actions)
---
- Prefer **hostname blocks** > IP blocks.
- If IP block needed → restrict to smallest CIDR.
- Always **set expiry** (7–30 days typical).
- Log reasoning: source, evidence, and decision.
- Cloud/CDN infra → block by **domain/path**, not ASN.
- For abuse handling:
    - Google → [support.google.com/legal](https://support.google.com/legal)  
    - Microsoft → [abuse reports](https://www.microsoft.com/en-us/msrc/abuse)  
    - AWS → abuse@amazonaws.com  


