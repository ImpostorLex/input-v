---
{"dg-publish":true,"permalink":"/cards/windows/service-host-svchost-exe/"}
---

[[cards/blue-team/endpoint-security/Windows Process Analysis\|Windows Process Analysis]]
### Introduction
---

![Service Host (svchost.exe).png](/img/user/cards/windows/images/Service%20Host%20(svchost.exe).png)
Services that is running on the svchost.exe are implemented as [[DLL\|DLL]] and are stored in `Parameters` subkey in `ServiceDLL`:

```Powershell
HKLM\SYSTEM\CurrentControlSet\Services\<SERVICE NAME>\Parameters
```

Example:

![Service Host (svchost.exe)-1.png](/img/user/cards/windows/images/Service%20Host%20(svchost.exe)-1.png)

From Process Hacker: `svchost.exe` -> `service` and `Properties`:

![Service Host (svchost.exe)-2.png|350](/img/user/cards/windows/images/Service%20Host%20(svchost.exe)-2.png)
The `-k` in the binary path is how legitimate svchost.exe process is called with the purpose of grouping similar services to share the same process with the goal of reducing resource consumption each service runs its own process.

![Service Host (svchost.exe)-3.png](/img/user/cards/windows/images/Service%20Host%20(svchost.exe)-3.png)

Extra reading - [Hexacorn Blog](https://www.hexacorn.com/blog/2015/12/18/the-typographical-and-homomorphic-abuse-of-svchost-exe-and-other-popular-file-names/)



