---
{"dg-publish":true,"permalink":"/cards/linux/linux-proc-file-system/"}
---

~ [[cards/blue-team/endpoint-security/Linux Process Analysis\|Linux Process Analysis]] | ~ [[atlas/linux\|linux]]

- location: `/proc`
- It is a virtual file system that does not correspond to any physical storage, all of it is in memory.
- It provides an interface for the kernel to provide information on various running processes on the system.

![Linux -  proc file system.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20-%20%20proc%20file%20system.png)
- directories in blue with numbers are the processes itself.

Viewing the process of the reverse shell:

![Linux -  proc file system-1.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20-%20%20proc%20file%20system-1.png)
- **cmdline** refers to the argument given - we can `cat` this out
- cwd or `exe`
- `environ` - such as special settings

```bash
cat environ | tr '\0' '\n'
```

