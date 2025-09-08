---
{"dg-publish":true,"permalink":"/cards/dots/passkeys/"}
---

~ [[atlas/dots\|dots]]
### Introduction
---
The reason why many cybersecurity experts wants the user/password format:

Passkey is any authentication method that replaces username/password format such as biometrics: face and fingerprint scan, screen-lock feature, or even a pin. 

The browser or operating system prompts you to create a passkey 

The process instead of sending a password to the server first - the device decides if your authentication is successful through the mentioned authentication alternatives then your devices tells the server you want to log in.

1. Private key is stored in your device. 
2. The matching public key is stored in the server.

## Pros and Cons
---

- Server hacked? no password can't be stolen 
- Phishing is not possible - since you don't even need to use password and again your device authenticates you. 
- The passkey is not sent to the server, it stays on your device.

Downsides:

Lose your device then you will have a hard time recovering your account.
### Questions
---
> So assuming synch is not supported say on Windows when you create passkey on Windows machine you are basically creating a new account?

No, website or apps usually asks for an email or username to identify you - then uses passkey to authenticate, so when you create a new passkey for not synched device - it will simply create a new passkey telling the server that this account has multiple passkeys.
