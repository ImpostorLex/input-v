---
{"dg-publish":true,"permalink":"/cards/blue-team/digital-forensics/volatility/"}
---

[[atlas/blue-team\|blue-team]]
### Introduction
---
A tool used to analyze snapshot of the systems memory, it supports the three commonly used operating system, these are: Windows, MacOS, and Linux.

- [Volatility Notes](obsidian://open?vault=notes&file=Atlas%2FCyber%20Defense%2FVolatility)
### Analysis
---
- Source file [here](https://drive.google.com/file/d/1arIElSgK7k_Bz2C4bdzcuX7zkskYiTOV/view) password: `infected`.
- Always ensure you are analyzing a honest and the intended image by hashing.

**Identify what operating system and other metadata**:
```Python
python3 vol.py -f <filename> windows.info
```

- Other options are: `mac.info` and `linux.info`.
- It will say invalid choice, if the given option is incorrect.

**Network Memory Analysis:**
```Python
python3 vol.py -f <filename> windows.netstat
```

- Perform `whois` lookup on suspicious IP address.
- Take note of the Process ID.
- `windows.netscan` a much more powerful version.

**Process Memory Analysis:**
```Python
python3 vol.py -f <filename> windows.pslist
```

- Additionally add the `--pid` for specific process.
- Remember the [[cards/blue-team/endpoint-security/Windows Process Analysis\|Windows Process Analysis]], what's normal and what is not? what should have one instance. We can use `grep services.exe` to quickly determine the number of instances.

In this case the `services.exe` has a PID of 636, so `svchost.exe` should have the PPID:
![Volatility.png](/img/user/cards/blue-team/digital-forensics/images/Volatility.png)
**note:** due to grep formatting issue the **500** as seen in the image is the PPID of services.exe

Finding evil:
![Volatility-1.png](/img/user/cards/blue-team/digital-forensics/images/Volatility-1.png)
The `svchost.exe` is opened via the file explorer:
![Volatility-2.png](/img/user/cards/blue-team/digital-forensics/images/Volatility-2.png)
**Display the raw process:** it will display all unlinked, terminated and hidden processes (that some malware does):

```C
python3 vol.py -f <file> windows.psscan
```

**Display parent-process format:** 

```python
python3 vol.py -f <file> windows.pstree
```

- `--pid`
- It also includes the commandline invocation. so remember the `svchost.exe -k parameter`.

**Display all commandline invocation of processes:**
```python
python3 vol.py -f <file> windows.cmdline
```

- `--pid`

**Dumping the bit for bit binary of specific process**
```python
python3 vol.py -f <filename> -o <output> windows.pslist --pid <pid>
```

**Remember** in the image memory capture analysis, we are copying the bit for bit and all of it's dependencies, so we can perform malware analysis or check against a database.

`dlllist` plugin will list all `.dll` files that is associated with a processes.

```python
python3 vol.py -f <file> windows.dlllist
```

**Registry Memory Analysis**

Viewing usage of programs, execution counts, focus times, user and more:
```Python
python3 vol.py -f <file> windows.registry.userassist | less
```

Output: 
![Volatility-3.png](/img/user/cards/blue-team/digital-forensics/images/Volatility-3.png)
- **3** - number of times used.
- **4** - number of focus count. (this and below is not 100% accurate)
- **0:02:54** - focus time / duration

**View/list specific hives:**

```python
python3 vol.py -f <file> windows.registry.hivelist
```

- `--filter "Bill Daughtry"` to retrieve hives that are available for this user.
	- `--filter "Bill Daughtry\ntuser.dat"` then dump it and analyze it using tools such as registry explorer.
- `-o ~/Desktop/` then at the end `--dump`.

**Viewing sessions: as other way to determine which session belong to user or proccess**:

```C
python3 vol.py -f ~/Desktop/SOC101/Extra_Challenges/endpoint-forensic/192-Reveal.dmp sessions
```

**Query specific registry like `reg query`:**
```python
python3 vol.py -f <file> windows.registry.printkey
```

- `--key "Software\Microsoft\Windows\CurrentVersion\Run"`