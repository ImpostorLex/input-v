---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/reactive-phishing-defense/"}
---

[[map-of-contents/blue-team\|blue-team]]
## Reactive Phishing Defense
---
Again, to make an effective SOC is how we respond to incident

- **Containment**
	- **Determine the scope first**: did only one user received the email or not and this user what level of authority she/he has.
	- **Blocking sender artifacts** (collected from the triage & analysis) such as IP address, URL, domain name, and blocking email address (dangerous analyze scope first). can be done on email gateway such as microsoft exchange.
	- **Blocking web artifacts** 
		- Blocking domain is dangerous but asking this question
		- "Is the employee or executive going to visit this web in the future?"
		- Block at the firewall level or endpoint level such as EDR.
	- Additionally checking the domain age is important with tools such as whois.
	- **Blocking file artifacts** file hashes or name through EDR.
- **Eradication**
	- Remove malicious emails  and attachment from any machines.
	- Report abuse form submissions to any relevant entities such as phish tank.
		- `whois` tool outputs an `abuse contact email`
	- Credential changes to the affected user, regardless if they did not type anything to the site, we can't really trust them.
- **Recovery**
	- Restore systems.
- **Communication**
	- Notify affected users.
	- Update stakeholders.
- **User Education