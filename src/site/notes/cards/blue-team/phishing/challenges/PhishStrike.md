---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/challenges/phish-strike/"}
---

~ [[atlas/blue-team\|blue-team]]
### Initial Triage
---
At first glance, it looks legitimate however hovering the links shows an IP address:
107.175.247.199/loader/install.exe which is raises red flag as receipt usually points out to a printable page not an executable.
![PhishStrike.png](/img/user/cards/blue-team/phishing/images/PhishStrike.png)
As for the sender's email address, it matches with the **Uptc** official email address format.

The only concern for the receiver is if he/she does not remember purchasing at all or worse an account has been hijacked.
## Header Analysis
---
Date:Thu, 9 Dec 2022 09:58:26 +0100

From: ERIKA JOHANA LOPEZ VALIENTE erikajohana.lopez[@]uptc.edu.co

Message-ID:
```C
<CABWu4iua5_uex6=G8pi_OJz1tBLJiNakMK-1=7128orpzxbKxw@mail.gmail.com>
```

To: is undisclosed or redacted for confidentiality.

Subject: COMMERCIAL PURCHASE RECEIPT ONLINE 27 NOV

Return-Path: erikajohana.lopez[@]uptc.edu.co

Received: by mail-wr1-f65.google.com with SMTP id 

The **subject** and the **email sending date** does not much however it is understandable as purchases receipts may come late depending on the business system, as for the **Message-ID** and **Received** header, it shows that the sender uses Google's gmail to send the email.

Performing a look up on the university's domain: uptc.edu.co using: [whois.domaintools.com/uptc.edu.co](https://whois.domaintools.com/uptc.edu.co), matches with the IP location of university's (general) location: Boyaca Tunja is located in columbia.

![PhishStrike-1.png|450](/img/user/cards/blue-team/phishing/images/PhishStrike-1.png)
With this the IP address found on the link which is 107.175.247.199 does not match with 132.255.20.10 at all.

Verifying with another checks and determining `spf` and `dkim` record:
```
dig TXT uptc.edu.co | grep -i "spf"
```

Output:

![PhishStrike-2.png](/img/user/cards/blue-team/phishing/images/PhishStrike-2.png)
It outputs **google.com** which means that it handles both `spf` and `dkim` checks for this domain, **note** the `whois` lookup versus the `dig` command shows different IP address, most likely the **.20.10** is the registrar or the DNS hosting services versus the **20.20** and **20.21** is most likely the mail server.

**Analyzing DKIM**
We need the following information: the selector (**s**) and the domain (**d**):

![PhishStrike-3.png](/img/user/cards/blue-team/phishing/images/PhishStrike-3.png)
Then inputting it to:

[mxtoolbox.com](https://mxtoolbox.com/SuperTool.aspx?action=dkim%3auptc.edu.co%3agoogle&run=toolpage)

![PhishStrike-4.png](/img/user/cards/blue-team/phishing/images/PhishStrike-4.png)
However performing a commandline version of the same purpose:

```C
nslookup -type=txt google._domainkey.uptc.edu.co
```

Then:

![PhishStrike-5.png](/img/user/cards/blue-team/phishing/images/PhishStrike-5.png)
At the **Authentication Results** header says otherwise:

![PhishStrike-6.png](/img/user/cards/blue-team/phishing/images/PhishStrike-6.png)
The `spf=softfail` the email is suspicious and mark it as spam while `dkim=fail` means that the email's integrity is tampered with, cross verifying with another tool:

https://mxtoolbox.com/Public/Tools/EmailHeaders.aspx?huid=aa9e41a3-e9a0-47cd-acee-a8de4b48b61c

![PhishStrike-7.png|450](/img/user/cards/blue-team/phishing/images/PhishStrike-7.png)
Definetly a suspicious email.

Also performing a lookup on the **Sender IP address** found on the image above ( 18.208.22.104):

![PhishStrike-10.png|450](/img/user/cards/blue-team/phishing/images/PhishStrike-10.png)
Ironic enough `tmes.trendmicro.com` is 

> is an enterprise-class solution that delivers continuously updated protection to stop phishing, ransomware, Business Email Compromise (BEC) scams, spam and other advanced email threats before they reach your network.

## Web and URL Analysis
---
Pasting 107.175.247.199 to VirusTotal flags it with a score of 6/94:

![PhishStrike-8.png](/img/user/cards/blue-team/phishing/images/PhishStrike-8.png)
Using tools such as `urlscan.io` to view the website results into unavailable as most likely the website is taken down:

![PhishStrike-9.png](/img/user/cards/blue-team/phishing/images/PhishStrike-9.png)
**Submitting the URL to urlhaus**

![PhishStrike-11.png](/img/user/cards/blue-team/phishing/images/PhishStrike-11.png)
Then clicking the `Bazaar's Coinminer` icon to determine it's behavior:

![PhishStrike-12.png](/img/user/cards/blue-team/phishing/images/PhishStrike-12.png)
Selecting `anyrun.io` shows contacted IP addresses:

![PhishStrike-13.png](/img/user/cards/blue-team/phishing/images/PhishStrike-13.png)


**Verdict**

A lot of red flags, as the email authentication checks failed or not compliant, plus the email containing links does not match with the description mask provided such as the link "Erika Johana Lopez Valiente" should lead to his/her profile but instead redirects to a downloadable executable on a domain nameless link making this email **malicious**.

**Reacting Defense**
Blocking the sender's email temporarily is a good option assuming the email is legitimate but got hijacked (or hacked.) in the email gateway then blocking the malicious link (107.175.247.199) is safe to block indefinetly, it can be blocked within the Firewall.

**Proactive Defense**
If the receiver's normally does not expect `.exe`,  it is safe to block all file attachments or `.exe` downloaded from the web to be blocked.

**Organize and refine notes after each session including adding meta-tags**

57 mins 
1 hour 34 mins total missed and answered questions.