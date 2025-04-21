---
{"dg-publish":true,"permalink":"/cards/blue-team/phishing/url-analysis/"}
---

[[cards/blue-team/phishing/Email Analysis\|Email Analysis]]
### URL Analysis
---

![Email Analysis-2.png](/img/user/cards/blue-team/phishing/images/Email%20Analysis-2.png)
- **Spoofing**
	- **Top level domain** such as facebook.tor instead of facebook.com.
	- **Subdomain spoofing** facebook.busix.com instead of facebook.com.

**Sample Analysis**
![Email Analysis-3.png](/img/user/cards/blue-team/phishing/images/Email%20Analysis-3.png)
- The use of http instead of https is suspicious especially dealing with login credentials.
- subdomain spoofing apple
- Unusual base domain.
- A very weird subdirectory
- **prefilling information** to make it look much more authentic.

![Email Analysis-4.png](/img/user/cards/blue-team/phishing/images/Email%20Analysis-4.png)
- Subdomain spoofing
- The use of **legitimate service** to conduct malicious action, in this case webflow.io is a site builder as the use of legitimate service may lower the guard of victims as webflow is trusted.

![Email Analysis-5.png](/img/user/cards/blue-team/phishing/images/Email%20Analysis-5.png)
> [!tip]- ANSWER
> It is a legitimate website as the base domain matches with the real wellsfargo but the parameters are strange most likely for developers or frontend reasons.

1. Extract URLs
	- Manually through using sublime text with search filter. OR
	- Upload the file on CyberChef.
		- Keep in mind the **content type**: such as 'quoted printable'.
2. Defang URLs
3. Check site reputation. (via VirusTotal or Cisco Talos)
4. **Analyze base URL**.
	- As we can never know if the attacker compromised a legitimate domain and started a subdomain/subdirectories to perform malicious actions

**Additional tools**
- **Phistank** to see live potential phishing sites.
- **URL2PNG**
- **urlscan.io** to get an overall information including images of the site.
- **virustotal**
	- Best to check both sites, as some phishing sites are new and no submission.
- **wannabrowser** see the actual http content from GET and POST.
- **unshorten.url**