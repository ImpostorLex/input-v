---
{"dg-publish":true,"permalink":"/cards/blue-team/siem/cas-command-injection/"}
---

[[cards/blue-team/siem/Common Attack Signatures\|Common Attack Signatures]]

- Execution of OS commands through vulnerable applications such as web app, desktop application

**Look for:**
- `;`, `||`, `&&` - characters that seperate commands.
- **os** commands