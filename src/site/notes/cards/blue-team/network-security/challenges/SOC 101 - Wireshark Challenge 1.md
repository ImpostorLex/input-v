---
{"dg-publish":true,"permalink":"/cards/blue-team/network-security/challenges/soc-101-wireshark-challenge-1/"}
---

~ [[map-of-contents/blue-team\|blue-team]] 

Here is the metadata of this `.pcap` file found at statistic's file properties:

![SOC 101 - Wireshark Challenge 1.png|450](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201.png)
At statistic's protocol hierarchy, the bulk of the packets comes from http and smb:

![SOC 101 - Wireshark Challenge 1-1.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-1.png)
Here are the top talkers in our network and most likely our endpoint is **172.16.1.191**

![SOC 101 - Wireshark Challenge 1-2.png|450](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-2.png)
Immediately our endpoint made a DNS query to **webmasterdev.com** associated IP address is 184.168.98.68:

![SOC 101 - Wireshark Challenge 1-3.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-3.png)
Using the below display filter as this is our endpoint, I see which most likely a web request to the third top talker **'162.252.172.54'** using powershell:

```bash
http and ip.addr == 172.16.1.191
```

Output:

![SOC 101 - Wireshark Challenge 1-4.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-4.png)
Then following the HTTP stream, the user made a GET request to download a `.exe` however looking at the `content-type` header, it says that it claims to be a **image/gif**:

![SOC 101 - Wireshark Challenge 1-5.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-5.png)
Additionally **23.163.0.37** sends back or pushed some data back to our endpoint:

![SOC 101 - Wireshark Challenge 1-6.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-6.png)
### Investigating the downloaded executable
---
The file we are interested in is **6ctf5JL**, however wireshark detected other objects that can be exported:

![SOC 101 - Wireshark Challenge 1-7.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-7.png)
Using `file` on the executable:

![SOC 101 - Wireshark Challenge 1-8.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-8.png)
Retrieving sha256sum of the `.dll` and uploading it to VirusTotal, as well as checking the reputation of the host who serves this file:

```
9b8ffdc8ba2b2caa485cca56a82b2dcbd251f65fb30bc88f0ac3da6704e4d3c6  6ctf5JL
```

The host flagged as malicious and tagged as malware:

![SOC 101 - Wireshark Challenge 1-9.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-9.png)
As for the file itself:

![SOC 101 - Wireshark Challenge 1-10.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-10.png)
VirusTotal flagged it as **trojan.pikabot/mikey** and now looking for reports about pikabot Indicator of Compromise (IOCs) which is the popular threat label assigned to it:

One of the reports says that it make uses of SMB to propagate across the network which matches with our packet capture:

![SOC 101 - Wireshark Challenge 1-11.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-11.png)
Then it makes network connection to their C2 server using non-traditional ports such as 2221 and 2078 according to the report.

- Other reports include port 1194.
## Investigating Ports
---
**Note:** with how cloud computing works, it is easy to change IP address so VirusTotal may update this for future report.

```bash
tcp.port == 2078
```

Output:

![SOC 101 - Wireshark Challenge 1-12.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-12.png)
Unfortunately connection is encrypted as can be seen from the image they iniated a TLS handshake but VirusTotal flagged '45.85.235.39' as malicious:

![SOC 101 - Wireshark Challenge 1-13.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-13.png)

```
tcp.port == 1194
```

Unfortunately it is the same situation for port 2078 and got flagged by VirusTotal as well:

![SOC 101 - Wireshark Challenge 1-14.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-14.png)
## Investigating the second top conversation
---

Flagged by VirusTotal:

![SOC 101 - Wireshark Challenge 1-15.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-15.png)There is no way to rebuild this data being pushed to our endpoint:

![SOC 101 - Wireshark Challenge 1-16.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-16.png)
And then this IP address made a lot of DNS queries, all of it's packet are related to DNS:

```C
ip.addr == 172.16.1.16
```

![SOC 101 - Wireshark Challenge 1-17.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-17.png)
Here is where it gets suspicious:

![SOC 101 - Wireshark Challenge 1-18.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-18.png)
My assumption is that it is exfiltrating data, let's confirm this by looking up the domain:

```
steasteel.net
```

![SOC 101 - Wireshark Challenge 1-19.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-19.png)

```
dns.steasteel.net
```

![SOC 101 - Wireshark Challenge 1-20.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Wireshark%20Challenge%201-20.png)