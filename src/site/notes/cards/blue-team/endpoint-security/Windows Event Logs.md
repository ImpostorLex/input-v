---
{"dg-publish":true,"permalink":"/cards/blue-team/endpoint-security/windows-event-logs/"}
---

[[Endpoint Security MOC\|Endpoint Security MOC]]

- **Event ID: 4624 (Logon Successs)** we can use the Logon ID to correlate other data.
	- **Logon Type**: 2 for someone actually behind the machine, and 3 someone logon to the machine using the network.
- (Under Windows Logs) **System** event types is just as import as **Security** event types especially when correlating events.

### Security Event IDs
---
- **4720** user acc was created.
	- Persistence, user required exploit and muc more.
- **4722** a user account was enabled from disabled.
	- Attacker reactiving account for persistence or access.
- **4723** attempt was made to change an account's password. (old password did not match.)
	- Bruteforce
- **4724** an attempt was made to reset an account's password. (Forgotten)
- **4738** a user account was changed such as account group membership
- **4725** a user account was disabled.
	- Lockout from brute-force attempt or prevent legitimate login.
- **4726** a user account was deleted.
	- Cleaning out their tracks.
- **4732** a member was added to a security-enabled local group.
	- Security enabled settings such as file permissions on specific directories or active directory only in **local computer**.
- **4688** a new process has been created. (not logged by default)
- **1102** the audit log was cleared
### System Event IDs
---
- **7045** - a service was installed in the system.
	- Persistence
- **7030** - The Service Control Manager tried to take a corrective action (Restart the service)
- **7035** - The SCM is enabling a service to a running, stopping, and pausing state.
- **7036** - The SCM reports that a service has entered the running state.
## Headless Query
---

```cmd
wevtutil qe security /c:5 /f:text /rd:true
```

- `qe` query events
- `/c` give back 5 only.
- `/f:text` output in text only not XML.
- `/rd:true` reverse means that only the latest events.

**Query specific event ID**:

```bash
wevtutil qe security /c:5 /f:text /rd:true /q:"*[System[(EventID=4720)]]"
```

**Powershell**

```bash
Get-WinEvent -LogName System or Security
```

Filter [hash tables refer here](https://learn.microsoft.com/en-us/powershell/scripting/samples/creating-get-winevent-queries-with-filterhashtable?view=powershell-7.4):

![Windows Event Logs.png](/img/user/cards/blue-team/endpoint-security/images/Windows%20Event%20Logs.png)
**Example pull back 4624 Event ID**

```Powershell
Get-WinEvent -FilterHashTable @{logname='Security'; ID=4624;} -MaxEvents 2
```

- Add `| Format-List *` to pull all information.