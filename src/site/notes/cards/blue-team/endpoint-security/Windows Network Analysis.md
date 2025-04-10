---
{"dg-publish":true,"permalink":"/cards/blue-team/endpoint-security/windows-network-analysis/"}
---

[[Network Security MOC\|Network Security MOC]]
### Introduction
---
A great way to start an investigation, as everything starts with a network connection.

_Context: Windows Machine is connected to Ubuntu Machine via Reverse Shell created using msfvenom_.

- `net view \\127.0.0.1` or `net share`
	- View shares that the attacker may have setup to exfiltrate data or store tools to be used for compromising other machines.
	- `net share` shows local shares in our system, `net view` otherwise.

Demonstration (at meterpreter):

```bash
mkdir exfill
net share Exfil=C:\Users\lianne\Downloads\exfill
```

Output:

![Windows Network Analysis.png](/img/user/cards/blue-team/endpoint-security/images/Windows%20Network%20Analysis.png)
- `net session`
	- Shows which system are connected to our system, as well as shows the attacker mounted a share on our system.
- `net use`
	- Shows which system are currently communicating with our system. (OUTBOUND)

Output:

![Windows Network Analysis-1.png](/img/user/cards/blue-team/endpoint-security/images/Windows%20Network%20Analysis-1.png)
**Shows established and listening IP address and ports**

```bash
netstat -anob
```

Output:

![Windows Network Analysis-2.png](/img/user/cards/blue-team/endpoint-security/images/Windows%20Network%20Analysis-2.png)
- `tcpview` is basically netstat but GUI with more.
	- Can pause and refresh manually or automatic refresh.
	- Toggle IPv4 or 6.
	- Green highlight - if the state of the connection has changed or recently added while red the connection is terminated.
	- Extract current list in a csv format.
	- Shows **metadata** of any process by right clicking.
	- can perform `whois`

**Organize and refine notes after each session including adding meta-tags**

