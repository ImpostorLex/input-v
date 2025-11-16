---
{"dg-publish":true,"permalink":"/cards/linux/linux-logging-for-soc/"}
---

~ [[atlas/linux\|linux]] | ~ [[atlas/blue-team\|blue-team]]
### Introduction
---
The long leader in servers and embedded systems known because of it's speed and lightweight. With the recent rise of cloud computing Linux OS is now more relevant.

And no unfortunately, Linux now is as popular as Windows when it comes to number of malware developed for the respective OSes.

It's important to note the format of each logs may vary depending on the OS.

#### Key Topics
---

## Working With Text Logs
---
 Default Linux logs are less structured as there are no event codes and strict log formatting rules. Most Linux logs are located in `/var/log` folder, one worth noting is `/var/log/syslog` file -- it agregrates stream of various system events:

**Filter:**

```C
cat /var/log/syslog | grep CRON
```

Or to exclue CRON:

```C
grep -v CRON
```

**Finding that rows of log that you want but don't know which log file is it:**

```C
grep -R -E "auth|login|session" /var/log
```

## Authentication Logs
---
The first and often the most useful log file you want to monitor is `/var/log/auth.log` (or `/var/log/secure` on RHEL-based systems). It also store user management events, sudo commands, and much more.

![Linux Logging for SOC.png](/img/user/cards/blue-team/soc/images/Linux%20Logging%20for%20SOC.png)

**There are many ways user authenticate into a Linux machine:** locally, via SSH, "sudo" or "su" commands, or automatically to run a cron job. Each successful logon and logoff is logged, and can be filtered using "session opened" or "session closed":

```C
cat /var/log/auth.log | grep -E 'session opened|session closed'
```

Sample log file for login on-keyboard and remote login via ssh:

```C
2025-08-02T16:04:43 thm-vm login[1138]: pam_unix(login:session): session opened for user bob(uid=1001) by bob(uid=0)

2025-08-04T09:09:06 thm-vm **sshd**[839]: pam_unix(sshd:session): session opened for user alice(uid=1002) by alice(uid=0)
```

For failed try **"Invalid user"** or **"Invalid password"**:

```C
2025-08-13T15:56:27.989388+00:00 thm-vm sshd[1192]: Invalid user admin from 10.14.94.82 port 57697

2025-08-13T15:56:31.754409+00:00 thm-vm sshd[1192]: Failed password for invalid user admin from 10.14.94.82 port 57697 ssh2
```

It is also worth noting the `SSH daemon` stores its own log of succesful and failed SSH login attempts. These logs are sent to the same `auth.log` file.

**User Management Events:**

Such as `useradd` and the use of `passwd` command:

```C
cat /var/log/auth.log | grep -E '(passwd|useradd|usermod|userdel)\['
```

## Generic System Logs
---
Linux keeps track of many other events scattered across files in `/var/log`: kernel logs, network changes, service or cron runs, package installation, and many more.

- `/var/log/kern.log`: Kernel messages and errors, useful for more advanced investigations
- `/var/log/syslog (or /var/log/messages)`: A consolidated stream of various Linux events
- `/var/log/dpkg.log (or /var/log/apt)`: Package manager logs on Debian-based systems
- `/var/log/dnf.log (or /var/log/yum.log)`: Package manager logs on RHEL-based systems

The listed logs are valuable for DFIR, but are rarely seen in a daily SOC routine as they are often times noisy and hard to parse.
#### App-Specific Logs
---

Some of third party apps have their own log files:

```C
/var/log/nginx/access.log
```

#### Bash History
---
A feature that records each command you run after pressing Enter. By default, commands are first stored in memory during your session, and then written to the per-user `~/.bash_history` file when you log out or use `history` to view past and present commands.

It is rarely ued by SOC as it does not track non-interactive commands (like those initiated by your OS, cron jobs, or web servers) and has some other limitations:

```C
# Attackers can simply add a leading space to the command to avoid being logged
ubuntu@thm-vm:~$  echo "You will never see me in logs!"

# Attackers can paste their commands in a script to hide them from Bash history
ubuntu@thm-vm:~$ nano legit.sh && ./legit.sh
 
# Attackers can use other shells like /bin/sh that don't save the history like Bash
ubuntu@thm-vm:~$ sh
$ echo "I am no longer tracked by Bash!"
```

## Runtime Monitoring
---
None of the previously mentioned logs files can answer questions like "Which programs did Bob launch today?" or "Who deleted my home folder, and when?". That's because, by default, Linux doesn't log process creation, file changes, or network-related events, collectively known as **runtime** events (**events that happen while the system is running**). In other words _Dynamic_ system activity.

- **Static data** - Persistent configuration or state stored on disk (like `/etc/passwd`, `/etc/sudoers`, user accounts, permissions). These define _what can_ happen.

- **Runtime data (events)** - Actions that actually _do_ happen while the system is running (processes starting, files changing, users logging in).

In Windows they faced the same limitations resulting into born of Sysmon.
#### System Calls
---
**A very important concept to understand.** 

Whenever you need to open a file, create a process, access the camera, or request any other OS service, you make a specific system call. Like `execve` to execute a program:

![Linux Logging for SOC - 3.jpg](/img/user/cards/blue-team/soc/images/Linux%20Logging%20for%20SOC%20-%203.jpg)

All modern EDRs and logging tools rely on them - they monitor the main system calls and log the details in a human-readable format. Since there is nearly no way for attackers to bypass system calls, all you have to do is choose the system calls you'd like to log and monitor.

## AuditD
---
Auditd (Audit Daemon) is a built-in auditing solution often used by the SOC team for runtime monitoring. Located in `/etc/audit/rules.d`.

![Linux Logging for SOC - 2.jpg](/img/user/cards/blue-team/soc/images/Linux%20Logging%20for%20SOC%20-%202.jpg)

More rules does not equal to more detection. That's why SOC teams often focus on the highest-risk events and build balanced rulesets, like [this one](https://github.com/Neo23x0/auditd/blob/master/audit.rules).
#### Using Auditd
---
You can view the generated logs in real time in `/var/log/audit/audit.log`, but it is easier to use the `ausearch` command, as it formats the output for better readability and supports filtering options.

**Note:** the `proc_wget` is used/created see the previous image:

```C
ausearch -i -k proc_wget
```

Output:

```C
type=PROCTITLE msg=audit(08/12/25 12:48:19.093:2219) : proctitle=wget https://files.tryhackme.thm/report.zip

type=CWD msg=audit(08/12/25 12:48:19.093:2219) : cwd=/root

type=EXECVE msg=audit(08/12/25 12:48:19.093:2219) : argc=2 a0=wget a1=https://files.tryhackme.thm/report.zip

type=SYSCALL msg=audit(08/12/25 12:48:19.093:2219) : arch=x86_64 syscall=execve [...] ppid=3752 pid=3888 auid=ubuntu uid=root tty=pts1 exe=/usr/bin/wget key=proc_wget
```

The terminal above shows a log of a single "wget" command. Here, auditd splits the event into four lines: the `PROCTITLE` shows the process command line, `CWD` reports the current working directory, and the remaining two lines show the system call details:

- `pid=3888, ppid=3752`: Process ID and Parent Process ID. Helpful in linking events and building a process tree
- `auid=ubuntu`: Audit user. The account originally used to log in, whether locally (keyboard) or remotely (SSH)
- `uid=root`: The user who ran the command. The field can differ from auid if you switched users with sudo or su
- `tty=pts1`: Session identifier. Helps distinguish events when multiple people work on the same Linux server
- `exe=/usr/bin/wget`: Absolute path to the executed binary, often used to build SOC detection rules
- `key=proc_wget`: Optional tag specified by engineers in auditd rules that is useful to filter the events

Though the problem it is too verbose making it hard to ingest into SIEMs. Alternatives exists such as Sysmon for Linux.
