---
{"dg-publish":true,"permalink":"/cards/networking/tls-handshake/"}
---

~ [[cards/dots/Public Key Infrastructure (PKI)\|Public Key Infrastructure (PKI)]] | ~ [[cards/networking/Transport Layer Security\|Transport Layer Security]]
### Introduction
---
SSL/TLS certifcates are issued by [[cards/dots/Certificate Authority\|Certificate Authority]].

![TLS handshake.png|500](/img/user/cards/networking/images/TLS%20handshake.png)

After doing [[cards/networking/The threeway handshake\|The threeway handshake]], if the website uses [[http#HTTPs\|HTTPs]] protocol it then does the *TLS handshake*, this make sures that the data transmitted is secured by encryption.

The client and the server on TLS handshake:

1. Specify which version of TLS is used(1.0, 1.2, 1,.3, etc).
2. Decide on which [[cipher suites\|cipher suites]] (a set of algorithms) will be used.
3. Authenticate the server's identity via the server's public key and the [[cards/dots/Certificate Authority\|Certificate Authority]]'s digital signature
4. Generate session keys in order to use symmetric encryption after the handshake is complete

## Steps

1. **Client Hello** message : - the client initiates the handshake by sending a hello message, this includes the TLS version that the client supports, the cipher suites supported, and a random string known as 'client random'.
2. **Server Hello** message: - reply's back with the server's SSL cefcates, the server chosen cipher suit and the random string known as 'server random'
3. **Authentication** : - client verifies server's ssl certificate with the [[cards/dots/Certificate Authority\|Certificate Authority]] that issued it, this confirms that the server say who it is and interacting with the actual owner of the domain
4. **Premaster secret** : - client sends one more random string known as 'premaster secret' is encrypted with the public key it got from the server's ssl certificate and can be decrypted by the server's private key
5. **Private Key** : - server decrypts the premaster secret.
6. **Session Key** : - client and server generates session key from client random ,server random, and premaster secret.
7. **Client is ready:** - client sent a **finished** message encrypted with the session key
8. **Server is ready:** - server sends a finished messages      
9. **Secure Symmetric encryption acheived** : - handshake is completed and communication continues using session keys 

The method above describes [[cards/dots/Hybrid Encrpytion\|Hybrid Encrpytion]]