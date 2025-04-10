---
{"dg-publish":true,"permalink":"/cards/windows/session-manager-subsystem-smss-exe/"}
---

[[cards/blue-team/endpoint-security/Windows Process Analysis\|Windows Process Analysis]]
### Introduction
---
Launches processes at startup these are _wininit.exe_ and _csrss.exe_ (client server runtime subsystem)

- **There should be no more than two instances of `smss.exe` at any given time**, and
- **One instance should terminate after completing its tasks, leaving only a single `smss.exe` running.**
## Normal Process Flow Summary
---
- `smss.exe` (Parent) → Spawns child `smss.exe` → Launches `csrss.exe` & `wininit.exe` → Child `smss.exe` terminates.
- `wininit.exe` → Starts critical system services (remains running).
- `smss.exe` (Parent) → Spawns `winlogon.exe` → Manages user login (remains running).
- `winlogon.exe` → Launches `userinit.exe` → Loads the user desktop (Explorer.exe).

This process put csrss.exe and *winnit.exe* in Session 0:

![Session Manager Subsystem (smss.exe).png|center](/img/user/cards/windows/images/Session%20Manager%20Subsystem%20(smss.exe).png)
And another isolated Windows session for csrss.exe and *winlogon.exe* which is the user session:

![Session Manager Subsystem (smss.exe)-1.png](/img/user/cards/windows/images/Session%20Manager%20Subsystem%20(smss.exe)-1.png)
It provides separation between system-level processes and user-level processes, enhances system stability and security, and ensures that critical system tasks are executed with the necessary privileges

- Is also responsible for creating environment variables, virtual memory paging files and starts winlogon.exe.
- Any other subsystem listed in the `Required` value of `HKLM\System\CurrentControlSet\Control\Session Manager\Subsystems` is also launched.

![Session Manager Subsystem (smss.exe)-2.png](/img/user/cards/windows/images/Session%20Manager%20Subsystem%20(smss.exe)-2.png)




