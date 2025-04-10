---
{"dg-publish":true,"permalink":"/cards/blue-team/siem/cas-xss/"}
---

[[cards/blue-team/siem/Common Attack Signatures\|Common Attack Signatures]]

- Injecting malicious code into javascript
	- Steal cookies.
	- Hijack user sessions.

**Look for**
- `<script>` tags sometimes url encoded and have bad letter casing.
- event handlers such as: `onload`, `onclick`, and `onmouseover`.

#### References
---
- [https://github.com/payloadbox/xss-payload-list](https://github.com/payloadbox/xss-payload-list)
- [https://www.w3schools.com/tags/ref_urlencode.ASP](https://www.w3schools.com/tags/ref_urlencode.ASP)