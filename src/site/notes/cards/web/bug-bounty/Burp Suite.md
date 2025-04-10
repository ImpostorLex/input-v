---
{"dg-publish":true,"permalink":"/cards/web/bug-bounty/burp-suite/"}
---

[[]]****

It is a tool mainly use for web application security testing and works best with foxyproxy for quickly switching settings and disabling proxy.

**Adding CA Certificate**
This makes our browser trust burp suite to intercept and analyze encrypted traffic:

1. http[:]//burp at browser, make sure foxyproxy is configured and enabled.
2. Top right corner click CA-Certificate.
3. Browser settings -> Privacy & Security -> Certificates -> View Certificates.
4. Click Import then check the two checkboxes:
	- Trust this CA to identify websites.
	- Trust this CA to identify email users.
5. Test by intercepting HTTPs traffic.

- Firefox Multi Account Containers is great if you want to test multiple states of a web app such as logged in, not logged in, and admin account without opening up a different browser.
- Take advantage of the comparer.
- Payloadallthethings.


**Some cool features**

**Scan Specific Endpoint**
This allows you to analyze an endpoint reducing the noise and increasing performance.

1. Intercept a request.
2. Send to Intruder.
3. Highlight specific parameters.

![Burp Suite.png](/img/user/cards/web/images/Burp%20Suite.png)

**Copy URLs**

![Burp Suite-1.png|450](/img/user/cards/web/images/Burp%20Suite-1.png)

- Copy all URLs found in the target including links to third parties or sites that is not in scope.
- Copy URLS only in scope.

