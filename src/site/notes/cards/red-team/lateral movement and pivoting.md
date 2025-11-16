---
{"dg-publish":true,"permalink":"/cards/red-team/lateral-movement-and-pivoting/","tags":["red-team/ad"]}
---

~ [[atlas/red-team\|red-team]]
### Introduction
---
A group of techniques used by the attackers to move around a network. Moving is essential to reach your goals as attacker (which in this case as a red teamer to expose as much as you can to help fill those gaps).

Though the cyber kill chain reference lateral movement as a linear proces, it is actually a cycle, once you gained another credentials you move again and again.

![lateral movement and pivoting-xx.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting-xx.png)

Of course this depends on your goal such as accessing a code repository from the developers department but your initial foothold is a PC from marketing team via phishing campaign.

![lateral movement and pivoting0xx.png|450](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting0xx.png)
#### Administrators and UAC
---
There are users accounts created directly on one machine, not in the domain:

- Administrators
- AdminUser
- Any local account in Local Administrator group

They exists only on that machine, and their passwords don't apply to other computers. Next is Active Directory accounts. (e.g, `CORP\Alex`) that are members of the **Administrators group** on the local machine - either directly or through a domain group like `Domain Admins`. These credentials works across multiple systems.

Windows has a built-in protection called **User Account Control (UAC)**.  

![lateral movement and pivoting-xzz.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting-xzz.png)

But UAC also affects **how admin privileges are used remotely**.

**Here’s the key behavior:**

- If you use a **local administrator account** (not the default one) to **remotely** connect (like over **SMB, RPC, or WinRM**):

    - Windows gives you a **“filtered” token** - basically, **you log in, but with limited rights**.
    
    - You can’t perform most administrative actions remotely.
       
    - This is a **security restriction** to reduce the risk of malware spreading laterally.

- Only the **built-in Administrator account** (the one created when Windows was installed) is **exempt** - it always gets **full privileges** remotely.

When a **domain account** (like `CORP\AdminUser`) has local admin rights, it’s not affected by this UAC filtering.  
So when you connect remotely with that account:

- You get a **full admin token**
    
- You can perform **administrative actions** remotely (copy files, run commands, manage services, etc.)

That’s why **domain admin credentials** (or any domain account added to local Administrators) are **powerful for lateral movement** — they bypass this limitation.

## Knowledge Base
---
Username: jenna.field Password: Income1982
### Core Techniques
---
- [[cards/red-team/Spawning Process Remotely\|TECH – T1021 Remote Services]]
- [[cards/red-team/Moving Laterally using WMI\|TECH – T1047 Windows Management Instrumentation (WMI-Based Lateral Movement)]]
- [[cards/red-team/Use of Alternate Authentication Material\|TECH – T1550 Use of Alternate Authentication Material]]
- [[cards/red-team/Abusing User Behavior\|TECH – T1574.008 Hijack Execution Flow:]]
- [[cards/red-team/Port Forwarding\|TECH – T1090 Proxy / Port Forwarding & Pivoting]]

## Tools Commonly Used
---

### Detection & Defense Response
---


## Case Notes / Labs

- INC | 20251015 - THM-Blue-Lab-Lateral-Movement
- INC | 20250920 - HTB - Pivoting Exercise