---
{"dg-publish":true,"permalink":"/cards/red-team/phishing/","tags":["sunday","template"]}
---

[[map-of-contents/blue-team\|blue-team]]
### Introduction 
---
It is a type of attack that tricks users by pretending to be a known entity such as organization, friends, and more into performing unwanted action.
## Spoofing
The Simple Mail Transfer Protocol (SMTP) is a trust based protocol meaning it does not verify the identity of the sender.

Modern authentication solves this problem: [[SPF, DKIM, and DMARC\|SPF, DKIM, and DMARC]] however not all domains implements this. tools such as Mxtoolbox can help.

## Email Header
The header is a section of the email containing information such as sender, recipient, and date. There are also components such as 'Return-Path', 'Reply-To', and 'Received'

![Phishing.png|450](/img/user/cards/red-team/images/Phishing.png)
- Sender and Receiver address.
- **Return-Path/Reply-To** the address where the replies from the email goes.
- **Message-ID** uniquely identifies email.
- **MIME-Version** converts non-text content such as images, videos and other attachments into text.
- **Received** listed as reverse chronological order, it shows the email servers that the email passed through.
### Email Header Analysis
---
Refer to bulleted points for questions to ask when doing analysis

- Was the email sent from the correct SMTP server?

The server has an email of **101.99.94.116** as seen from the `Received` key:
![Pasted image 20241205163437.png](/img/user/cards/red-team/images/Pasted%20image%2020241205163437.png)
Then at the sender, we can see the domain **letsdefend.io**:
![Phishing-1.png](/img/user/cards/red-team/images/Phishing-1.png)
So, under normal circumstances, **letsdefend.io** should be using **101[.]99.94.116** to send mail. To confirm this, we can query the MX servers that **letsdefend.io** is actively using.

"mxtoolbox.com" shows:

![Phishing-2.png](/img/user/cards/red-team/images/Phishing-2.png)
The output tells that **letsdefend.io** uses Google addresses and shows no relation to **emkei[.]cz** or **101.99.94.116** resulting into email did not come from the original address (spoofed).

- Are the data "From" and "Return-Path / Reply-To" the same?

Except in exceptional cases, The `From` and `Return-Path` should be the same, in this scenario: 

![Phishing-3.png](/img/user/cards/red-team/images/Phishing-3.png)
The highlighted keys are different, this means that the email will go to the address found in `Reply-To` instead of the `From` in most cases should be a cause for concern especially to cases such as Invoicing, payment, bank or anything. Context is important.

## Conclusion


