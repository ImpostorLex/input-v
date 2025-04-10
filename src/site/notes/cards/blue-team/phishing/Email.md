---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/email/"}
---

[[cards/blue-team/phishing/Phishing\|Phishing]]
### Introduction
---
![Pasted image 20250102135751.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250102135751.png)
- **Mail Transfer Agents (MTA)** are software programs within mail servers responsible for sending, receiving, routing email between mail servers.
- **Mail Delivery Agents (MDA)** are software programs within mail servers responsible for **delivering incoming emails to the recipient's mailbox**.
- **Mail User Agent (MUA)** - interface between user and email system: gmail, outlook, and thunderbird.

- Refer to vault input on how _DNS works for email_ (Note name: "How the web works") 
- In modern software: MTA and MDA are merged.
## Email Authentication
---
- [[cards/blue-team/phishing/Sender Policy Framework (SPF)\|Sender Policy Framework (SPF)]] 
	- Verifies that the **domain found on 'return-path' header** is authorized to send emails on behalf of the sender's domain.
- [[cards/blue-team/phishing/DomainKeys Identified Mail (DKIM)\|DomainKeys Identified Mail (DKIM)]]
	- Ensures that the **content** of the email (including headers and body) has **not been tampered with** in transit and verifies the email's **authenticity**
- [[cards/blue-team/phishing/Domain-based Message Authentication Reporting & Conformance (DMARC)\|Domain-based Message Authentication Reporting & Conformance (DMARC)]]
 - Attacker relies on **user bad awareness** as attacker could setup domain -look alikes and then configure their own SPF and DKIM effectively bypassing both authentication methods.
## Email Content
---

![Email.png](/img/user/cards/blue-team/phishing/images/Email.png)
- **Multipurpose Internet Mail Extension (MIME)** 
	- Allows different type of contents such as plain, html, images or any attachments including plain and html version for compatibility.
	- It uses **boundary markers** to seperate each parts of the email.
- **Content-Type**
	- Indicates what type of content is expected.
	- **Quoted Printable**
		- Soft line breaks at the end of each line and has a line limit of 76 characters.
		- It is used for obsfucation alongside Base64, URL and HTML entities encoding



### Questions
---
> What does the three authentication methods solves?

It effectively mitigates spoofing of email address but not domain-look a like attacks.

> Is it up to the receiving administrator to honor DMARC?

Yes, **The receiving mail server admin** decides whether to **honor the DMARC policy**. Most major email providers (like Gmail, Outlook, etc.) **honor DMARC** policies.

This check happens on the receiving end.


**Organize and refine notes after each session including adding meta-tags**