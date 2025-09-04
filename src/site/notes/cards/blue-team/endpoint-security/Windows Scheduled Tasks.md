---
{"dg-publish":true,"permalink":"/cards/blue-team/endpoint-security/windows-scheduled-tasks/"}
---

[[atlas/blue-team\|blue-team]]

- Managed by Task Scheduler Utility.

**Demonstrations**

```Powershell
schtasks /create  /tn "SystemCleanupNotMalware" /tr  "C\users\notmalware.exe" /sc daily /st 09:00 /ru SYSTEM
```

## Investigations
---
- Look for tasks run by Administrator or any high privileged accounts.
- Tasks with no username sets or tasks run from suspicious locations.
- Tasks that runs every minute or hour.

- **Task Scheduler** graphical
- **cmd** version
- It can also be found on **AutoRuns** sysinternal.

```PowerShell
schtasks /query /fo LIST /v
```

- you can use `TABLE` instead of list

Query a specific tasks:

```PowerShell
schtasks /query /tn "SystemCleanup" /v /fo LIST
```

- You can use the AutoRuns module to compare baselines [[cards/blue-team/endpoint-security/Windows Registry#Baselining from scratch\|here]].
