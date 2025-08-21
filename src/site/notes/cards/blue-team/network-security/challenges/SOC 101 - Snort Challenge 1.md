---
{"dg-publish":true,"permalink":"/cards/blue-team/network-security/challenges/soc-101-snort-challenge-1/"}
---

[[cards/blue-team/network-security/Snort\|Snort]] | ~ [[map-of-contents/blue-team\|blue-team]]

> First, investigate `snort_challenge.pcap` in **Wireshark** and identify any unique **User-Agent** strings. Upon doing so, what penetration testing tool did the attacker use to brute force the admin page of the web server?

Since we are dealing with `User-Agent` and it can be found only on HTTP GET request, we can use the filter below:

```
http.request.method == "GET
```

Output:

![SOC 101 - Snort Challenge 1.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Snort%20Challenge%201.png)
> The login page on the web server responds with an `HTTP 401` status code for failed login attempts. Create a Snort rule to detect if **10 failed login attempts** (HTTP 401 response codes) occur **within a 30-second period** from the **same IP address**. How many alerts were logged?

```bash
 alert tcp any 80 -> any any ( msg:"10 Failed Login Attempts within 30 seconds possible brute force detected"; content:"401"; http_stat_code; flow:to_client,established; threshold:type threshold, track by_src, count 10 , seconds 30; sid:10000001; rev:1; ) 
```

**174**

> When a login is successful, it results in an `HTTP 302` redirect to the admin portal. Create a Snort rule to detect any successful logins. What is the **UNIX epoch timestamp** of the attacker's first successful login?

```
alert tcp any 80 -> any any ( msg:"Succesful admin login"; content:"302"; http_stat_code; flow:to_client,established; sid:10000001; rev:1; )
```

Using tcpdump to convert to unix epoch time:

```
tcpdump -r f.pcap -tt
```

**1717795241.242056**

> A common LFI payload is to include the string `../` within URL parameters to attempt to traverse directories and access restricted files or directories outside of the web root. Create a Snort rule to detect any packets **with this string** in the **Request URI**. How many detections or alerts were logged?

**5**

 ```
alert tcp any any -> any 80 ( msg:"Local File Inclusion Payload found on URL"; content:"|2e 2e 2f|"; http_uri; sid:10000001; rev:1; ) 
```

> By abusing the Local File Inclusion (LFI) vulnerability on the admin panel, the attacker stole a private SSH key from the web server. Given the known **file signature** of an **OpenSSH private key file**, create a Snort rule to detect any instances of these private keys traversing the network. Apply this rule to analyze the provided PCAP file. What was the **Content-Length** of the HTTP response containing the private key file?

![SOC 101 - Snort Challenge 1-1.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Snort%20Challenge%201-1.png)

35 for depth

 ```bash
alert tcp any any -> any any ( msg:"Usage of Private Keys Detected"; file_data; content:"|2d 2d 2d 2d 2d|BEGIN|20|OPENSSH|20|PRIVATE|20|KEY|2d 2d 2d 2d 2d|"; depth: 35; sid:10000001; rev:1; ) 
```

> After obtaining the private key, the attacker gained access to the web server using **SSH** and started exfiltrating sensitive data using **FTP**. Create a Snort rule to detect any **outgoing connections** to an **external FTP server outside of the `192.168.1.0/24` network range**. What is the **Autonomous System Number (ASN)** of the FTP server the attacker connected to?

```
alert tcp 192.168.1.0/24 any -> any 21 ( msg:"Outbound FTP connection detected"; sid:10000001; rev:1; ) 
```

whois: 13036

![SOC 101 - Snort Challenge 1-2.png](/img/user/cards/blue-team/network-security/images/SOC%20101%20-%20Snort%20Challenge%201-2.png)