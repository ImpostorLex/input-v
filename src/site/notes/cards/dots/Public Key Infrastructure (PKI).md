---
{"dg-publish":true,"permalink":"/cards/dots/public-key-infrastructure-pki/"}
---

~ [[atlas/dots\|dots]]

It usually composed of three things:

1. **Client** - TLS handshake iniator - rarely authenticated
	 - **Generally A Web Browser** - but we are in the Internet of Things so basically everything that is connected to the Internet uses TLS.
2. **Server** - TLS handshake receiver - always authenticated by providing a certificate proving that this is what we are looking for (client).
3. [[cards/dots/Certificate Authority\|Certificate Authority]]

## Samples
---
**The Internet**: try the drawing for better visualization: [[PKI Infrastructure.excalidraw\|PKI Infrastructure.excalidraw]]

![Public Key Infrastructure (PKI).png](/img/user/cards/dots/images/Public%20Key%20Infrastructure%20(PKI).png)

It performs the [[cards/networking/TLS handshake\|TLS handshake]] after [[cards/networking/The threeway handshake\|The threeway handshake]] is performed. 

But why, create a symmetric key after the _TLS handshake_ because [[cards/dots/Asymmetric Encryption\|Asymmetric Encryption]] is great for small data! but symmetric is, great for bulk data why not both? Introducing [[cards/dots/Hybrid Encrpytion\|Hybrid Encrpytion]]

**Code Signing Example:**

![Public Key Infrastructure (PKI)-1.png](/img/user/cards/dots/images/Public%20Key%20Infrastructure%20(PKI)-1.png)

The (1) is used to verify the software and the (2) is to make the OS trust the software.

**Internal Corporate PKI:**

![Public Key Infrastructure (PKI)-2.png](/img/user/cards/dots/images/Public%20Key%20Infrastructure%20(PKI)-2.png)

Companies will implement their own CA for their corporate resources to make the employee trust the corporate resources.

**Another key component:**

Is [[digital signature\|digital signature]] often times referred only as a signature.

A _digital signature_, the sender signs data using their private key, this allows anyone to verify the signature using the sender's public key.

This proves the following:

- **Authentication** proves the sender really owns the private key.
- **Integrity** shows the message wasn't alterated in transit (explained below)
- **Non-repudiation** the sender can't later deny they signed it (since only their private key could have made that signature)

**Example:**

- Sender takes the message and compute its hash.

- Encrypts the hash with their **private key** -> that’s the signature.
    
- Receiver gets the message + signature.
    
- Receiver uses sender’s **public key** to check if the signature matches the message by decrypting the signature it will yeild or results into the hash the sender created (at step 1).
	
	- You may ask how? the receiver checks the hash of the message and compares it to the decrypted signature's hash. 
	- If the hash did not match, it means the message was tampered.

