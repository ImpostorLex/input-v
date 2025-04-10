---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/phishing/"}
---

[[Phishing MOC\|Phishing MOC]]
### Introduction
---
An attempt of stealing sensitive information by using social engineering tactic via communication mediums such as email, SMS, phone calls, and the Internet, usually posing as a **known entity** and instilling **fear**.

- Scarcity
- Familiarity
- Urgency
- Social Proof - "Hundreds got rich and now it could be you."
## Attack Types
---
- **Information Gathering (Recon phase)** - craft credible phishes
	- Determine if the account is used and if the receiver is prone to opening emails.
	- **Tracking pixels** (usually in a size of 1px and transparent) when email is opened, the attacker could gather the OS, how many times the user opens it, IP address, when was the email opened, and email client.

![Pasted image 20250103152615.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250103152615.png)
- **Credential Harvesting**
	- Including any sensitive information.
	- Done through fake login pages, deceptive URLs.
- **Malware Delivery**
	- Malicious attachments or links.
	- **Drive-by download** - automatically download without user action.
- **Spear Phishing**
	- Targeted and customized phishing
- **Whaling**
	- High-profile individuals such as CEOs, and executives
- **Vhishing, SMSishing. and Quishing**
- **Business Email Compromise (BEC)**
	- Unauthorized wire transfers, invoice scams such as "I am CEO, please buy me gift cards and send codes thanks"
- **Spam** not typically malicious but still dangerous.
### Techniques
---
These are techniques use to make the _attack types_ even more effective.

- **Pretexting**
	- Fabricating a backstory.
- **Spoofing and Impersonation**
	- Email Address Spoofing.
	- Domain Spoofing.
- **URL Manipulation**
	- URL Shortening.
	- Subdomain Spoofing.
	- Homograph Attacks - _homograph_ means one word that have two or more meaning such as bat (animal and baseball bat).
		- For example, the [Cyrillic](https://en.wikipedia.org/wiki/Cyrillic_script "Cyrillic script"), [Greek](https://en.wikipedia.org/wiki/Greek_alphabet "Greek alphabet") and [Latin](https://en.wikipedia.org/wiki/Latin_script "Latin script") alphabets each have a letter ⟨o⟩ that has the same shape but represents different sounds or phonemes in their respective writing systems
		- Also known as _internationalized domain name_ attack.
- **Typosquatting**
	- It is a type of attack that intentionally uses misspelled domain name.
	- Similar to **Homograph attack** but it mostly relies on the user not paying attention such as 'linkedin.com' vs 'linkedinn.com'.
- **Encoding**
	- Obsfucate malicious content to evade detection.
	- Base64, URL encoding, HTML encoding.
 - **Abuse of Legitimate Services**
	 - Google Drive, Dropbox, etc.
- **Pharming**
	- Two steps technique - downloads a malicious program with the goal of redirecting victim to attacker controlled sites.


**Organize and refine notes after each session including adding meta-tags**

