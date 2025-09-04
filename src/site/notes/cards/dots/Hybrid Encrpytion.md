---
{"dg-publish":true,"permalink":"/cards/dots/hybrid-encrpytion/"}
---

~ [[cards/dots/Public Key Infrastructure (PKI)\|Public Key Infrastructure (PKI)]]
### Introduction
---
It uses [[cards/dots/Asymmetric Encryption\|Asymmetric Encryption]] to facilitate key exchange and secret key is used (symmetric encryption) for exchaning data:

1. Pam generates a Symmetric Secret Key (1s and 0s)
2. Pam uses Jim's public key to encrypt the SSK 
3. Pam sends the encrypted SSK to the wire safely
4. Jim decrypts the encrypted SSK with his private key - now both have same SSK. 
5. Pam uses SSK to encrypt data and jim uses the SSK to decrypt the data and vice cersa

With these you get the best of both worlds:

1. More secure - Private key is never shared 
2. Faster - Lower CPU cost (Symm)
3. Cipher text is same size as plain text (symm)

This is used by TLS, IPPsec, SSH used to protect data transfer.