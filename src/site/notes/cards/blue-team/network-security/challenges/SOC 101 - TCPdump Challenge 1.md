---
{"dg-publish":true,"permalink":"/cards/blue-team/network-security/challenges/soc-101-tc-pdump-challenge-1/"}
---

~ [[map-of-contents/blue-team\|blue-team]]

Displaying the packet without filters shows interesting packets, these are accessing a file share using ftp and the use of ping:

```bash
sudo tcpdump -tt -r challenge1.pcap | less
```

- `--count` 1344 packets.

Output:

![SOC 101 - TCPdump Challenge 1.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20TCPdump%20Challenge%201.png)
Immediately I see some ftp credentials as well as anonymous login which is bad, as anyone in the network can access the file share:

![SOC 101 - TCPdump Challenge 1-1.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20TCPdump%20Challenge%201-1.png)
Filtering for ftp:

```bash
sudo tcpdump -tt -r tcpdump_challenge.pcap -n port 21
```

It shows the software used to setup a file share and it's created using `.NET` and as well as the user trying to login to ftp as admin with an IP address of 194.108.117.16:

![SOC 101 - TCPdump Challenge 1-2.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20TCPdump%20Challenge%201-2.png)
A quick google search shows the rebex pack:

![SOC 101 - TCPdump Challenge 1-3.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20TCPdump%20Challenge%201-3.png)
Scrolling down to, we can see a user retrieves a file `readme.txt`:

![SOC 101 - TCPdump Challenge 1-4.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20TCPdump%20Challenge%201-4.png)
However this is not enough for the investigation, so decide to check the t**op conversation:**

```bash
sudo tcpdump -tt -r tcpdump_challenge.pcap -n tcp | cut -d " " -f 3 | cut -d "." -f 1-4  | sort | uniq -c | sort -nr
```

Output:

![SOC 101 - TCPdump Challenge 1-5.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20TCPdump%20Challenge%201-5.png)
Filtering for the second top communicator:

```
sudo tcpdump -tt -r tcpdump_challenge.pcap -n host 184.84.32.184
```

Shows that **10.0.2.10** made a GET request to the IP address in question:

![SOC 101 - TCPdump Challenge 1-6.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20TCPdump%20Challenge%201-6.png)
Perform IP reverse lookup on the host shows:

![SOC 101 - TCPdump Challenge 1-7.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20TCPdump%20Challenge%201-7.png)
It is a cybersecurity and cloud computing company.

Investigating the pings, at a glance shows the same IP address:

```C
sudo tcpdump -tt -r tcpdump_challenge.pcap -n icmp
```

Output:

![SOC 101 - TCPdump Challenge 1-8.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20TCPdump%20Challenge%201-8.png)
Looking up the IP address:

![SOC 101 - TCPdump Challenge 1-9.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20TCPdump%20Challenge%201-9.png)
POST request found:

```
sudo tcpdump -tt -r tcpdump_challenge.pcap -n port 80 | grep -i "POST" | sort -r | uniq -c | sort -f
```

![SOC 101 - TCPdump Challenge 1-10.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20TCPdump%20Challenge%201-10.png)
```
sudo tcpdump -tt -r tcpdump_challenge.pcap -n port 80 -A | grep -i "user\|password\|login" | grep -vE "User-Agent|huffpost.com"
```




