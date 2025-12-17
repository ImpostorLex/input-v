---
{"dg-publish":true,"permalink":"/cards/active-directory/active-directory-certificate-services/"}
---

~ [[cards/active-directory/windows active directory\|Active Directory]]
## AD Certificate Services
---
It is the Microsoft's [[cards/dots/Public Key Infrastructure (PKI)\|Public Key Infrastructure (PKI)]] implementation and it can be used as a [[cards/dots/Certificate Authority\|Certificate Authority]] to prove and delegate trust. It has multiple purpose: encrypting file systems, creating and verifying digital signatures, and even user authentication.

**Probably let this paragraph stay:**

It usually runs on selected domain controllers, normal users can't really interact with the services directly. **Certificate templates** are handy for administrator distributing each certificate manually but they can allow any user with relevant permissions to request a certificate themselves. 

These templates have parameters that say which user can request the certificate and what is required.

- **Certificate Template** - a collection of settings and policies that defines how and when a certificate may be issued by a CA
- **CSR** - Certificate Signing Request is a message sent to a CA to request a signed certificate
- **EKU** - Extended/Enhanced Key Usage are object identifiers that define how a generated certificate may be used