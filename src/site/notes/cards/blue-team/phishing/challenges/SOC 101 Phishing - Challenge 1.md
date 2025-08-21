---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/challenges/soc-101-phishing-challenge-1/"}
---

~ [[map-of-contents/blue-team\|blue-team]] | [[cards/blue-team/phishing/challenges/SOC 101 Phishing - Challenge 1#Unformatted Report (Initial Investigation)\|Jump to unformatted investigation]]
Headers
======================================
Date: Tue, 31 Oct 2023 10:10:04 -0900
Subject: Your account has been flagged for unusual activity

To: dderringer[@]mighty-solutions.net
From: social201511138[@]social.helwan.edu.eg

Reply-To:
Return-Path: social201511138[@]social.helwan.edu.eg

Sender IP: 40.107.22.60
Resolve Host: mail-am6eur05on2060.outbound.protection.outlook.com 

![SOC 101 Phishing - Challenge 1-2.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-2.png)

Message-ID: <JMrByPl2c3HBo8SctKnJ5C5Gp64sPSSWk76p4sjQ@s6>

URLs
=======================================
hxxps[://]0[.]232[.]205[.]92[.]host[.]secureserver[.]net/lclbluewin08812/
Description
======================================
The email claims that Microsoft Outlook Team flagged the receiver's account due to unusual activity and if left ignored for 24 hours the account will be suspended however the sender's email address did not match with Microsoft's Outlook domain.
Artifact Analysis
======================================
Sender Analysis:

![SOC 101 - Challenge 1.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Challenge%201.png)
The email states that the outlook team flagged this account for unusual activity and if left ignored for 24 hours, the account will be suspended.

The email is claiming to be from Microsoft Outlook Support Team however the sender's email says otherwise: **social201511138[@]social.helwan[.]edu[.]eg**.

It also includes a pre-pended message claiming that the email is sent by a trusted sender certified by the Outlook Online Support Team however the `from` email includes an `.edu` top level domain which should be from outlook.

![SOC 101 Phishing - Challenge 1.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201.png)
Resolving the `Return-Path` domain shows using [whois.domaintools](whois.domaintools.com):

![SOC 101 Phishing - Challenge 1-3.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-3.png)
My initial assumption is that `helwan.edu` domain make use of Microsoft Outlook as their email provider and one of it's users took advantage of a legitimate service.
#### Authenticating Email
---
Performing reverse lookup on the `return-path` shows:

![SOC 101 Phishing - Challenge 1-3.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-3.png)

```
dig TXT social.helwan.edu.eg | grep -i "spf"
```

Output:

![SOC 101 Phishing - Challenge 1-4.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-4.png)
Based on the given information: the `helwan.edu` email provider is Microsoft Outlook.

**Analyzing DKIM**

Using the information found on `DKIM-Signature` header:

```
nslookup -type=txt selector1-socialhelwanedu-onmicrosoft-com._domainkey.socialhelwanedu.onmicrosoft.com
```

Output:

![SOC 101 Phishing - Challenge 1-5.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-5.png)
Checking the authentication results shows that both `SPF` and `DKIM` passed:

![SOC 101 Phishing - Challenge 1-6.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-6.png)

URL Analysis:

#### Content Examination
---
The `Content-Type` is 'text/html' and the `Content-Transfer-Encoding` is 'base64':

![SOC 101 Phishing - Challenge 1-7.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-7.png)
This should raise suspicion as base64 is commonly used to bypass weak email filters and considering that the email sender's domain is not outlook.

![SOC 101 Phishing - Challenge 1-9.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-9.png)

Hovering to the link shows a non-microsoft outlook URL.
![SOC 101 Phishing - Challenge 1-8.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-8.png)
- Sender's domain does not match with Microsoft's outlook domain.
- Generic customer greeting **'Dear Customer'**
- Sense of Urgency: **'Account Suspension'** and **'Unusualy Activity'** is used.

## URL Analysis
---
Copying the entire base64 encoded content to cyberchef and decoding it with base64 to extract the URL:

![SOC 101 Phishing - Challenge 1-10.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-10.png)
Defanged: 

```
hxxps[://]0[.]232[.]205[.]92[.]host[.]secureserver[.]net/lclbluewin08812/
```

Using [urlscan.io](urlscan.io) against the URL did not flagged this as malicious and the website does not exist anymore:

It also worth mentioning that the email sender address resolves to egypt but the URL provided is located in Germany.

![SOC 101 Phishing - Challenge 1-11.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-11.png)
Using [VirusTotal](virustotal.com) against the domain flagged the site as phishing:

![SOC 101 Phishing - Challenge 1-12.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-12.png)
In conclusion, It is safe to say that this email is indeed malicious and it make use of legitimate services such as Outlook.com and even using a `.edu` email address.
Verdict
======================================
The email came from a legitimate service Microsoft Outlook and even used a `.edu` address which is commonly used for education however the email claims that they are the Outlook support team with a mismatching sender's email address and a non Microsoft Outlook URL. 

hxxps[://]0[.]232[.]205[.]92[.]host[.]secureserver[.]net/lclbluewin08812/

Analyzing the provided URL using [urlscan.io](urlscan.io) shows that it is not malicious however the screenshot of the web page reveals that the page does not exist anymore:

![SOC 101 Phishing - Challenge 1-11.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-11.png)
VirusTotal flagged the website as phishing:

![SOC 101 Phishing - Challenge 1-12.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-12.png)

Defense Actions
======================================
- Since the attacker make uses of legitimate services such as GoDaddy and Microsoft Outlook to perform it's attack even using a `.edu` address blocking the email service provider and the sender's email address is not recommended, the SENDER exact email address should be blocked using email gateway.
- Since the URL is highly unlikely will be visited by any employee in the organization, the domain or URL can be blocked using EDR, Firewall and Web Proxy.

## Unformatted Report (Initial Investigation)
----
Date: 
![SOC 101 - Challenge 1.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Challenge%201.png)
The email states that the outlook team flagged this account for unusual activity and if left ignored for 24 hours, the account will be suspended.

The email is claiming to be from Microsoft Outlook Support Team however the sender's email says otherwise: **social201511138[@]social.helwan[.]edu[.]eg**.

It also includes a pre-pended message claiming that the email is sent by a trusted sender certified by the Outlook Online Support Team however the `from` email includes an `.edu` top level domain which should be from outlook.
### Analyzing Headers
---
The `return-path` matches with the `from` header, it also includes a `X-Sender-IP` address:

![SOC 101 Phishing - Challenge 1.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201.png)
Resolving the IP address found on `X-Sender-IP` using `whois.domaintools.com`:

![SOC 101 Phishing - Challenge 1-2.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-2.png)
My initial assumption is that `helwan.edu` domain make use of Microsoft Outlook but again Microsoft Outlook Team will not use this domain.
#### Authenticating Email
---
Performing reverse lookup on the `return-path` shows:

![SOC 101 Phishing - Challenge 1-3.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-3.png)

```
dig TXT social.helwan.edu.eg | grep -i "spf"
```

Output:

![SOC 101 Phishing - Challenge 1-4.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-4.png)
Based on the given information: the `helwan.edu` email provider is Microsoft Outlook.

**Analyzing DKIM**

Using the information found on `DKIM-Signature` header:

```
nslookup -type=txt selector1-socialhelwanedu-onmicrosoft-com._domainkey.socialhelwanedu.onmicrosoft.com
```

Output:

![SOC 101 Phishing - Challenge 1-5.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-5.png)
Checking the authentication results:

![SOC 101 Phishing - Challenge 1-6.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-6.png)
## Content Examination
---
The `Content-Type` is 'text/html' and the `Content-Transfer-Encoding` is 'base64':

![SOC 101 Phishing - Challenge 1-7.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-7.png)
This should raise suspicion as base64 is commonly used to bypass weak email filters and considering that the email sender's domain is not outlook.

![SOC 101 Phishing - Challenge 1-9.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-9.png)

Hovering to the link shows a non-microsoft outlook URL.
![SOC 101 Phishing - Challenge 1-8.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-8.png)
- Sender's domain does not match with Microsoft's outlook domain.
- Generic customer greeting **'Dear Customer'**
- Sense of Urgency: **'Account Suspension'** and **'Unusualy Activity'** is used.
## URL Analysis
---
Copying the entire base64 encoded content to cyberchef and decoding it with base64 to extract the URL:

![SOC 101 Phishing - Challenge 1-10.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-10.png)
Defanged: 

```
hxxps[://]0[.]232[.]205[.]92[.]host[.]secureserver[.]net/lclbluewin08812/
```

Using [urlscan.io](urlscan.io) against the URL did not flagged this as malicious and the website is not found anymore:

![SOC 101 Phishing - Challenge 1-11.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-11.png)
Using [VirusTotal](virustotal.com) against the domain flagged the site as phishing:

![SOC 101 Phishing - Challenge 1-12.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20Phishing%20-%20Challenge%201-12.png)
In conclusion, It is safe to say that this email is indeed malicious and it make use of legitimate services such as Outlook.com and even using a `.edu` email address.



