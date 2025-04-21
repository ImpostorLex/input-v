---
{"dg-publish":true,"permalink":"/cards/blue-team/endpoint-security/linux-process-analysis/"}
---

[[map-of-contents/blue-team\|blue-team]]

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

![Linux Process Analysis.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Process%20Analysis.png)
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