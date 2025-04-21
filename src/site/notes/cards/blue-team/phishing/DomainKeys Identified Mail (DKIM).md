---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/domain-keys-identified-mail-dkim/"}
---

[[cards/blue-team/phishing/Email\|Email]]

Allows the receiver to verify an email message if it was indeed sent from the domain it claims to be and ensures the message is not tampered using Public Key Infrastructure (PKI).

![Pasted image 20250104190256.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250104190256.png)

**bob@chase[.]com**
1. **Private Key Signing**
	- Sending mail server digitally signs certain parts of the email or the whole using the domain's private key. (**hash then encrypts using  private key**) 
	- This signature is included in the email's headers as a `DKIM-Signature`.
2. **Public Key Verification**
	- The recipient's mail server queries the **DNS records** of the sender's domain (`chase.com`) to retrieve the **public key**.
	- The recipient uses this public key to decrypt the signature.
	- If the decrypted signature matches the content of the email, it confirms two things:
		- The email has not been **tampered with**.
		- The email was sent by an **authorized mail server for the domain.**

# Analysis
---
filename: `sample3.eml`

There is two `DKIM-Signature` one for **namecheap** and one for `sendgrid`:

![Pasted image 20250105145412.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250105145412.png)
- It most likely means that the email is processed through multiple email systems each applying their own DKIM signature.
- It has the same body hash and header fields (some)

**The goal is looking for integrity : main purpose** therefore we should focus on the signature that is closest to the source (**namecheap**).

![Pasted image 20250105145916.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250105145916.png)
- `v=1` version and it should be typically 1.
- `a=rsa-sha256` algorithm used to generate signature (strong algorithm).
- `c=relaxed/relaed`
	- **Canoncalization algorithm** one header for the body and one for the header. It is used by DKIM to ensure that the signed content is **consistent** across email systems: headers and body is not failing or mismatched due to whitespace or formatting changes.
	-  `relaxed` minor changes in the whitespaces and linebreaks are allowed and is an accepted variation that maintains the integrity
- `d` domain and `s=s1` selector used to locate the public key used to verify the signature
- `bh` **body hash** - it is encrypted using the specified hash and encoded in base64.
- `b` **DKIM Signature itself** which is calculated based on the header fields and  body hash.

**Verifying and locating public key**

```bash
nslookup -type=txt <selector>._domainkey.namecheap.com
```

![Pasted image 20250105150844.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250105150844.png)
Or [DKIM lookup](https://mxtoolbox.com/dkim.aspx)

Putting the entire content and analyzing DKIM namecheap.com
https://mxtoolbox.com/Public/Tools/EmailHeaders.aspx?huid=92d20dbc-9a48-4567-a6c3-17ecd5b28fac

![Pasted image 20250105151919.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250105151919.png)
The body in the hash did not verify meaning the content is tampered.


## Problem
---
It has the same issue as [[cards/blue-team/phishing/Sender Policy Framework (SPF)\|Sender Policy Framework (SPF)]] the attacker could rely on look-alike domain and then setup their own DKIM.

