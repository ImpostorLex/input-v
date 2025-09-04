---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/challenges/soc-101-phishing-challenge-2/"}
---

~ [[atlas/blue-team\|blue-team]] 
## Initial Triage
---
The email seems legitimate as the email claims that they are from dropbox and the email is sent because the user or in this case receiver: emily.nguyen[@]glbllogistics.co requested a password reset.

![SOC 101 - Phishing Challenge 2.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202.png)
Hovering the reset password button shows a URL with a base domain of dropbox.com, which is a good sign and the sender address no-reply[@]dropbox.com is a no-reply which is use to send only emails and not receive any replies from the user.
## Header Analysis
---
Message-ID:
```
<0101018f6aff12b2-5bcaa145-861b-45da-a06e-b5c1ee3ca941-000000@us-west-2.amazonses.com>
```

Date: Sun, 12 May 2024 04:10:52 +0000

Subject: Reset your Dropbox password

![SOC 101 - Phishing Challenge 2-1.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202-1.png)
Looking up the domain found on `return-path` header using [whois.domaintools.com](whois.domaintools.com) matches with dropbox:

![SOC 101 - Phishing Challenge 2-2.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202-2.png)
The domain found on `Message-ID` header matches with Amazon Simple Email Services using the same tool [whois.domaintools.com/amazonses.com](https://whois.domaintools.com/amazonses.com):

![SOC 101 - Phishing Challenge 2-3.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202-3.png)
### Authenticating Email
---
Both DKIM and SPF passed:

![SOC 101 - Phishing Challenge 2-4.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202-4.png)
```
dig TXT email.dropbox.com | grep -i "spf"
```

Output:
![SOC 101 - Phishing Challenge 2-5.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202-5.png)
Looking up the SPF record of amazonses.com and comparing with the IP address found on a `received` header:

![SOC 101 - Phishing Challenge 2-6.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202-6.png)
Command:

```
dig TXT amazonses.com | grep -i "spf"
```

Output:

The result confirms that the email server's IP address found on the `received` header is permitted on sending email on behalf of dropbox.com

![SOC 101 - Phishing Challenge 2-7.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202-7.png)
Additionally this is the resolved address of 54.240.60.143:

![SOC 101 - Phishing Challenge 2-13.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202-13.png)
## Content Analysis
---
Based on the initial triage, there is not really any social engineering flags found on this email.
### URL Analysis
---
```
hxxps[://]www[.]dropbox[.]com/l/ABCIzswwTTJ9--CxR05fYXX35pPA-Y0m3PY/forgot_f=
inish

%% Decoded Quoted Printable > Defanged version %%
hxxps[://]www[.]dropbox[.]com/l/ABCIzswwTTJ9--CxR05fYXX35pPA-Y0m3PY/forgot_finish
```

Using [urlscan.io](urlscan.io) against the link, did not flagged as malicious and the web page is not available anymore:

![SOC 101 - Phishing Challenge 2-8.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202-8.png)
But using VirusTotal against the same link, one of the security vendors flagged it as malicious:

![SOC 101 - Phishing Challenge 2-9.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202-9.png)
Getting a second opinion using Cisco Talos shows nothing:

Web reputation results into **favorable = Displaying behavior that indicates a level of safety** per documentation.

![SOC 101 - Phishing Challenge 2-10.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202-10.png)
Second Link:

```
hxxps[://]www[.]dropbox[.]com/l/AADQ493u2QLcZrv2kCW6C6i0Ac-mR0hUXxU/help/365
```

No flags raised by VirusTotal and urlscan.io:
![SOC 101 - Phishing Challenge 2-11.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202-11.png)
urlscan.io:

![SOC 101 - Phishing Challenge 2-12.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%202-12.png)
Verdict not malicious.

## Ask Questions
---
> Should I still analyze URLs that the user will not click or find?