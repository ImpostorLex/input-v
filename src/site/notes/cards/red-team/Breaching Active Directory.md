---
{"dg-publish":true,"permalink":"/cards/red-team/breaching-active-directory/","tags":["red-team/ad"]}
---

~ [[atlas/red-team\|red-team]]
### Introduction 
---
The most used identity and access management, almost every organization or at least the global fortune top 1000 companies uses Active Directory.

**Before exploitation of AD misconfigurations:**
The most important step is gaining an initial set of valid AD credentials, in this phase you should not care or focus on the permission associated with the account because your goal is to have a way to authenticate to AD, this will allow you to do further enumeration on AD itself.
#### Key Topics
---

- [[cards/red-team/Breaching Active Directory#Prerequisites\|Fundamental setup and environment requirements such as Windows and Active Directory basics, plus DNS client configuration for targeting the domain.]]
    
- [[cards/red-team/Breaching Active Directory#OSINT and Phishing\|Collect public credentials and contextual intel (email, GitHub leaks) and run phishing to harvest logins or deliver RATs for initial access.]]
    
- [[cards/red-team/Breaching Active Directory#NTLM Authenticated Services\|Test harvested credentials against exposed services using safe spraying techniques (password spraying, custom scripts or hydra) rather than brute force to avoid lockouts.]]
    
- [[cards/red-team/Breaching Active Directory#LDAP Bind Credentials\|Abuse service LDAP binds and perform LDAP pass-back by hosting a rogue LDAP endpoint with slapd to capture cleartext or hash material from misconfigured devices.]]
    
- [[cards/red-team/Breaching Active Directory#Authentication Relays\|Intercept and capture NetNTLM via LLMNR/NBT-NS/WPAD poisoning with Responder, crack hashes with hashcat, or perform NTLM relays when SMB signing is not enforced.]]
    
- [[cards/red-team/Breaching Active Directory#Microsoft Deployment Toolkit\|Enumerate and retrieve PXE/MDT artifacts via nslookup, TFTP, and PowerPXE to extract BCD/WIM files and recover embedded credentials or implant images.]]
    
- [[cards/red-team/Breaching Active Directory#Mitigations\|Defensive countermeasures: reduce internet exposure of AD services, enforce SMB signing and least-privilege, deploy NAC, and strengthen user training to reduce credential theft.]]
## Prerequisites

- [[atlas/windows\|windows]]
- [[cards/active-directory/Active Directory\|Active Directory]]

Configure your DNS:

```C
sed -i '1s|^|nameserver $THMDCIP\n|' /etc/resolv-dnsmasq
```

Replace `$THMDCIP` with the actual IP of the active directory and the topology:

![Breaching Active Directory-12.png](/img/user/cards/red-team/images/Breaching%20Active%20Directory-12.png)



## OSINT and Phishing
---

**OSINT:**

Any public information available that can be used to authenticate such as email, hardcoded credentials in Github, and [[cards/red-team/Reconnaissance - Red Team\|more]].

However maybe unreliable as they can be outdated and credentials might not be used for authentication.

**Phishing:**

You can harvest credential through a fake web login page or have the user install a Remote Access Trojan (RAT) gives you instantly users context. more [[cards/red-team/Phishing\|here]]
## NTLM Authenticated Services
---
Read first: [[cards/active-directory/New Technology LAN Manager (NTLM)\|New Technology LAN Manager (NTLM)]] 

As mentioned in the link above exposed services are a great way to test credentials discoverd using other means. Since most AD environments have account lockout configured, full brute-force attacks is not recommended.

**Password Spraying:**

Instead of trying multiple different passwords, which may trigger account lockout mechnism, you choose and use one password and attempt to authenticate with all the usernames you have acquired.

Custom script or using hydra is a great choice but for much finer control custom script is recommended.

```C
python3 ntlm_passwordspray.py -u usernames.txt -f za.tryhackme.com -p Changeme123 -a http://ntlmauth.za.tryhackme.com/
```

Output:

![cards/red-team/images/Breaching Active Directory.png](/img/user/cards/red-team/images/Breaching%20Active%20Directory.png)

The usual process should be:

1. OSINT.
2. Find exposed services.
3. Find valid credentials.
4. Test.
## LDAP Bind Credentials
---
Read first: [[cards/active-directory/Lightweight Directory Access Protocol\|Lightweight Directory Access Protocol]]

If a web app or service on the Internet uses **LDAP** to check usernames/passwords, that app usually **has its own AD account or can see user passwords**. If an attacker compromises that app, they can steal those credentials and then use them to talk to Active Directory directly.

![Breaching Active Directory-1.png](/img/user/cards/red-team/images/Breaching%20Active%20Directory-1.png)
### Pass-back attacks
---
If you can log into a device's admin UI or otherwise edit it's LDAP settings, you can **point it at a server you control** and make the device **try to authenticate to you**, forcing it to "pass back" credentials that you can capture.

- Printers, scanners, cameras, NAS, routers, etc. often have an admin web UI where you set “LDAP server”, “Bind Device Name / username” and “Password”.

- That password is used by the device to talk to Active Directory (to look up users, validate credentials, etc.).

- Many devices keep `admin:admin` or `admin:password` out of the box, and admins forget to change them. That makes it easy to log in to the device and **change some settings.**

	- _I have admin creds tho?_ often times admin creds and ldap creds are different and admin creds is only used to access the control panel of the device.

- If you change the LDAP server IP in the device config to an attacker-controlled server, the device will try to connect and authenticate — and send whatever credentials it has stored or the authentication handshake it normally performs

	- Many devices uses unencrypted LDAP by default, you can see plaintext credentials or authentication hashes for later cracking.

- **Goal:** The LDAP password is typically stored in the device config and shown as `******` in the UI, the key the device uses to talk to AD (usually a service account or stored password).
	
	- Resulting into: **the ability to query AD or impersonate the device**.

#### Performing an LDAP Pass-Back
---
There is a printer UI exposed on the network:

![Breaching Active Directory-2.png|450](/img/user/cards/red-team/images/Breaching%20Active%20Directory-2.png)

Inspecting element, you can also verify that the printer website was at least secure enough to mask the LDAP password back to the browser:

![Breaching Active Directory-3.png](/img/user/cards/red-team/images/Breaching%20Active%20Directory-3.png)

This is basically the **"control panel"** for demonstration purposes it skipped the logging into the admin device UI.

**Pointing device to our attack server by hosting a rogue LDAP server:**

```C
sudo apt-get update && sudo apt-get -y install slapd ldap-utils && sudo systemctl enable slapd
```

Then:

```C
sudo dpkg-reconfigure -p low slapd
```

- Press "No" when requested if you want to skip server configuration
- **DNS Domain Name:** the target domain name in this example `za.tryhackme.com`
- **Organization Name:** same as above.
- Provide a password.
- **Ensure the database is not removed when purged:** "No"
- **Move old database:** "Yes"

**Downgrading our LDAP server to accept PLAIN and LOGIN authentication:**

Create a file called: `olcSaslSecProps.ldif`

```C
#olcSaslSecProps.ldif
dn: cn=config
replace: olcSaslSecProps
olcSaslSecProps: noanonymous,minssf=0,passcred
```

- **olcSaslSecProps:** Specifies the SASL security properties
- **noanonymous:** Disables mechanisms that support anonymous login
- **minssf:** Specifies the minimum acceptable security strength with 0, meaning no protection.

**Use the ldif file and patch our server:**

```C
sudo ldapmodify -Y EXTERNAL -H ldapi:// -f ./olcSaslSecProps.ldif && sudo service slapd restart
```

Verify your rogue LDAP server's configuration has been applied using the following command:

```C
ldapsearch -H ldap:// -x -LLL -s base -b "" supportedSASLMechanisms
```

Your rogue LDAP server has now been configured. When we click the "Test Settings" at [hxxp://printer.za.tryhackme[.]com/settings.aspx](http://printer.za.tryhackme.com/settings.aspx), the authentication will occur in clear text and we can capture this by using TCPdump:

```C
sudo tcpdump -SX -i breachad tcp port 389
```

**Don't forget to change the Server IP to your Server Attack IP**.

![Breaching Active Directory-4.png](/img/user/cards/red-team/images/Breaching%20Active%20Directory-4.png)
## Authentication Relays
---
Earlier, the NTLM authentication used on a web is demonstrated, this time you will look at this authentication mechanism in the network's perspective. **The NTLM network challenge/response used when a Windows client authenticates to a file/share (SMB)**. It's what the network sees when Kerberos isn't used.

Read [[cards/active-directory/Server Message Block (SMB)\|Server Message Block (SMB)]]

Though some of these attacks are patched in the newer versions of SMB protocol, in some organization it still exists as the newest version is not supported by legacy systems, there are two ways for this exploit:

- Since the NTLM Challenges can be intercepted, we can use offline cracking techniques to recover the password associated with the NTLM Challenge. However, much slower than cracking the NTLM hashes directly.

- You can use your rogue device to stage a man in the middle attack, relaying the SMB authentication between the client and server, which will provide us with an active authenticated session and access to the target server. **Instead of cracking we have the client authenticate to use then forwards the authentication to the actual server providing as an active authenticated session.**

### LLMNR, NBT-NS, and WPAD
---
You will use `Responder` to attempt to intercept the NetNTLM challenge and crack it. There are usually  a lot of these challenges flying around on the network. 

- If DNS records are stale (point to removed hosts) or users try to access `host.local` or mistyped names, more broadcast lookups are sent — more opportunities for poisoning.

- Hosts often try LLMNR/NBT-NS (NBT-NS is a much better version of LLMNR) when DNS lookup fails, allow hosts itselves to perform their own local DNS resolution for all hosts on the same local network or times out. 
	
	- These protocol exists to avoid overburdening DNS servers. 
	- Since these protocols rely on requests broadcasted on the local network, our rogue device would also receive these requests. Usually, these requests would simply be dropped since they were not meant for our host.

- On a real LAN, `Responder` will attempt to poison any  Link-Local Multicast Name Resolution (LLMNR),  NetBIOS Name Service (NBT-NS), and Web Proxy Auto-Discovery (WPAD) requests that are detected.

### Intercepting NetNTLM Challenge
---
It's important to **note** about `responder` is that it always tries to win the race condition by poisoning the connections to ensure that you intercept the connections. These means two things:

- The tool is usually limited to local network. 
- The tool is disruptive and resulting into possible detection, by poisoning authentication request, normal network authentication attempts would fail, meaning users and services would not connect to the hosts and shares they intend to.

```C
sudo responder -I breachad
```

If you were using your rogue device, you would probably run Responder for quite some time, capturing several responses. Once you have a couple, you can start to perform some offline cracking of the responses in the hopes of recovering their associated NTLM passwords.

```C
[+] Listening for events...
[SMBv2] NTLMv2-SSP Client   : <Client IP>
[SMBv2] NTLMv2-SSP Username : ZA\<Service Account Username>
[SMBv2] NTLMv2-SSP Hash     : <Service Account Username>::ZA:<NTLMv2-SSP Hash>
```

![Breaching Active Directory-6.png](/img/user/cards/red-team/images/Breaching%20Active%20Directory-6.png)

Copy the NTLMv2-SSP hash to a textfile for offline cracking with `hashcat` (All):

```C
hashcat -m 5600 <hash file> <password file> --force
```

**Process:**

1. **Place on the network** - place a device where client broadcasts (LLMNR, NBT-NS, WPAD) are visible.
2. **Activate Poisoning** - of course responder.
### Relaying the Challenge
---
In some cases, you can relay the challenge instead of just capturing it directly. An attacker **forwards that live authentication** to another server and completes the authentication there. If the target accepts the relayed authentication, the attacker obtains an **active authenticated session** on that target as the victim user -- no password recovery needed.

![Breaching Active Directory-5.png](/img/user/cards/red-team/images/Breaching%20Active%20Directory-5.png)

Difficult, requires the following:

- **SMB signing / message integrity**
    - If the server _requires_ SMB signing (or equivalent protection), the attacker cannot tamper with or re-use the authentication message — the server will reject it.

    - If signing is disabled, or allowed but not enforced, the attacker can alter and forward messages and the server may accept them.

- **Privileges of the relayed account**

    - The attack only gives you whatever the victim account can do on the target. If that account has no rights on the target, the relay is useless.

    - To gain meaningful access you generally want an account with appropriate privileges on the target (ideally admin or at least file/write rights).

- **Knowing where to relay (guesswork / intel required)**

    - A blind relay (randomly relaying to servers hoping for a privileged account) has low success probability.

    - Attackers usually prefer to **first gain AD footholds and perform enumeration** (find which accounts have rights on which machines), then target relays where success likelihood is high.
## Microsoft Deployment Toolkit
---
Large organizations need tools to deploy and manage their infrastructure of the estate, they can't have their IT personnel running around with USB flash drives installing software, here comes _Microsoft Deployment Toolkit (MDT)_.

- It is a Microsoft service that assists with automating the deployment of Microsoft Operating Systems including installing default software like Office365. This allows organization to deploy new images more efficiently since these images can be maintained and updated in a central location.

- Integrated with Microsoft's System Center Configuration Manager (SCCM), which manages all updates for all Microsoft applications, services, and Operating Systems.

- This allows the IT team to simply plug in a network cable to the new machine, and everything happens automatically.

- **Patch Management:** It allows the IT team to review available updates to all software installed across the organization.
### Preboot Execution Environment (PXE)
---
Large organizations use PXE boot to allow new devices that are connected to the network to load and install the OS directly over a network connection. MDT can be used to create, manage, and host PXE boot images. PXE boot is usually integrated with DHCP, which means that if DHCP assigns an IP lease, the host is allowed to request the PXE boot image and start the network OS installation process

![Breaching Active Directory-13.png](/img/user/cards/red-team/images/Breaching%20Active%20Directory-13.png)

Once the process is confirmed, the client will use a TFTP connection to download the PXE boot image, there are two exploit against PXE boot image:

- Inject a privilege escalation vector, such as a Local Administrator account, to gain Administrative access to the OS once the PXE boot has been completed.
- Perform password scraping attacks to recover AD credentials used during the install.
### PXE Boot Image Retrieval
---
In this step: the goal is to scrape all possible AD credentials but we need two things:

- The IP address of the MDT server.
- The names of the BCD files. These files store the information relevant to PXE boots for the different types of architecture.

![Breaching Active Directory-11.png](/img/user/cards/red-team/images/Breaching%20Active%20Directory-11.png)

Normally, you would use TFTP to request each of these BCD files and enumerate the configuration for all of them. But in this case focus on the 64-bit machine `x64{GUID}.bcd` (since this is a laboratory and for learning purposes).

- Since `TFTP` cannot list files, you must request the exact filename or the transfer will fail.
 
The file interest: `x64{4D399C59-619E-4484-BF3C-6C5CC4D044D5}.bcd` and proceed to the next step this requires `PowerPxe` tool and then you will have to look up the MDT IP using `nslookup thmmdt.za.tryhackme.com` command.

```C
tftp -i <THMMDT IP> GET "\Tmp\<paste-full-x64-filename>.bcd" conf.bcd
```

- `tftp` — Windows TFTP client.

- `-i` — binary/image mode (required for non-text files).

- `<THMMDT IP>` — IP from `nslookup`.

- `GET` `"\Tmp\<filename>.bcd"`  exact remote path (note the `\Tmp\` and quotes).

- `conf.bcd` — local filename to save as.

You should see something "Transfer successful: ......" then after the `.bcd` files are now recovered, you are going to use powerpxe to read it's contents:

```C
powershell -executionpolicy bypass
Import-Module .\PowerPXE.ps1
$BCDFile = "conf.bcd"
Get-WimFile -bcdFile $BCDFile
```

![Breaching Active Directory-14.png](/img/user/cards/red-team/images/Breaching%20Active%20Directory-14.png)

`Get-WimFile` will parse `conf.bcd` and print the `<PXE Boot Image Location>` (the path to the `.wim`). Copy that path exactly for the next TFTP step:

```C
tftp -i <THMMDT IP> GET "<PXE Boot Image Location>" pxeboot.wim
```

#### Recovering Credentials from a PXE Boot Image
---
Now that you have recovered the PXE Boot image, you can exfiltrate stored credentials. It should be noted that there are various attacks that you could stage. You could inject a local administrator user, so we have admin access as soon as the image boots, we could install the image to have a domain-joined machine. If you are interested in learning more about these attacks, you can read this [article](https://www.riskinsight-wavestone.com/en/2020/01/taking-over-windows-workstations-pxe-laps/).

Recover the credentials using `powerpxe` or do it manually by extracting the image by looking for the `bootstrap.ini` file:

```C
Get-FindCredentials -WimFile pxeboot.wim
Open pxeboot.wim
```

Output:

![Breaching Active Directory-15.png](/img/user/cards/red-team/images/Breaching%20Active%20Directory-15.png)

I did not do the last task intentionally.

## Mitigations
---

- User awareness and training - The weakest link in the cybersecurity chain is almost always users. Training users and making them aware that they should be careful about disclosing sensitive information such as credentials and not trust suspicious emails reduces this attack surface.
- Limit the exposure of AD services and applications online - Not all applications must be accessible from the internet, especially those that support NTLM and LDAP authentication. Instead, these applications should be placed in an intranet that can be accessed through a VPN. The VPN can then support multi-factor authentication for added security.
- Enforce Network Access Control (NAC) - NAC can prevent attackers from connecting rogue devices on the network. However, it will require quite a bit of effort since legitimate devices will have to be allowlisted.
- Enforce SMB Signing - By enforcing SMB signing, SMB relay attacks are not possible.
- Follow the principle of least privileges - In most cases, an attacker will be able to recover a set of AD credentials. By following the principle of least privilege, especially for credentials used for services, the risk associated with these credentials being compromised can be significantly reduced.

Now that we have breached active Directory: next step is to perform [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)\|enumeration of Active Directory]] to gain a better understanding of the domain structure and identify potential misconfigurations that can be exploited.
### Questions and Problems
---
## Conclusion


