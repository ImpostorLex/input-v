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
Starting off with the tool `oledump.py <file doc>` to determine data streams with macro indicated by a capital 'M':

Then input the index number replace **'3'** in the code below:

```C
python3 ../Tools/oledump.py AR_Wedding_RSVP.docm -s 3 --vbadecompresscorrupt
```


**pdf analysis** - using pdf to send a URL as payload is an effective way of evading firewall or filters plus it is widely used such as invoices.

```C
pdf-parser.py <mal.pdf>
```

![Attachment Examination.png](/img/user/cards/blue-team/soc/images/Attachment%20Examination.png)

Search for a specific key such as a `URL` or a `URI` to search in the data streams:

```bash
pdf-parser <mal.pdf> -s "/URI"
```

![Attachment Examination-1.png](/img/user/cards/blue-team/phishing/images/Attachment%20Examination-1.png)

**pdf-id** can scan a pdf file and give a high level overview:

```bash
pdf-id.py <file.pdf_
```

![Attachment Examination-2.png|450](/img/user/cards/blue-team/phishing/images/Attachment%20Examination-2.png)

Things to look out for:

1. Javascript or JS
2. OpenAction - tell a pdf reader to do some action after opening the document.
3. Launch - used to launch another object such as external software after opening the document.
4. Embedded File. 

Then we can use `pdf-parse.py` to further analyze the `.pdf`:

```bash
pdf-parser.py <file.pdf>
```

![Attachment Examination-3.png](/img/user/cards/blue-team/phishing/images/Attachment%20Examination-3.png)
The Javascript is basically used to launch the embedded file which in this case  **eicar-dropper.doc**, to extract the file:

The object (.doc) file is located at **object 8** in our case:

![Attachment Examination-4.png](/img/user/cards/blue-team/phishing/images/Attachment%20Examination-4.png)

```C
pdf-parser.py <file.pdf> --object 8 --filter --raw --dump <good_file_name>
```

- `--object` what object.
- `--filter` the instance of **'/FlateDecode'** means the file is encoded - this basically 'decodes' the file.
- `--raw` display it in raw format.
- `--dump` where to save the file and what name? (for best practice use the same name provided.)

Since it's a `.doc` file -- we can go back and use `oledump.py` again:

```C
oledump.py eicar-dropper.doc
```

Output here .....

```C
python3 ../Tools/oledump.py eicar-dropper.doc -s 3 --vbadecompresscorrupt
```



**Good idea to get a hash of the file then use it for threat intelligence.**

[https://www.phishtool.com/](https://www.phishtool.com) - automated.

