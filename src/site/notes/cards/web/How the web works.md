---
{"dg-publish":true,"permalink":"/cards/web/how-the-web-works/"}
---

[[map-of-contents/web\|map-of-contents/web]]

WebQuery: google.com (retrieves A (IPv4) or AAAA (IPv6) records).
Email Query: user@example[.]com (retrieves MX records or short for Mail exchange)

#### 1) Domain Name Resolution
1. The local DNS resolver checks if the requested domain's IP is cached. if not then it forwards it to the root name server.
2. The DNS resolver send the request to root nameserver (`a.root-server.com`, `b.root-server.com`) basically points resolve DNS to appropriate TLD. 
3. Root nameserver redirect the request to the appropriate Top Level Domain nameserver using the domain's suffix : `.com`.
4. TLD nameserver holds the records to find the appropriate authorative server.
5. Finally DNS resolver make request to Authorative nameserver then it provides the IP address of the domain name.
	- **In the context of email:** the authorative nameserver gives the address of the mail server responsible for the domain, the responsibilities such as email processing and accepting email messages.

**What if the request is not found on the last step?**
The DNS resolver is responsible for making multiple queries to satisfy the user request if not found then returns a non-existent domain or **NXDOMAIN**, something like this:

![How the web works.png|450](/img/user/cards/web/How%20the%20web%20works.png)

#### 2) TCP Connections
---
Performs the three way handshake:

1. Client initiates a connection by sending a `SYN` flag to server.
2. Server (in our case google.com) respond with `SYN`, `ACK `flag back to client.
	- The server can respond with `RST` which means, it aborts the TCP connection, either it does not want to make any connection (via Firewall or web-proxy level blocking) or the server is down
3. Client sends a `ACK` flag.

4. `FIN` flag is used to end the connection. 
	- Assuming communication is done: the server sends a `FIN` flag to the client.
	- Client receives the `FIN` flag and sends back a `ACK` flag to the server.
	- The server receives the client's `ACK` flag then closes the connection.

#### 3) TLS Handshake
---
Perform the Transport Layer Security Handshake:

1. It starts with a CLIENT hello providing supported TLS and cipher suite.
2. Both client and server agrees to what version of TLS and what encryption algorithm to use.
3. The server provides it's digital certificate to client to verify the server's identity.
4. The client then generates a 'pre-master' key encrypting it with the server's public key from the digital certificate then sends it back to the server.
5. The server decrypts the 'pre-master' key with it's private key.
6. Then generates session key from the client's decrypred pre-master secret.


Then communication now may happen.

**Why http is stateless?**
Server does not remember any previous request, it treats every request as independent from each other, it make uses of session and cookies to remember something.

For example:
To not make the user log in everytime, it make uses of session keys.

**How does Certificate of Authority verifies the legitimacy of  a Web App?**
There are multiple types of certificate that can be issued:

TL;DR
**Certificate of Authority does not verifiy the security of web application only the identity.**

- **Domain Validation (DV) Certificates** verifies if the applicant owns or controls the domain:
	- Sending an email to a domain-related email address.
	- Requesting of specific files.
	- **Trustworthiness Assessed:** Minimal. The CA only confirms domain control, not the applicant's legitimacy or the application’s safety.
- **Organization Validation (OV) Certificates** confirms both domain control and identity of the organization.
	- Business Registration Records
	- Proof of Operational Existence (tax records and utility bills)
	- Phone call.
	- **Trustworthiness Assessed:** Moderate. The CA verifies the organization's legitimacy but does not audit the web application's security.
- **Extended Validation (EV) Certificates** everything above + validate the organization if it's legitimate and good, and determine if the application has the authority to request the certificate. 

