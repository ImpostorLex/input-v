---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/domain-based-message-authentication-reporting-and-conformance-dmarc/"}
---

[[cards/blue-team/phishing/Email\|Email]]

Enhance both [[cards/blue-team/phishing/Sender Policy Framework (SPF)\|Sender Policy Framework (SPF)]] and [[cards/blue-team/phishing/DomainKeys Identified Mail (DKIM)\|DomainKeys Identified Mail (DKIM)]] using additional reporting mechanisms to domain owners.

Allow domain owners to specify what to do if email fails one or both the authentication method:

- **None (p=none)**
	- Still gathers data for reporting
- **Quarantine (p=quarantine)** directs suspicious emails to the recipient spam folder.
- **Reject (p=reject)** rejects or drops the email.