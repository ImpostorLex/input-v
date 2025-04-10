---
{"dg-publish":true,"permalink":"/cards/web/bug-bounty/owasp-top-10/"}
---

[[cards/web/bug-bounty/Access Control and IDORs\|Access Control and IDORs]]
A user or an attacker can bypass authorization.

- **Insecure Direct Object Reference (IDOR)** - Access to objects based on user-provided input such as ID or username.

**Cryptographic failure**
A vulnerability of misusing or not using cryptographic algorithms for protecting sensitive information.

**Injection**
Occurs when application interprets user-controlled input as commands or parameters.

- **SQL Injection** - SQL queries are inputted to manipulate outputs.
- **Command Injections** - user inputs are passed down to the operating system.
	- Prevent: use virtualization technologies such as docker
	- Prevent: Limit user privilege.
	- Prevent: Sanitize data.
	- Detect: Look for terminal languages/code - most of the time it is UNIX based OS.
	- Detect: Look for common payloads.

Inline commands allows user to run commands inside a command with a format like this:

![OWASP Top 10.png](/img/user/cards/web/images/OWASP%20Top%2010.png)

**Insecure Design**
The idea behind the whole or part of the application is flawed from the start, not from poor coding practices or improper configuration but flawed from the start.

For example:
Instagram 6 digit code password reset are brute-forced using 250 attempts per IP address.

**Security Misconfiguration**
Security configuration was not or not properly implemented.

For example:
Forgot to change default password for admin.

**Identification and Authentication Failure**
- Brute force attacks.
- Weak credentials.
- Weak session cookies.

**Software and Data Integrity Failures**
It is either software or data that has been modified in transit by the attacker.

A simple trick for web application including external libraries:

If the hash is changed, any client navigating to the application won't execute the modified version.

```html
<script src="https://code.jquery.com/jquery-3.6.1.min.js" integrity="sha256-o88AwQnZB+VDvE9tvIXrMQaPlFFSUTR+nldQm1LuPXQ=" crossorigin="anonymous"></script>
```

**Security Logging and Monitoring Failures**
Is the footprints of every events performed, failure to log important activities could lead to fines and even jail time.

**Server - Side Request Forgery**
A web application is tricked into making request to another resources on behalf of the attacker.

### Questions
---
> How User-Agent is vulnerable to Injections?

It ultimately depends on the context or use case such as displaying user-agent
back to the user or administrator pages such as for Analytics, Tracking and Feature Detection and Compatability (For web applications)