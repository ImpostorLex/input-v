---
{"dg-publish":true,"permalink":"/cards/blue-team/siem/cas-path-traversal-and-local-file-inclusion/"}
---

[[cards/blue-team/siem/Common Attack Signatures\|Common Attack Signatures]]

- include or execute unintended files or scripts.
 - **path traversal** 
	 - Access files/directories outside of the web root; such as accessing the uploaded malicious script in the temp directory
- **local file inclusion**
	- include a local file that is already present from the system.

**Remember the double '....//.....'** refer to LFI or path traversal bypasses.