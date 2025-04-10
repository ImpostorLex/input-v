---
{"dg-publish":true,"permalink":"/cards/blue-team/digital-forensics/challenges/reveal-lab/"}
---

[[Challenges TimeKeeping\|Challenges TimeKeeping]]

**Determing OS information**
```C
python3 ~/volatility3/vol.py -f 192-Reveal.dmp windows.info
```

Output:

![Reveal Lab.png](/img/user/cards/blue-team/digital-forensics/images/Reveal%20Lab.png)
The most standout process to me is **net.exe** with a foreign port of **8888**:

![Reveal Lab-1.png](/img/user/cards/blue-team/digital-forensics/images/Reveal%20Lab-1.png)
Command:

```C
python3 vol.py -f ~/Desktop/SOC101/Extra_Challenges/endpoint-forensic/192-Reveal.dmp windows.netstat
```

The next one is **powershell.exe** with a PID of **3692** and PPID of **4120**:

![Reveal Lab-2.png](/img/user/cards/blue-team/digital-forensics/images/Reveal%20Lab-2.png)
I noticed that **wordpad.exe** has the same PPID as the powershell.exe and then since it is a powershell, we can view the command executed:

```C
 python3 vol.py -f ~/Desktop/SOC101/Extra_Challenges/endpoint-forensic/192-Reveal.dmp windows.cmdline
```

Output:

![Reveal Lab-3.png](/img/user/cards/blue-team/digital-forensics/images/Reveal%20Lab-3.png)
So it accessed a network share which in this case it is a `WebDAV` basically does is enables webserver to act as a file server then executed 3435.dll with the `rundll32` command, we can also see the network share IP address and it's port number.

The `rundll32` command executed a specific function in `3435.dll` named **entry**, the attack technique make use of **windows utility** to execute malicious code and avoid triggering security tools as they are known or signed binaries by Microsoft.

44 mins

