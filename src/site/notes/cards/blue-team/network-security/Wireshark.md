---
{"dg-publish":true,"permalink":"/cards/blue-team/network-security/wireshark/"}
---

[[map-of-contents/blue-team\|blue-team]]

- `http` 
	- `http.request.uri contains mal.exe`.

## Statistics
---
**The best way to start an analysis**.

**Note**: it can applies statistics to current display filter so **BECAREFUL**.

- **Capture File Properties**
	- Include timestamp of first and last sent packet.
- **Resolved Address**
- **Protocol Hierarchy** BEST WAY
	- You can easily determine if there is a file sent or retrieved in HTTP.
- **Conversations**
	- It has a powerful appy/prepare as filter function.
- **Endpoints**
	- See all endpoints and total packets or bytes.
- **HTTP -> Request**
	- Shows a chart of all unique domain and url requested.

## Methodology
---
**Note:** make sure timestamp is consistent, ideally UTC.

1. Starting with **statistics**
	- Conversations - sort by packets sent or size.
	- Protocol Hierarchy - identify low hanging fruits such as http, ftp, and smb. 
2. **Make it make sense** - identified a malware or APT? **look up reports of Indicators of Compromise (IOC)** and match it with available evidence.
	- Lateral movement? check arp broadcast then ping then use the `ip.addr` of the pinged address to see some port scans.
3. **Research every unknown protocol** there is a reason why it was captured because it was being used.

Example here: [[cards/blue-team/network-security/challenges/SOC 101 - Wireshark Challenge 1\|SOC 101 - Wireshark Challenge 1]] 

