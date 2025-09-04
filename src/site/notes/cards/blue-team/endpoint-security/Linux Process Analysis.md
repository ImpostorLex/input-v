---
{"dg-publish":true,"permalink":"/cards/blue-team/endpoint-security/linux-process-analysis/"}
---

[[atlas/blue-team\|blue-team]]

- A program, command, a script is a process
- [[cards/linux/Linux -  proc file system\|/proc file directory]] proc = processes
	- We can also view the `cmdline` directory or the argument given to a executable but we need the process ID of the binary.
	- `cwd` or the current working directory.
	- `environ` shows environment variables.

**View process for specific user**
```bash
sudo ps -u tcm
```

**System wide process**

```bash
sudo ps -AFH | less
```

- `-A` all
- `-F` full format response
- `-H` forest output parent-child

![cards/blue-team/endpoint-security/images/Linux Process Analysis.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Process%20Analysis.png)
**List out all child of parent by providing PID of parent**

```bash
sudo ps --ppid 3401
```

**Visualize the hierarchy of a process**

```bash
pstree -ps 3401
```

![Linux Process Analysis-1.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Process%20Analysis-1.png)

**Dynamic update process list by showing recent process**

```bash
sudo top -u tcm -c -o -TIME+
```

- `-c` verbose
- `-o` sort via `time`
- Then navigate via up and arrow keys.

**Shows deleted files or malwares but still in use by other processes**
```
lsof +L1
```

- Why it remains? because it is still in our memory and in used by other processes
- You can still extract the hash by `sha256sum exe`

**Great if you suspect a user has only access to his/her home directory** or for specific processes running in a specific directory:

```C
lsof +D /home/mircoservice
```

Also used to check for hidden process.

![cards/blue-team/soc/images/Linux Process Analysis.png](/img/user/cards/blue-team/soc/images/Linux%20Process%20Analysis.png)

**Viewing malware under services**

```C
systemctl list-timers --all
```

Or

```C
systemctl list-unit-files --type=service --all  
```

Output:

![cards/blue-team/siem/images/Linux Process Analysis.png](/img/user/cards/blue-team/siem/images/Linux%20Process%20Analysis.png)

Note Enabled - Enabled | STATE and PRESET look for services that both columns enabled.

Since we are looking for persistence mechanism at least the `STATE` must be enabled.

- `STATE` - This is the **actual current state** of the service unit file in the file system.
- `PRESET` - This shows what the **preset policy** suggests should be done with this service — based on the system-wide or vendor-supplied policy

Look into here as well: 

```C
/etc/systemd/system/
```

**Show all recently modified or added services:** 

```C
find /etc/systemd/system /lib/systemd/system -type f -name "*.service" -exec stat --format '%y %n' {} \; | sort
```

