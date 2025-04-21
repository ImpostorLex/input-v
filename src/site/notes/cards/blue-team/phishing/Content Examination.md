---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/content-examination/"}
---

[[cards/blue-team/phishing/Email Analysis\|Email Analysis]]
## Content Examination
---
![Email Analysis-1.png](/img/user/cards/blue-team/phishing/images/Email%20Analysis-1.png)
- Org name mismatch. 
- Awkward greeting.
	- Sometimes generic such as "Dear Customer".
- Sense of urgency.
- Grammar Issues. 
	- The above should raise a flag but not enough evidence.
- obsfucated content such as the use of base64 and the use of _html entities_ are reserved keywords by html but rendered as content rather than interpreted as HTML code:

![Email Analysis.png](/img/user/cards/blue-team/phishing/images/Email%20Analysis.png)
- **URL Encoding**
	- Replaces content with a percent sign ' % ' followed by a two-digit hexadecimal value that represents the character in the ASCII set.