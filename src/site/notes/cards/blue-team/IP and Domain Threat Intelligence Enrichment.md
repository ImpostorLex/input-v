---
{"dg-publish":true,"permalink":"/cards/blue-team/ip-and-domain-threat-intelligence-enrichment/","tags":["blue-team"]}
---

[[cards/blue-team/threat-intelligence/IP and Domain Threat Intel\|IP and Domain Threat Intel]]
### Introduction
---
It's important you know the [[cards/blue-team/threat-intelligence/IP and Domain Threat Intel\|WHY]] first before proceeding:

## DNS Building Blocks
---

- **Get core records**
	- Command / site: `nslookup <domain>` or `dig +noall +answer <domain>`
	- Web: https://dnschecker.org
	-  What to note: A/AAAA, NS, MX, TXT, SOA, TTL values and any very low TTLs.

- **Registrar and age**
	- Command / site: `whois <domain>` or https://whois.domaintools.com
	- What to note: registrar, creation date, expiration, registrant organization, privacy service.

- **IP churn / fast-flux check**
	- Repeat DNS queries over a few minutes; use https://dnschecker.org to view global A record distribution.
	- What to note: multiple unrelated ASNs, frequent changes, low TTLs.

- **Typosquatting and Unicode**

	- Visual check for character swaps and lookalikes; convert to punycode if needed.
	- Tool: https://www.punycoder.com
	- What to note: subtle character differences or use of non-Latin characters.

## IP Enrichment (RDAP, ASN, Geo)
---

- **RDAP / ownership**
	- Site / command: https://rdap.org or `whois <ip>`
	- What to note: organization name, abuse contact, allocation date, netblock size.

- **ASN and vendor classification**
	- Sites: https://bgpview.io, https://ipinfo.io
	- What to note: ASN owner, whether ASN is cloud/hosting/CDN/residential, common netblocks.

- **Geolocation**
	- Sites: https://iplocation.net, https://ipinfo.io
	- What to note: country from at least two sources, note discrepancies and treat city-level as low confidence.

- **Reverse DNS**
	- Command: `dig -x <ip>` or `host <ip>`
	- What to note: rDNS pattern indicating hosting provider or consumer ISP.

- **Internal context**

	- Search SIEM / proxy logs for occurrences in the last 30 days.
	- What to note: frequency, user agents, destination URLs, timestamps.

## Services and Certificates
---

- **Open ports and banners**
	- Sites: https://www.shodan.io/host/<IP>, https://search.censys.io
	- What to note: open ports, service banners, software versions, unusual ports like admin panels.

- **TLS certificate facts**
	- Inspect in browser padlock or use crt.sh and Censys.
	- What to note: issuer, SANs, validity dates, certificate fingerprint.

- **Certificate pivots**
	- Site: https://crt.sh (search fingerprint or SAN)
	- What to note: sibling domains using same cert, sudden bursts of certificates.

- **Blast-radius assessment**
	- Rule of thumb: RDP/SSH on residential IPs suggests compromise; many unrelated SANs on cert suggests shared infra; self-signed cert on small netblock suggests attacker panel.
## Reputation and Historical Context
---
- **VirusTotal**
	- Site: https://www.virustotal.com
	- What to note: detection ratio, first seen, last seen, community comments, associated files/URLs.

- **Cisco Talos and other reputations**
	- Site: https://talosintelligence.com
	- What to note: category, recent changes in score, email/web reputation.
- **Proxy/VPN/Tor checks**
	- Site: https://www.ip2proxy.com or https://ipinfo.io
	- What to note: flagged as proxy, VPN, or Tor exit.

- **Passive DNS and churn**
	- Sites: https://securitytrails.com, https://dnsdb.info (paid), https://viewdns.info
	- What to note: first seen, last seen, number of distinct IPs, ASN spread in window.

- **Certificate Transparency and Wayback**
	- Sites: https://crt.sh and https://web.archive.org
	- What to note: certificate issuance bursts (or multiple certs different contexts), historical content changes (clean site that recently became phishing).

## Quick Action Guidance (operational)
---

- Prefer blocking FQDN or URL path over IP where possible.
- If blocking IP, restrict to minimal CIDR and set an explicit expiry date (suggestion: 7–30 days depending on evidence).
- Document evidence with screenshots/log snippets, source/time, and analyst rationale.
- For cloud/CDN-hosted infra, submit abuse/takedown to provider rather than broad IP blocks.
  - Examples: Google abuse form, AWS abuse@amazonaws.com, Microsoft security/abuse pages.
- Preserve artifacts if escalation is needed: save HTTP responses, certificate fingerprints, pcap or SIEM logs.

## Fast command cheat-sheet (copy-paste)
---

- DNS: `nslookup example.com`  or `dig +noall +answer example.com`
- WHOIS/ RDAP: `whois example.com`  or `curl https://rdap.org/domain/example.com`
- Reverse DNS: `dig -x 1.2.3.4`
- Shodan lookup: open `https://www.shodan.io/host/1.2.3.4`
- Cert pivot: open `https://crt.sh/?q=<cert_fingerprint_or_domain>`
- Passive DNS (SecurityTrails): open `https://securitytrails.com/domain/example.com/history/a`
- VirusTotal: open `https://www.virustotal.com/gui/domain/example.com` or `https://www.virustotal.com/gui/ip-address/1.2.3.4`

