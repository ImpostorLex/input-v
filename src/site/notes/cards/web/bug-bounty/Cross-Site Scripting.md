---
{"dg-publish":true,"permalink":"/cards/web/bug-bounty/cross-site-scripting/"}
---

[[+ simmering/Bug Bounty\|Bug Bounty]]

It is where malicious actor is able to inject scripst commonly javascript into a web application, the script is executed on the client side.

- **Stored XSS** - payload is stored on database and called every time a page is loaded.
- **Blind XSS** - payload executed but cannot see the output or in different page that is not accessible normally.
	- Use XSS hunter
	- Well crafted XSS payload to check if XSS executed.
- **DOM XSS**: 
	- _Document Object Model (DOM)_  represents every objects (html elements and other properties of the webpage) into a hierarchical tree, making it possible to dynamically manipulate content, structure and styling of a web page.
		- `document.URL`, `.location`, `.cookie`, `.innerHTML`, and `.outerHTML`
	- It only happens on the client side.
- **Reflected XSS** - payload is not stored but simply reflected on another page.

Basic payload:

```bash
"> <script>alert(0)</script>
```

**How to find them?**
- Try nettitude's xx_payloads in github
- spam alert on every form


**Things to avoid as a developer**
- No input sanitation
- Inline javascript as tools such as burp intercepts loaded external `.js` seperately from the `.html` as oppose to inline.


```
<script/"<a"/src=data:=".<a,[8].some(alert)>
```

![Pasted image 20241121175922.png](/img/user/cards/web/images/Pasted%20image%2020241121175922.png)

**Detection**
- Keywords navigate to a github for payloads and find the most common such as alert and script.
- Special characters such as `<>`.


```
from flask import Flask, Response
from werkzeug.http import parse_cookie

app = Flask(__name__)

@app.after_request
def add_security_headers(response: Response):
    response.headers['X-XSS-Protection'] = '1; mode=block'
    response.headers['Content-Security-Policy'] = "default-src 'self'; script-src 'self';"
    response.headers['Strict-Transport-Security'] = 'max-age=31536000; includeSubDomains'
    response.headers['X-Content-Type-Options'] = 'nosniff'
    return response

```

https://owasp.org/www-project-web-security-testing-guide/assets/archive/OWASP_Testing_Guide_v4.pdf