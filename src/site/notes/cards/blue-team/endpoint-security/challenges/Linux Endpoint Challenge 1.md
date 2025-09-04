---
{"dg-publish":true,"permalink":"/cards/blue-team/endpoint-security/challenges/linux-endpoint-challenge-1/"}
---

~ [[atlas/blue-team\|blue-team]] 

```bash
sudo lsof -i -P -n
```

Output: found a suspicious listening network connection at port **31337** with a command name of **kitty.meow**

![Linux Endpoint Challenge 1.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Endpoint%20Challenge%201.png)

```bash
sudo lsof -p 3211
```

Output: it shows the working directory of where the process (binary) executed, as well as another binary with the name of **kitty.meow**, this most likely is the binary responsible for opening up network listener.

![Linux Endpoint Challenge 1-1.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Endpoint%20Challenge%201-1.png)
Navigating to the `/proc/3211` to determine what is **kitty.meow**:

![Linux Endpoint Challenge 1-3.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Endpoint%20Challenge%201-3.png)
The parameters and argument confirms that `-p` flag sets the listening port to 31337 which matches with the `lsof` command previously executed and as for the `-l` flag it is most likely stands for listening:
![Linux Endpoint Challenge 1-2.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Endpoint%20Challenge%201-2.png)
Submitting the `exe` hash to VirusTotal:

![Linux Endpoint Challenge 1-4.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Endpoint%20Challenge%201-4.png)
- Based on the result - netcat

```bash
sudo ps -AFH | less
```

Output: the parent process id of the **kitty.meow** is 1972

![Linux Endpoint Challenge 1-5.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Endpoint%20Challenge%201-5.png)
**Visualizing the hierarchy**: my theory why 3198 is not the `parent ID` of **kitty.meow** is that it got executed by other means, so basically **challenge.elf** only 'setups' the environment

```bash
pstree -ps 3401
```

![Linux Endpoint Challenge 1-6.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Endpoint%20Challenge%201-6.png)

```bash
sudo cat /etc/crontab
```

Output: shows that it will execute _At 17:45 on day-of-month 13 in October._

![Linux Endpoint Challenge 1-7.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Endpoint%20Challenge%201-7.png)
Analyzing the `.elf` file using `strings`:

![Linux Endpoint Challenge 1-8.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Endpoint%20Challenge%201-8.png)
