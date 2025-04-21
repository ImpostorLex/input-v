---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/attachment-examination/"}
---

[[cards/blue-team/phishing/Email Analysis\|Email Analysis]]
## Attachment Examination
---
**Extract the attachment safely**
- In thunderbird, we can right click the attachment and hit 'save as', if it's not available we can use `emldump.py`:

**Determine available stream number:**
```C
python3 ../Tools/emldump.py challenge3.eml 
```

Then:
```python
python3 ../Tools/emldump.py sample1.eml -s <stream_number> -d > filename.exe
```

then get the hash: `sha256sum`, `sha1sum`, and `md5sum` or the `eioc.py` does this at the same time.

- `get-filehash .\file.exe -algorithm SHA256` **for Windows**.
- upload the file or hash (much recommended) 
	- Best **check company policy** such as uploading suspicious file in VirusTotal or Cisco talos ending up a legitimate file - community with enterprise subscription can download the uploaded file.
### Dynamic Analysis
---
1. Process Activity.
2. Registry Changes.
3. Network Connections.
4. File Activity.

- Hybrid Analysis - might be queued (generates public report becareful - free tier.)
- Joe sandbox - requires business email.
- Any run - require business email.
### Static Analysis
---
- `oledump.py <file doc>` to determine data streams with macro indicated by a capital 'M'

then input the number replace **' 3 ' **:

```C
python3 ../Tools/oledump.py AR_Wedding_RSVP.docm -s 3 --vbadecompresscorrupt
```

- pdf analysis - using pdf to send a URL as payload is an effective way of evading firewall or filters plus it is widely used such as invoices.
	- pdf parser = `-s "/URI` to search in data streams. 
	- pdf-id.py
		- launch - launch an object when pdf is open

**Good idea to get a hash of the file.**

[https://www.phishtool.com/](https://www.phishtool.com) - automated.

