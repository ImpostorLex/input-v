---
{"dg-publish":true,"permalink":"/cards/blue-team/endpoint-security/challenges/countdown/"}
---

[[cards/blue-team/endpoint-security/Endpoint Security\|Endpoint Security]] | ~ [[map-of-contents/blue-team\|blue-team]]

> Verify the Disk Image. Submit SectorCount and MD5

A `.eo1` image is provided, we can provide this to autopsy.

![Countdown.png](/img/user/cards/blue-team/endpoint-security/images/Countdown.png)
> What is the decryption key of the online messenger app used by Zerry

Run Programs -> signal.exe as well as using the "messenger" keyword: two showed up signal.exe and AOL Instant messenger:

![Countdown-1.png](/img/user/cards/blue-team/endpoint-security/images/Countdown-1.png)
**Based on** google result with search term "Signal decryption key", it shows this

Then navigate using the partitions:

![Countdown-2.png](/img/user/cards/blue-team/endpoint-security/images/Countdown-2.png)

c2a0e8d6f0853449cfcf4b75176c277535b3677de1bb59186b32f0dc6ed69998

> What is the registered phone number and profile name of Zerry in the messenger application used?

![Countdown-3.png](/img/user/cards/blue-team/endpoint-security/images/Countdown-3.png)
Then use the decryption key but first extract the files and open it via SQLbrowser.

raw key -> 0xkeyhere

conversations -> result here.

> What is the email id found in the chat?

messagse > eekurk[@]baybabes.com

> What is the filename(including extension) that is received as an attachment via email

Downloads.... or recent documents since most likely attachments will be downlaoded:

`C:\Users\ZerryD????\Downloads\???.PNG	`
however you need to convert the icon:

![Countdown-4.png](/img/user/cards/blue-team/endpoint-security/images/Countdown-4.png)
> What is the Date and Time of the planned attack?_(7 points)_

2020 - thumbcache viewer.

01-02-2021 PM