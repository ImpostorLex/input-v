---
{"dg-publish":true,"permalink":"/cards/active-directory/active-directory-enumeration/","tags":["red-team/ad"]}
---

[[cards/active-directory/active-directory\|active-directory]]
### Introduction
---
During internal penetration test we are often given a Virtual Private Network access to the target network without user credentials this means we need to gather information about the domain: users, groups, computers, and policies that will serve us our attack paths.

The topology of the environment:

![Active Directory Enumeration.png](/img/user/cards/active-directory/images/Active%20Directory%20Enumeration.png)
#### Key Topics
---

- [[#Host Discovery|Host discovery using fping and nmap to identify live hosts and Domain Controllers]]
- [[#Listing SMB Shares|Enumerating SMB shares anonymously using smbclient and smbmap]]
- [[#LDAP Enumeration (Anonymous Bind)|Anonymous LDAP queries to enumerate users and domain data]]
- [[#Enum4linux-ng|Enumerating AD info via SMB and RPC with enum4linux-ng]]
- [[#RPC Enumeration (Null Sessions)|RPC enumeration using null sessions with rpcclient]]
- [[#RID Cycling|Enumerating users via RID cycling in RPC]]
- [[#Username Enumeration With Kerbrute|Validating usernames against Kerberos using kerbrute]]
- [[#Password Spraying|Performing password spray attacks after discovering policy]]

### Prerequisites
---
- [[cards/active-directory/active-directory\|active-directory]]
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

![Active Directory Enumeration-1.png](/img/user/cards/active-directory/images/Active%20Directory%20Enumeration-1.png)

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

Assuming the Domain Controller IP address is not shown in [[cards/active-directory/Active Directory Enumeration#Introduction\|introduction page]], the indicators are the mentioned ports are open and the **smb-os-discovery** shows the following:

![Active Directory Enumeration-2.png](/img/user/cards/active-directory/images/Active%20Directory%20Enumeration-2.png)

The domain is shown in the **ldap** line 'Domain: tryhackme.loc' and (not shown in the screenshot) below the **'Computer name: DC'** the **Domain name** field holds the domain name of the AD environment.

Comparing to a normal endpoint:

![Active Directory Enumeration-3.png](/img/user/cards/active-directory/images/Active%20Directory%20Enumeration-3.png)

Alternatively, if we are not familiar with the environment running a full port scan is great idea as well:

```bash
nmap -sS -p- -T3 -iL hosts.txt -oN full_port_scan.txt
```

- `-sS`: TCP SYN scan, which is stealthier than a full connect scan
- `-p-`: Scans all 65,535 TCP ports.
- `-T3`: Sets the timing template to "normal" to balance speed and stealth.
- `-iL hosts.txt`: Inputs the list of live hosts from the previous nmap command.
- `-oN full_port_scan.txt`: Outputs the results to a file.
## Network Enumeration With SMB
---
The technique assumes we compromised a Linux-based box, `nmap` will be used to discover listening ports and identify services, access/grab shares content using `smbclient` and `smbmap`.

Focusing on MS Windows and Active Directory-related ports, so we should focus on the following ports:

- TCP 88 ([[cards/active-directory/Kerberos\|Kerberos]]): It is used for authentication in Active Directory - possible attacks are [[Pass-the-Ticket\|Pass-the-Ticket]] and [[Kerberoasting\|Kerberoasting]].
- TCP 135 ([[cards/networking/Remote Procedure Call\|RPC]] Endpoint Manager): It might be leveraged to identify services for lateral movement or remote code execution via DCOM.
- TCP 139 (NetBIOS Session Service): This port is used for file sharing in older Windows systems. It can be abused for null sessions and information gathering.
- TCP 389 ([[cards/active-directory/Lightweight Directory Access Protocol\|LDAP]]): It is in plaintext and can be a prime target for enumerating AD Objects, users, and policies. 
- TCP 445 (SMB): Critical for file sharing and remote admin; abused for exploits like EternalBlue, SMB relay attacks, and credential theft.
- TCP 636 (LDAPS): This port is used by Secure LDAP. Although it is encrypted, it can still expose AD structure if misconfigured and can be abused via certificate-based attacks like AD CS exploitation.

```bash
nmap -p 88,135,139,389,445,636 -sV -sC TARGET_IP 
```

- `-sV` detect version
- `-sC` run default script included in nmap.

![Active Directory Enumeration-4.png](/img/user/cards/active-directory/images/Active%20Directory%20Enumeration-4.png)

> [!NOTE]+ What's the difference between the two commands?
> 
> This scan is quick, targeted scan to see if a host might be a Domain Controller or important AD server. Best when you know or suspect the DC IP.
### Listing SMB Shares
---
In this point - we don't have any valid credentials yet, we should focus on trying anonymous connection also known as _null session_ to get access:

`smbclient` is similar to a FTP client:

```bash
smbclient -L //10.211.11.10 -N
```

- `-L` list shares
- `-N` no password.

![Active Directory Enumeration-5.png](/img/user/cards/active-directory/images/Active%20Directory%20Enumeration-5.png)

`smbmap` is a reconnaissance tool that enumerates a shares across a host, display read and write permission for each share:

```bash
smbmap.py -H 10.211.11.10
```

Tip: `which smbmap` to locate location.

Did not work in kali box but it should display shares and their permissions - no output shown.
#### Accessing SMB shares
---
The shares that we should focus on should have a `READ` access among their permssion, this should be identifiable using the previous command:

```C
smbclient //10.211.11.10/SharedFiles -N
```

- `ls` to display files in the share.
- `get file_name` to download the file in question.
- `--user=USER --password=PASSWORD` if you have credentials
- `-w` to specify domain for domain accounts.

Why this exist?

System administrators may have enabled it for legacy systems to work. Maybe an old printer needs to read a file, or an old scanner needs to write a file. Accessing a particular folder from any computer on the network is convenient
## Domain Enumeration
---
After understanding our target's network - it's time to enumerate users to help us identify valid accounts and potential targets through various unauthenticated methods relying on misconfigurations.
### LDAP Enumeration (Anonymous Bind)
---
Read what is [[cards/active-directory/Lightweight Directory Access Protocol\|Lightweight Directory Access Protocol]] here before moving. some LDAP servers allow anonymous users to perform read-only queries. This can expose user accounts and other directory information.

Testing if anonymous bind is enabled with:

```bash
ldapsearch -x -H ldap://10.211.11.10 -s base
```

- `-x`: Simple authentication, in our case, anonymous authentication.
- `-H`: Specifies the LDAP server.
- `-s`: Limits the query only to the base object and does not search subtrees or children.

Anonymous bind enabled when there is a lot of output:

![Active Directory Enumeration-6.png](/img/user/cards/active-directory/images/Active%20Directory%20Enumeration-6.png)
Then query user information with this command:

```
ldapsearch -x -H ldap://10.211.11.10 -b "dc=tryhackme,dc=loc" "(objectClass=person)"
```

Looking at one specific person, we can see that **rduke** is part of the domain users indicated by **primaryGroupID**:

![Active Directory Enumeration-7.png](/img/user/cards/active-directory/images/Active%20Directory%20Enumeration-7.png)

#### Enum4linux-ng
---
This tool does the same also but this time it utilizes SMB and RPC protocols together to gather informations

```bash
enum4linux-ng -A 10.211.11.10 -oA results.txt
```

- `-A`: Performs all available enumeration functions (users, groups, shares, password policy, RID cycling, OS information and NetBIOS information).
- `-oA`: Writes output to YAML and JSON files.
### RPC Enumeration (Null Sessions)
---
Read this [[cards/networking/Remote Procedure Call\|Remote Procedure Call]] - RPC services can be accessed over the SMB protocol. When SMB is configured to allow null sessions that do not require authentication, an unauthenticated user can connect to the IPC$ share and enumerate users, groups, shares, and other sensitive information from the system or domain.

Verify null session access with:

```bash
rpcclient -U "" 10.211.11.10 -N
```

- `-U`: Used to specify the username, in our case, we are using an empty string for anonymous login.
- `-N`: Tells RPC not to prompt us for a password.

Then if successful run the command `enumdomusers` to view available users, additionally `help` command has a lot of commands available with the right permissions we can enumerate the domain with RPC only.
#### RID Cycling
---
_Relative Identifiers_ in Active Directory are used to assign unique identifiers to users and group objects, these RIDs are components of the Security Identifier (SID) which uniquely identifies each object within a domain.

Here are well known RIDs:

- **500** is the Administrator account
- **501** is the Guest account and **512-514** are for the following groups: Domain Admins, Domain users and Domain guests. 
- User accounts typically start from RID **1000** onwards.

Previous command `enumdomusers` already provides it's RID and alternatively if `enumdomusers` is restricted:

```bash
for i in $(seq 500 2000); do echo "queryuser $i" |rpcclient -U "" -N 10.211.11.10 2>/dev/null | grep -i "User Name"; done
```

#### Username Enumeration With Kerbrute
---
Read [[cards/active-directory/Kerberos\|Kerberos]] first here.

Tools like **enum4linux-ng** or **rpcclient** may return _some_ usernames, but they could be:

- Disabled accounts
- Non-domain accounts
- Fake honeypot users
- Or even false positives

Here is a modified version of the previous command to save username in a `.txt` file:

```bash
for i in $(seq 500 2000); do echo "queryuser $i" |rpcclient -U "" -N 10.211.11.10 2>/dev/null |grep -i "User Name" | cut -d: -f2 | xargs -I{} echo {} | sed 's/^ *//' >> users.txt; done
```

Then we can use `users.txt` to determine valid accounts using the below command:

```C
./kerbrute userenum --dc 10.211.11.10 -d tryhackme.loc users.txt
```

If other tools aren't available but `kerbrute` is, we can download a wordlist like [this one](https://github.com/danielmiessler/SecLists/blob/master/Usernames/Names/names.txt) to discover user accounts.
## Password Spraying
---
Now that we have a list of username, it's time to test them but first we need to determine it's password policy including: minimum password length, complexity, and the number of failed attempts that will lock out an attack.

_Password Spraying_ is testing one or small set of password against multiple accounts to avoid account lockouts, these happens for a few reasons:

- Require frequent password changes, leading users to pick predictable patterns
- Don't enforce their policies well.
- Reuse common passwords across multiple accounts.

**rpcclient**

Query password policy with `rpcclient`:

```bash
rpcclient -U "" 10.211.11.10 -N
```

Then hit `getdompwinfo`:

![Active Directory Enumeration-8.png](/img/user/cards/active-directory/images/Active%20Directory%20Enumeration-8.png)

`password_properties: 0x00000001`

This is a **bitmask** representing password policy flags. The value `0x00000001` corresponds to: `DOMAIN_PASSWORD_COMPLEX` meaning complexity password is enabled.

This means that at least three of the following four conditions need to be respected for a password to be created:

1. Uppercase letters
2. Lowercase letters
3. Digits
4. Special characters

Additionally, [in this link](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994562(v=ws.11)) as part of the password complexity policy - Microsoft does not allow including user's account name as part of the password.

**crackmapexec**

It supports various network protocols, such as SMB, LDAP, RDP, and SSH. If anonymous access is permitted, we can retrieve the password policy without credentials with the following command:

```bash
smb 10.211.11.10 --pass-pol
```

#### Performing the attack
---
It requires a password list most of them are available or built-in kali linux:

```bash
smb 10.211.11.20 -u users.txt -p passwords.txt
```

Then watch the magic.


