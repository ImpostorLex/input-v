---
{"dg-publish":true,"permalink":"/cards/active-directory/new-technology-lan-manager-ntlm/","tags":["windows/ad"]}
---

~ [[cards/active-directory/Active Directory\|Active Directory]] | ~ [[cards/red-team/Breaching Active Directory\|Breaching Active Directory]]
### Introduction
---
is one of Microsoft's old protocol family for authentication, this is done with a mechansim called _NetNTLM_ which allows authentication using a challenge-response-based scheme often used by services such as RDP, OWA, VPN or IIS web applications to authenticate a user against Active Directory without directly handling the user's password.

![New Technology LAN Manager (NTLM).png|500](/img/user/cards/active-directory/images/New%20Technology%20LAN%20Manager%20(NTLM).png)

1. **Client → Service:** "Hi, I’m Alice, I want to log in."

2. **Service → Client:** "Prove it. Here’s a random number (the challenge)."

3. **Client → Service:** Takes the challenge, encrypts it using Alice’s password hash, sends back the result (the response).

4. **Service → Domain Controller (AD):** Forwards both challenge + response to the DC.

5. **DC → Service:** "Yes, that matches Alice’s password hash" (or "no, it doesn’t").

6. **Service → Client:** "Okay, you’re authenticated."

The service never learns Alice’s password — it just acts as a **middle-man**.

**Note:** The described process applies when using a domain account. If a local account is used, the server/seriice can verify the response to the challenge itself without requiring interaction with the domain controller since it has the password hash stored locally on its SAM.


#### Key Topics
---
## Key Topic


