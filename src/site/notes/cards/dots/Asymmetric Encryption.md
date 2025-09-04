---
{"dg-publish":true,"permalink":"/cards/dots/asymmetric-encryption/"}
---

~ [[cards/dots/Public Key Infrastructure (PKI)\|Public Key Infrastructure (PKI)]] 
### Introduction
---
Uses two keys one is for the encryption the public key and the other one is for decryption the private key.

## Workflow 
---
1. Alice generates the two keys
2. Alice gives Bob the public keys 
3. Bob uses Alice's public key to encryp data 
4. Bob sends encrypted data to Alice 
5. Alice uses her private key to decrypt Bob's encrypted data.

### Signatures 
---
Integrity testing, if alice wants to prove she sent the message

1. Pam uses her **private key** to encrypt data 
2. Jim uses pam's public key to decrypt data
	- This proves that Pam must have sent the message - **Authentication**
	- Message was not modified because if someone captures and modified the data (assuming the decryption works) it would result into a mess of data once Jim decrypts it - **Integrity**


- However this type of encryption is mathemetically intensive which is impratical for everyday use.
- **Scalability**: The number of keys required in an asymmetric system grows linearly with the number of participants. Each participant only needs one key pair (one public key and one private key).
- Public key can be distrubted openly

## Symmetric Encryption
---
- The key cannot be sent the same way as the encrypted message as man-in-the-middle could easily intercept it.
	- The key is sent through different channel known as **out-of-band** key distrubution such as phone, courier, and fax.
- Not so great for scalability assuming that every employee wants to communicate with other employee individualy they will need a lot of keys.
- Great for bulk data
