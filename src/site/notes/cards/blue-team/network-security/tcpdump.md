---
{"dg-publish":true,"permalink":"/cards/blue-team/network-security/tcpdump/"}
---

[[Network Security MOC\|Network Security MOC]]

- Great for remote network capturing.
- `-i` to specify interface. (index number or interface name as value).
	- `tcpdump -D` display available interfaces.
- `any` captures all traffic from any interface but
	- **promiscuous mode** it is a configuration setting on _network interface cards_ that allows them to pass all the traffic receive to the operating system is not available.
	- Normally: It processes frames or packet that is addressed to it's MAC address, broadcast, and multicast frames.
- Ideally **filters should be wrapped in quotes**

- `-X` print each packet **in hexademical and ASCII format**.
	- `-A` **ASCII only**.
- `-r` **read from a file**
- **timestap**
	- `-t` no timestamp
	- `-tt` unix epoch time (UTC)
	- `-ttt` print in miliseconds since the previous packet.
	- `-tttt` full readable standard datetime.

```
sudo tcpdump -i enp0s3 [OPTIONS] [VALUES]
```

- `AND` must pass the two conditions.
- `OR` either one of the condition.
- `NOT` does not match the condition.
- `(port 443 or port 80)` treat this as a singular logical unit.

### Capturing Network Traffic
---
- `-n` **don't resolve ports and IP address**.
- `-w ~/Desktop/folder1/capture.pcap` save to a file.  

- **capture only from specific host**
	- `host examlocalhostple.com` or `host 8.8.8.8`
	- `src 10.0.2.15` - captures packet originating from.
	- `dst google.com` - captures packet destinated from.

- **capture traffic from specific network only**
	- `net 10.0.0.0/8`
	- Additionally you can add `src net` or `dst net`

- `port 21`
	- `src port 21` or `dst port 21`

- **filter based on protocol**
	- `tcp`, `arp`, `udp`, `icmp`.

## Methodology
---
**Note:** `less` can be useful and `cut`.

1. Get the whole picture (go up to the rooftop) and don't include `-n` flag yet.
2. Filter the packet of interest.
	- Good idea to use `--count` to determine how many packets are we dealing.
	- USE **GREP** such as `grep -E "GET|POST"` 
3. Analyze the request to build context and check IP reputation.

**Get src address**

```bash
sudo tcpdump -tt -r tcpdump_challenge.pcap -n tcp | cut -d " " -f 3 | cut -d "." -f 1-4  | sort | uniq -c | sort -nr
```

- Replace **3** with **5** for destination address (in a typical pcap format.)
- Replace **1-4** with **5** to get only PORT NUMBERS.

Output (without the port number got lazy lol):

![tcpdump.png](/img/user/cards/blue-team/network-security/images/tcpdump.png)

**Filter for login credentials**:

```C
tcpdump -tt -r file.pcap host 10.0.0.2 -A | grep -i "user\|pass\|login" | grep -v "User-Agent"
```

- Sometimes http headers include filename key so we can use `grep "filename"`


Better formatting add this:
- `| awk NF` - removes empty lines.
- `| sort -r` - sort recursively before handling values.
	- `-nr` to sort against the count values.
- `| uniq -c` - Show unique values and number of occurences.
- `| sort -f` - Sort from high occurences to less.