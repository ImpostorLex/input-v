---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/sender-policy-framework-spf/"}
---

[[cards/blue-team/phishing/Email\|Email]]

allows domain owners to which mail servers are authorized to send emails on their behalf. 

- It works by publishing SPF records on Domain Name Systems (DNS) - it contains list of IP address or servers.
	- `return-path`'s domain is used to contact DNS for spf record and tested against the immediate sending server IP address to see if it is listed.
- **SPF doesn't limit where your emails are sent**. Your emails can still go to any mail server globally, regardless of SPF

![Pasted image 20250104174142.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250104174142.png)
- **v** version
- **ip4** specific IPv4 address that is allowed to send emails on behalf of domain owners.
- **include:\_spf** authorizes Google's mail servers to send emails on behalf of domain owner.

**Switches:**
- **`-all` (hard fail)**: Reject the email outright or mark it as spam.
- **`~all` (soft fail)**: Accept the email but mark it as suspicious (spam folder).
- **`+all`**: Accept any email, effectively allowing spoofing (not recommended).

# Sample Analysis
---
filename: `sample3.eml`

**Note:** The SPF check doesn't directly look at the **Received headers** but focuses on the domain in the **return path** sender. Even though SendGrid (the initial two headers) handled the email initially, the domain used for SPF is tied to Namecheap's infrastructure.

**Note:** first image is the earliest mail server (originating) in this case it is the first header with a **from MTg4NTY2MDQ**.

![Pasted image 20250105140636.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250105140636.png)
Third received header:

![Pasted image 20250105141341.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250105141341.png)
Per the image, the connection is encrypted using a protocol called [[Transport Layer Security\|Transport Layer Security]] and this cannot happen without (most likely since it is email) a TCP connection, so therefore the IP address shown **149.72.142.11** is trustworthy and cannot be spoofed.

**Tools:** [whois](https://whois.domaintools.com/149.72.142.11)

![Pasted image 20250105142028.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250105142028.png)
It matches with the received headers and next is authentication checks:

![Pasted image 20250105142233.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250105142233.png)
- **Received-SPF**
	- Essentially, Our **Google mail server** reads the domain from the `Return-Path` header and asks **mailserviceemailout1.namecheap.com** if they send emails from **149.72.142.11** because we received an email that claimed to be from you (namecheap).

Then using tools such as `nslookup` or `dig` to confirm:

![Pasted image 20250105144134.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250105144134.png)
The `highlighted` matches with the IP address found on the `received` header as CIDR notation **/16** corresponds to **149.72.0.0 - 149.72.255.255**.

In summary, the SPF record found on the header includes sendgrid.net indicating emails may be sent through SendGrid's infrastructure and the receiving Google server verified this and allowed.


## Problem
---
- It does not verify the sender, it only verifies the 'from' or 'return-path' by querying the **domain's SPF records**.

If an attacker **registers their own domain**, maybe a look - alike domain and setups SPF records themselves or uses a service like Gmail or Yahoo, they will pass the SPF checks simply because recipient mail server will query the DNS SPF records that the attacker setup or using third-party such as Gmail which is already setup and configured.


#### Attack Scenario:
---
![Pasted image 20250104183740.png](/img/user/cards/blue-team/phishing/images/Pasted%20image%2020250104183740.png)

1. Attacker sends an email:
    - **From:** `alerts@chase.com` (spoofed sender address)
    - **Via:** ProtonMail (ProtonMail’s mail server IP).
2. Recipient server checks `chase.com`’s SPF record:
    - It finds that ProtonMail’s IP is **not listed**.
3. Outcome:
    - If the SPF record ends with `-all`, the email is **rejected**.
    - If it ends with `~all`, the email is flagged and likely sent to the spam folder.