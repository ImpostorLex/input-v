---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/challenges/soc-101-phishing-challenge-3/"}
---

~ [[map-of-contents/blue-team\|blue-team]] 
## Initial Triage
---
The email claims that Emily (our user) is invited by Adam (sender) to their wedding and he is asking emily to fill up the attached survey.

![SOC 101 - Phishing Challenge 3.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%203.png)
However looking at the last line of the email, it claims to be from **Alexia** not **Adam** and looking at the attached email has a `.docm` extension which means the document is macro enabled, this alone should raise a flag.
## Header Analysis
---
Viewing the source `.eml` file in sublime text:

Sender: abarry[@]live.com
Receiver: emily.nguyen[@]glbllogistics.co

Message ID:

```
<SA1PR14MB737384979FDD1178FD956584C1E32@SA1PR14MB7373.namprd14.prod.outlook.com>
```

Subject: You're Invited!

Date: Tue, 14 May 2024 23:31:08 +0000

Return-Path: abarry[@]live.com
### Authentication Checks
---

![SOC 101 - Phishing Challenge 3-1.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%203-1.png)
Performing a lookup on the domain found on `return-path` header [whois.domaintools.com/live.com](https://whois.domaintools.com/live.com) using `whois`:

![SOC 101 - Phishing Challenge 3-2.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%203-2.png)
**Confirming SPF**:

```
dig TXT live.com | grep -i "spf"
```

Output:

![SOC 101 - Phishing Challenge 3-3.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%203-3.png)
Based on the `authentication results` header shows that both `spf` and `dkim` passed the authentication checks as they make use of a legitimate services.
## Content Examination
---
Again, from the initial triage the sender's resolved email name is Adam Berry however in the email content it claims to be from Alexia Berry and there is a suspicious `.docm` file attached which is a macro-enabled document.

Checking the source code: the file is encoded in base64 and has a filename of `AR_Wedding_RSVP.docm` however at the `Content-Description` headers says it's **example.txt**:

![SOC 101 - Phishing Challenge 3-4.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%203-4.png)
## Attachment Analysis
---
I am going to use `emldump.py` to extract the file but I must first identify it's stream number:

```
python3 ../Tools/emldump.py challenge3.eml
```

Output shows that we are looking for stream number 5:

![SOC 101 - Phishing Challenge 3-5.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%203-5.png)
Extracting the file:

```bash
python3 ../Tools/emldump.py challenge3.eml -s 5 -d > AR_Wedding_RSVP.docm
```

Output:

![SOC 101 - Phishing Challenge 3-6.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%203-6.png)
Getting sha256 hash of the file and looking up using VirusTotal:

```
41c3dd4e9f794d53c212398891931760de469321e4c5d04be719d5485ed8f53e AR_Wedding_RSVP.docm
```

Output: VirusTotal flaged this as malicious with a populat threat label of 'downloader.autdwnlrner/w97m':

![SOC 101 - Phishing Challenge 3-7.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%203-7.png)
Interestingly, VirusTotal inspects the MACRO and explains it to us:

![SOC 101 - Phishing Challenge 3-8.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%203-8.png)
Getting the source code with `oledump.py`:

```
python3 ../Tools/oledump.py AR_Wedding_RSVP.docm
```

Output, we are interested in Data Stream 3:

![SOC 101 - Phishing Challenge 3-9.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%203-9.png)
Viewing the actual macros:

```
python3 ../Tools/oledump.py AR_Wedding_RSVP.docm -s 3 --vbadecompresscorrupt
```

Output, this matches with VirusTotal report:

![SOC 101 - Phishing Challenge 3-10.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%203-10.png)
### Static Analysis
---

- `Set http_obj = CreateObject("Microsoft.XMLHTTP")`
	- Creates a HTTP object used for making GET and POST request, in this case getting a file from github named **Pastebin-uploader.exe**.
- `Set stream_obj = CreateObject("ADODB.Stream")`
	- This object represents a stream of data from a URL or any type of record usually a file
	- In this case it saves to a filename `shost.exe`
- `Set shell_obj = CreateObject("WScript.Shell")`
	- Provides access to Windows Operating System PowerShell commands and in this case executing the `shost.exe`.
## Dynamic Analysis 
---
Since the macro is straightforward, I am going to use hybrid-analysis on the `.exe` found on github:

```
hxxps[://]github[.]com/TCWUS/Pastebin-Uploader[.]exe
```

It came clean most likely because of github a legitimate services and commonly used to host code, exe and more:

![SOC 101 - Phishing Challenge 3-12.png](/img/user/cards/blue-team/phishing/images/SOC%20101%20-%20Phishing%20Challenge%203-12.png)
Searching the user `TCWUS` cannot be found as most likely the user is taken down for malicious activity.
