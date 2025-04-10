---
{"dg-publish":true,"permalink":"/cards/web/bug-bounty/revealing-secrets-with-information-disclosure-bugs/"}
---

[[+ simmering/Bug Bounty\|Bug Bounty]]

- Personal Identifiable Information
- Access Control issues
- Error messages / Software Disclosure
- Comments, API keys.

### How do you know it's a information disclosure vulnerability?
---

- **Any information that identifies someone is**, even if it's out of date or wrong as long as it relates to an individual.
- Sensitive contexts such as business logics
	- Able to see the answers from a education app.
	- Seeing the real username when it says anonymous message.
	- Private post but viewable by others.

#### Things to think about
---

**Non-technincals**:
- How does your target make money? subscriptions
- What would ruin what makes the app fun?
- Would a customer of the target be annoyed?
	- Such as education app students being able to cheat.
- How could you game the system?
- What does your target value? E.g. anonymised, compliance.

**Technincals**
- Bugs disclosed information about what is the target is using or a third party target is using
	- **Not really a bug but should demonstrate the impact**: such as API keys are being rotated monthly (commited in github) so it is not a bug if the month passes or software version.

**Resources**
- Payloads All The Things
- FuzzDB
- TruffleHog - automate finding API keys on github/lab.
	- High risk for duplicates since it is automated.