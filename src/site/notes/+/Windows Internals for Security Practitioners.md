---
{"dg-publish":true,"permalink":"//windows-internals-for-security-practitioners/"}
---

~ [[Parent\|Parent]]
### Introduction 
---
This note is part of on-going series of learning host evasion after successfully completing the TryHackMe module to supplement my knowledge: [[Windows Defense Evasion\|Windows Defense Evasion]].

#### Key Topics
---

## Prerequisites

- [[cards/windows/Windows Internals\|Windows Internals]]

## Windows API Basics
---
Creating a basic program to spawn `notepad.exe` using `CreateProcess` as attackers commonly used Windows API to for process injection, memory manipulation and more.

see source code here on github:

In order to use `CreateProcessA`, we need to follow based on the documentation:
![Windows Internals for Security Practitioners-3.png](/img/user/+/images/Windows%20Internals%20for%20Security%20Practitioners-3.png)

After compiling and executing:

```cpp
g++ mycode.cpp -o myexe.exe
```

Then:
![Windows Internals for Security Practitioners.png](/img/user/+/images/Windows%20Internals%20for%20Security%20Practitioners.png)
### Observing with Sysmon
---
Event IDs that we are interested in:

- **Event ID 1** - Process creation.
- **Event ID 10** - Process access.
- **Event ID 11** - File creation.

![Windows Internals for Security Practitioners-2.png](/img/user/+/images/Windows%20Internals%20for%20Security%20Practitioners-2.png)

What about `CreateProcess` Windows API? unfotunately Sysmon is not built for that. Sysmon is designed for **forensic visibility and long-term monitoring**, not real-time API tracing. It focuses on key telemetry that aids incident detection and investigation.
### Process Monitor
---
In process monitor (for demonstration purposes) add the `createProces.exe` as filter and execute the binary again:

![Windows Internals for Security Practitioners-4.png](/img/user/+/images/Windows%20Internals%20for%20Security%20Practitioners-4.png)
We can see that in just a simple program, it generated a lot of background noises, therefore the best case scenario is to go for low hanging fruits such as filtering for 'File', 'Registry', and processes with ParentId of our executable.



### Questions and Problems
---
## Conclusion


