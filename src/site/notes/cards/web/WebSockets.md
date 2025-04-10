---
{"dg-publish":true,"permalink":"/cards/web/web-sockets/","tags":["sunday"]}
---

[[map-of-contents/web\|web]]
### Introduction 
---
Websockets opens a constant line of communication between browser and server unlike the other method of asking something then getting a response.
## HTTP Requests vs. WebSocket

- **Hyper Text Transfer Protocol** sends request to server, server responds, and then closes the connection.
- **WebSocket** is the opposite, send update anytime you want the line is open.
## Vulnerabilities

**Note:** This assumes that no proper security measures in place:

- **Weak Authentication and Authorisation** has no built-in ways to handle user authentication or session validation unlike http.
- **Message Tampering** unencrypted messages can be intercepted.
- **Cross-Site WebSocket Hijacking (CSWSH)** attacker tricks victim's brower into opening a WebSocket connection to another site.
- **Denial of Service (DoS)** since websockets remains open the attacker could flood the server with tons of message.
### Intercepting Messages
---
In burp suite, to setup:

![WebSockets.png](/img/user/cards/web/images/WebSockets.png)


### Questions and Problems
---
> Does server perform GET, REQUEST too?

No.


## Conclusion


