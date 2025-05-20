---
{"dg-publish":true,"permalink":"/active-directory-enumeration/"}
---

[[map-of-contents/active-directory\|active-directory]]
### Introduction
---
During internal penetration test we are often given a Virtual Private Network access to the target network without user credentials this means we need to gather information about the domain: users, groups, computers, and policies that will serve us our attack paths.


The topology of the environment:

![Active Directory Enumeration.png](/img/user/Active%20Directory%20Enumeration.png)
### Prerequisites
---
- [[map-of-contents/active-directory\|active-directory]]
- Linux fundamentals
- Computer networking.
## Mapping the Network
---
The goal in this stage is to identify the structure of the Active Directory environment incuding determining it's hosts and services, imagine the following subnet: `10.211.11.0/24`.

In most pentesting scope, the subnet is already included - we just need to know all live hosts.
### Host Discovery
---

**fping**

It's **ping** but better, it uses Internet Control Message Protocol (ICMP) requests to determine if the host is live or not but this time it allows us to specify any number of targets including **subnet**.

```bash
fping -agq 10.211.11.0/24 > hosts.txt
```

- `-a`: shows systems that are alive.
- `-g`: generates a target list from a supplied IP netmask.
- `-q`: quiet mode, doesn't show per-probe results or ICMP error messages.

![Active Directory Enumeration-1.png](/img/user/Active%20Directory%20Enumeration-1.png)

 **nmap**

```bash
nmap -sn 10.211.11.0/24
```

- `-sn`: Ping scan to determine which hosts are up without port scanning.


Then once we know what host are live - we can save them in a `.txt` file such as `hosts.txt` and we can identify critical AD-related services that are being used and can be exploited:

| Port | Protocol               | What it Means                                 |
| ---- | ---------------------- | --------------------------------------------- |
| 88   | Kerberos               | Potential for Kerberos-based enumeration      |
| 135  | MS-RPC                 | Potential for RPC enumeration (null sessions) |
| 139  | SMB/NetBIOS            | Legacy SMB access                             |
| 389  | LDAP                   | LDAP queries to AD                            |
| 445  | SMB                    | Modern SMB access, critical for enumeration   |
| 464  | [[cards/active-directory/Kerberos\|Kerberos]] (kpasswd) | Password-related Kerberos service             |

Running a version scan against the said ports is one way to determine the domain controller (DC) as `nmap` will show 'Window Server' banner for each services and sometimes the domain name as well:

```bash
nmap -p 88,135,139,389,445 -sV -sC -iL hosts.txt
```

- `-sV`: This enables version detection. Nmap will try to determine the version of the services running on the open ports.
- `-sC`: Runs Nmap Scripting Engine (NSE) scripts in the default category.
- `-iL`: This tells Nmap to read the list of target hosts from the file `hosts.txt`. Each line in this file should contain a single IP address or hostname.

Assuming the Domain Controller IP address is not shown in [[Active Directory Enumeration#Introduction\|introduction page]], the indicators are the mentioned ports are open and the **smb-os-discovery** shows the following:

![Active Directory Enumeration-2.png](/img/user/Active%20Directory%20Enumeration-2.png)

The domain is shown in the **ldap** line 'Domain: tryhackme.loc' and (not shown in the screenshot) below the **'Computer name: DC'** the **Domain name** field holds the domain name of the AD environment.

Comparing to a normal endpoint:

![Active Directory Enumeration-3.png](/img/user/Active%20Directory%20Enumeration-3.png)

Alternatively, if we are not familiar with the environment running a full port scan is great idea as well:

```bash
nmap -sS -p- -T3 -iL hosts.txt -oN full_port_scan.txt
```

- `-sS`: TCP SYN scan, which is stealthier than a full connect scan
- `-p-`: Scans all 65,535 TCP ports.
- `-T3`: Sets the timing template to "normal" to balance speed and stealth.
- `-iL hosts.txt`: Inputs the list of live hosts from the previous nmap command.
- `-oN full_port_scan.txt`: Outputs the results to a file.

