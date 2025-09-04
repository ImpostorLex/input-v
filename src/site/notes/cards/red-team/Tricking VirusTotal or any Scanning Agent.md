---
{"dg-publish":true,"permalink":"/cards/red-team/tricking-virus-total-or-any-scanning-agent/"}
---

~ [[atlas/red-team\|red-team]]

Source: https://medium.com/maverislabs/virustotal-is-not-an-incident-responder-80a6bb687eb9

Summary: tools such as VirusTotal scans submitted URLs and look for suspicious indicators - however this could be easily be bypassed by using obfuscated javascript to serve a benign webpage or a malicious one based on `user-agent`, a technique called **user-agent filtering**, or using Apache's mod_rewrite to serve different pages according to the discovered `user_agent`.

