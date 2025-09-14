---
{"dg-publish":true,"permalink":"/cards/red-team/firewall-evasion-techniques/","tags":["red-team/host-evasion"]}
---

~ [[atlas/red-team\|red-team]]
### Introduction 
---
A firewall is a software or hardware that monitors the network traffic and compares it against a set of rules before passing or blocking it.

Different types of firewalls are capable of inspecting various fields; basic firewall should be able to inspect at least the following fields (_red highlight_).

The following figure shows the field we expect to find in an IP header:

![Firewall Evasion.png|450](/img/user/cards/red-team/images/Firewall%20Evasion.png)

TCP header:

![Firewall Evasion-1.png|450](/img/user/cards/red-team/images/Firewall%20Evasion-1.png)
#### Key Topics
---

- [[cards/red-team/Firewall Evasion Techniques#Firewall Capabilities\|Explains different types of firewalls (packet-filtering, stateful, proxy, NGFW, cloud) and what layers they inspect.]]

- [[cards/red-team/Firewall Evasion Techniques#Evasion via Controlling the Source MAP/IP/Port\|Techniques to disguise scans with Nmap: decoys, proxies, spoofed MAC/IP, and fixed source ports.]]

- [[cards/red-team/Firewall Evasion Techniques#Evasion via Forcing Fragmentation, MTU, and Data Length\|Using packet fragmentation and MTU manipulation to evade inspection.]]

- [[cards/red-team/Firewall Evasion Techniques#Evasion via Modifying Header Fields\|Tweaking IP header fields (TTL, IP options, bad checksum) to evade firewalls or fingerprint monitoring devices.]]

- [[cards/red-team/Firewall Evasion Techniques#Evasion Using Port Hopping\|Switching communication ports during scanning or sessions to bypass blocks and monitoring.]]

- [[cards/red-team/Firewall Evasion Techniques#Evasion via Port Tunneling\|Forwarding traffic through allowed ports (e.g., 443 → 25) using tunneling techniques like ncat.]]

- [[cards/red-team/Firewall Evasion Techniques#Evasion using Non-Standard Port\|Running services on uncommon or high ports to evade basic firewall rules.]]

- [[cards/red-team/Firewall Evasion Techniques#All said and done NGFW kills them all\|Explains why Next-Generation Firewalls (NGFW) neutralize most evasion methods.]]

- [[cards/red-team/Firewall Evasion Techniques#Questions and Problems\|Clarifies misconceptions about spoofing (e.g., Nmap decoys, anti-spoofing filters) and detection by analysts.]]

## Prerequisites

- [[atlas/networking\|networking]] - ISO/OSI layers and TCP/IP layers. [[cards/networking/The threeway handshake\|The threeway handshake]]
- nmap

## Firewall Capabilities
---
Most basic firewall focus on layers 3, 4 and to a lesser extent, layer 2. Next-generation firewall are also designed to cover layers 5, 6, and 7. The more layers a firewall can inspect, the more processing power it needs.

- **Packet-Filtering Firewall**: Checks packet headers (IP, ports, protocol) but not contents; stateless.
    
- **Circuit-Level Gateway**:  Validates TCP handshakes before allowing sessions.
    
- **Stateful Inspection Firewall**: Tracks active sessions and only allows packets from valid connections.
    
- **Proxy Firewall (WAF)**: Acts as an intermediary and inspects application payloads, mainly for web traffic.
    
- **Next-Generation Firewall (NGFW)**:  Provides deep inspection, application awareness, and advanced threat protection.
    
- **Cloud Firewall (FWaaS)**:  Firewall delivered as a scalable cloud service, often with NGFW-like features.

## Evasion via Controlling the Source MAP/IP/Port
---
When scanning a host behind a firewall, the firewall will usually detect and block port scans.

Introducing `nmap`, allows you to hide or spoof the source as you can use:

1. Decoy(s)
2. Proxy
3. Spoofed MAC Address
4. Spoofed Source IP address
5. Fixed Source Port Number

**A simple `SYN` or stealth scan against top 100 common ports:**

```C
nmap -sS -Pn -F MACHINE_IP
```

- `-Pn` to skip host discovery and testing whether the host is live.
- `-F` top 100 ports.

![Firewall Evasion-2.png|450](/img/user/cards/red-team/images/Firewall%20Evasion-2.png)

1. The scan generated over 200 packets because each port is sent a second SYN packet if it does not reply to the first one.
2. Source port is **37710**.
3. The total length of the IP packet is 44 bytes, 20 bytes for the IP header, which leaves 24 bytes for the TCP header. No data is sent via TCP.
4. TTL is 42.
5. No errors are introduced in the checksum.
### Decoy(s)
---
Hide your scan with decoys. Using decoys makes your IP address mix with other "decoy" IP addresses. Effectively exhausting the blue team. `-D` for decoys.

```C
nmap -sS -Pn -D 10.10.10.1,10.10.10.2,ME -F MACHINE_IP
```

Wireshark capture:

![Firewall Evasion-3.png|450](/img/user/cards/red-team/images/Firewall%20Evasion-3.png)

Omiting the `ME` will make Nmap put your real IP address in random position, you can also set Nmap to use random source IP address instead using the `RND` for random:

```C
nmap -sS -Pn -D RND,RND,ME -F MACHINE_IP
```

Wireshark capture that shows two random IP address used:

![Firewall Evasion-4.png|450](/img/user/cards/red-team/images/Firewall%20Evasion-4.png)
### Proxy
---
Relaying the port scan via proxy helps keeps your Ip address unknown, the target will log the proxy Ip address, you can specify a list of proxy Ip address using a comma seperated list:

```C
nmap -sS -Pn --proxies 10.10.13.37, 10.10.13.37 MACHINE_IP
```
### Spoofed MAC address
---
This technique is tricky; this will only work as long as you and the target system is on the same network segment, the target system is going to reply to a spoofed MAC address, if you are not on the same network you will not receive and capture the responses.

```C
--spoof-mac MAC_ADDRESS
```

You can make your scans appear as if coming from a network printer.
### Spoofed IP address
---
Same with MAC address spoofing you need to be on the same network. Another use for spoofing IP address is when you control the system that has that particular IP address, this is great when the target started blocking the spoofed Ip address you can use the Ip address of the machine you can control.

```C
-S IP_ADDRESS
```

Both IP and MAC address spoofing technique should be applied to trust relationship based on IP and MAC address.
### Fixed Source Port Number
---
Scanning from one particular source port number can be helpful if you discover that the firewalls allow incoming packets from particular source port numbers, such as port 53 or 80.

```C
nmap -sS -Pn -g 8080 -F MACHINE_IP # or --source-port
```

## Evasion via Forcing Fragmentation, MTU, and Data Length
---
Running a Nmap TCP port scan means that the IP packet will hold 24 bytes, why do this? explained here [[cards/red-team/Network Security Solutions#Use Session Splicing (IP Packet Fragmentation)\|here.]]

Nmap builds a **24-byte TCP header** (20 base + 4 options).

![Firewall Evasion-5.png|450](/img/user/cards/red-team/images/Firewall%20Evasion-5.png)

The 24 bytes of the TCP header will be divided across 3 IP packets, another option is limiting the IP data to 16 bytes by adding another: `-ff`

- Fragment data payload = **16 bytes** (because of `-ff`).

- IP header = **20 bytes** (always added to each fragment).
    
- So each fragment’s total size = `20 (IP header) + 16 (data)` = **36 bytes**

![Firewall Evasion'.png|450](/img/user/cards/red-team/images/Firewall%20Evasion'.png)

 Another option is `--mtu SIZE` to provide a custom size for data carried within the IP packet. The size should be a multiple of 8. Setting the value to `--mtu 8` is equivalent of `-f` flag.
## Evasion via Modifying Header Fields
---
Nmap gives you further control over the different fields in the IP header
### Set TTL
---
Nmap gives you further control over the different fields in the IP header:

_Time to Live_, is a hop count limit meaning it represent the maximum number of routers a packet can traverse IN IP PACKET settings but _ttl_ on DNS represents actual seconds.

```C
nmap -sS -Pn --ttl 81 -F 10.201.89.111
```

Why do this?

- Different Operating systems use different default TTLs: Linux starts with 64, Windows with 128, and Cisco with 255. A defender can compare your packets’ TTLs to known defaults and guess your OS.
- Some firewalls or intrusion detection systems are placed **off-path** (not in the exact same route as the target). If you set TTL so low that your packet reaches the **target** but dies before hitting a monitoring sensor,
### Set IP Options
---
In the IPv4 header, there’s a rarely used field called **IP Options** that lets you add instruction to routers and destinations, normally, most packets dont use it so the headers stays 20 bytes.

```C
--ip-options
```

- **`R` -> Record-Route**
    
    - Routers along the path record their IP addresses in the packet.
    - Lets you see the path your packet took (like a traceroute built into the packet).
        
- **`T` -> Record-Timestamp**
    
    - Routers record their timestamp when forwarding the packet.
    - Helps measure latency or identify routing delays.
        
- **`U` -> Record-Route + Timestamp**
    
    - Combines both of the above.
        
- **`L` -> Loose Source Routing**
    
    - You specify a **list of IP addresses** the packet should go through, _but it can also take other hops in between_.
    - Example: force the packet to pass through some intermediate node, but allow flexibility.
        
- **`S` -> Strict Source Routing**
    
    - You specify **exactly which hops** the packet must take — no deviations.
    - Routers must follow that exact path, or drop the packet.

Why do this? If you know a firewall sits at hop 3, but you can find another route that bypasses it, you might use source routing to force Nmap packets that way. **HOWEVER** modern network source routing is always blocked.
### Use a Wrong Checksum
---
Send your packets with an intentionally wrong checksum. Some systems would drop a packet with a bad checksum, while others won’t.

```C
nmap -sS -Pn --badsum -F 10.201.89.111
```

But why would you do this sir? mainly for **detection**.

- Some firewalls or intrusion detection systems don’t validate checksums properly, they might forward your bad packet to the target, even though the target itself will drop them. **In this way you can learn if there is a filtering device in between.**
- If you get **any response** to a packet with a bad checksum, it didn't come from the real host, it came from some intermediate device such as firewall, IDS, or load balancer.
## Evasion Using Port Hopping
---
Port hopping is when a client and server **change the port number used for communication** instead of sticking to a single fixed port:

1. **At the start:** the client tries different ports until it finds one that isn't blocked.
2. **During the session:** the client and server deliberately switch to new ports while already exchanging data.

Why do this?

- **Firewall evasion:** some firewall block common ports.
- **Tracking evasion:** if defenders started monitoring for port 8080, but the app switches to random port, it becomes harder to follow the same session.

![Firewall Evasion-6.png|550](/img/user/cards/red-team/images/Firewall%20Evasion-6.png)
## Evasion via Port Tunneling
---
Port tunneling is also known as _port forwarding_ and _port mapping_. basically it forwards the packets sent to one destination port to another destination port.

For example:

We have an SMTP server listening on port 25; however, we cannot connect to the SMTP server because the firewall blocks packets from the Internet sent to destination port 25. We discover that packets sent to destination port 443 are not blocked, so we decide to take advantage of this and send our packets to port 443, and after they pass through the firewall, we forward them to port 25

![Firewall Evasion-7.png|500](/img/user/cards/red-team/images/Firewall%20Evasion-7.png)

Obviously, we need the server that host 443 (web server) compromised or can take whatever command we want.

> [!question]- We have a web server listening on the HTTP port, 80. The firewall is blocking traffic to port 80 from the untrusted network; however, we have discovered that traffic to TCP port 8008 is not blocked.
> 
> `ncat -l -p 8008 -c "ncat 127.0.0.1 80"`
> This assume the above command is executed in a compromised machine basically everytime we connect to port 8008 as attacker, the `ncat` command executes or in this case connects to whatever services is at port 80 effectively tunneling the traffic at port 8008.
## Evasion using Non-Standard Port 
---
Basically choose a port number that the firewall does not block, port number above 1024 as we cannot choose lower than 1024 requires `sudo` or `root` privileges.


## All said and done NGFW kills them all
---
Next-Generation Firewall (NGFW) is designed to handle the new challenges facing modern enterprises. For instance, some of NGFW capabilities include:

- Integrate a firewall and a real-time Intrusion Prevention System (IPS). It can stop any detected threat in real-time.
- Identify users and their traffic. It can enforce the security policy per-user or per-group basis.
- Identify the applications and protocols regardless of the port number being used.
- Identify the content being transmitted. It can enforce the security policy in case any violating content is detected.
- Ability to decrypt SSL/TLS and SSH traffic. For instance, it restricts evasive techniques built around encryption to transfer malicious files.

Making most of the attacks here useless.

### Questions and Problems
---
> What if I performed an `nmap` scan with `RND` or decoys flagged enabled against a network in the Internet, will Nmap hijack a real public IP address?

Nope, anti-spoofing (BCP 38 filtering) implemented by your ISPs and backbone routers drops any packet with a source IP that **doesn't belong to your network.** Your spoofed decoys most likely will never leave your network.

_So this is good from a red-teaming point of view?_

- Yes, but to a skilled analyst this should be easy, since those decoys does not receive or benefit from the replies of the target.
- **Volume of traffic:** decoys usually send fewer packets or slightly different patterns compared to your real host, the real scanner tends to have a more consistent packet stream.
- **TCP Handshake or SYN/ACK:** the real scanner completes the TCP handshake or at least gets a SYN/ACK flag.



## Conclusion


