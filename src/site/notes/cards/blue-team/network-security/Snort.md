---
{"dg-publish":true,"permalink":"/cards/blue-team/network-security/snort/"}
---

[[cards/blue-team/network-security/Intrusion Detection and Prevention System\|Intrusion Detection and Prevention System]]

- A network based.
- It is on **promiscuous mode**, it will capture any packet that traverses to the interface regardless of their destination address.
- `/etc/conf`
- Can be used on wireshark and tcpdump.
- ORDER matters in rule making.

Test configuration:

```bash
sudo snort -T -c /etc/snort/snort.conf
```
## Rules
---
**Note:** Refers to any communication that operates at the **network layer** of the TCP/IP model..

![Snort.png](/img/user/cards/blue-team/network-security/images/Snort.png)
**Alert** mode but show it in console immediately:
- `fast` is ideal for quick investigation for colleration with SIEM.
- [Create Snort rules Web GUI](https://anir0y.in/snort2-rulgen/)

```bash
sudo snort -A console -l /var/log/snort -i enp0s3 -c /etc/snort/snort.conf -q
```

- `-l` where logs will be generated.
- `-q` quite mode no need for banner and initialization. 

Output:

![Snort-1.png](/img/user/cards/blue-team/network-security/images/Snort-1.png)
## Intrusion Detection and Prevention System
---
- `-Q` runs in **in-line mode**, required for snort to be an IPS or IDS
- [[cards/blue-team/network-security/Snort IPS mode - why the need for two network interface?\|Snort IPS mode - why the need for two network interface?]]

```bash
sudo snort -A console -l /var/log/snort -i enp0s3:enp0s8 -c /etc/snort/snort.conf -q -Q --daq afpacket
```

### Mini-Challenge
---
Read and run:

```bash
sudo snort -r 1.pcap -A console -c /etc/snort/snort.conf -q
```

- `-d` to decode show ASCII and Hex output (when reading a snort file or .pcap)

#### 1. Snort rules to alert when .exe is download over http
---
Ideally with a `nocase` for case insensitive as attacker might bypass with miscapitalization:

```bash
alert tcp any any -> any 80 ( msg:"Executable file downloaded over http"; content:"|2e|exe"; http_uri; content:"|2e|exe"; http_uri; content:"|2e|exe"; nocase; http_uri; sid:10000001; rev:1; ) 
```

`Content-Type: application/x-msdownload` is commonly associated with executables being downloaded so browser knows how to handle the file:

```bash
alert tcp any 80 -> any any (msg:"Content-Type: application/msdownloads associated with executables found"; content:"Content-Type: application/x-msdownload"; http_header; sid:10000001; rev:1;)
```

- `-> any any` since the server from where we downloaded our file is making a response from our request therefore this part is where the 'client' receives the response.
- `any 80` refers to the server responding to our request.
- `http_header` means that the content can be found on http_header;

However the attacker can still bypass the above following, so we should try and detect the magic byte or the file signature with `file_data`:

Hexadecimal pattern `4D 5A`:

```bash
alert tcp any 80 -> any any ( msg:"HTTP payload contains DOS MZ or PE executable file signature"; file_data; content:"|4D 5A|"; depth:2; sid:10000001; rev:1;)
```

- `file_data` tells snort to inspect the payload.
- `depth` how deep should snort investigate in this case first **two bytes only**.
- Note `4D 5A` is converted from `MZ`

**User-Agent** example:

```bash
alert tcp any any -> any 80 ( msg:"SSLoad detected on User-Agent Potential Malware Infection"; content:"User-Agent: SSLoad/1.1"; http_header; sid:10000001; rev:1;)

```
#### 2. SSH Traffic
---

```bash
alert tcp any any -> any 22 ( msg:"SSH brute force attack"; flow:to_server,established; threshold:type both, track by_src, count 5 , seconds 30; sid:10000001; rev:1; )  
```

 1. `flow:to_server,established;`
	- **`flow` keyword**: Defines the direction and state of the traffic to match.
	- **`to_server`**: Specifies that this rule applies only to traffic **going to the SSH server** (port 22 on the destination).
	- **`established`**: Indicates the TCP session must be **fully established**, meaning the three-way handshake (SYN, SYN-ACK, ACK) has completed.
2.  `threshold:` Introduces a rate-limiting mechanism to prevent too many alerts.
	- **`type both`**: Applies to both event suppression and detection (alerts will fire at count and then be suppressed until reset, avoid duplication).
	- **`track by_src`**: Tracks the rule by the **source IP address**, meaning Snort counts events from each unique attacker separately or in short **track how many attempts from single source**.
	- **`count 5`**: If 5 matching packets occur (SSH login attempts) within the defined period...
	- **`seconds 30`**: ...within **30 seconds**, an alert is triggered.