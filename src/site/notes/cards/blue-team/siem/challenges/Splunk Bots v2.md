---
{"dg-publish":true,"permalink":"/cards/blue-team/siem/challenges/splunk-bots-v2/"}
---

~ [[map-of-contents/blue-team\|blue-team]] 

- August 24th - ransomware incident through USB drive. (Year 2016 per question)
- Victim: Bob Smith
- Workstation: `we8105desk`
- Word Document: `Miranda_Tate_unveiled.dotm`.

Knowing our sources:
```C
| metadata type=sourcetypes
| fields sourcetype
```

**Notable sources are:**
- fgt*
- nessus
- Sysmon
- stream:*
- suricata

---
Viewing the top talkers:
```C
index=* | stats count by src_ip | sort -count
```

![Splunk Bots v2.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v2.png)
Since we know the workstation name of the victim, let's determine Bob's IP address:

```C
index="botsv1" we8105desk
```

- Ideally if Sysmon is available we can check for EventCode: 3 for network connection and include  the workstation name.

==Bob's IP address: 192.168.250.100==

Viewing the `src_ip` field shows thousands of events belong to the mentioned IP address and viewng the `app` field and clicking the `src_ip` shows the same IP address:

![Splunk Bots v2-1.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v2-1.png)
> Amongst the Suricata signatures that detected the Cerber malware, which one alerted the fewest number of times? Submit ONLY the signature ID value as the answer.

2816763

```C
index="botsv1" sourcetype="suricata" eventtype=suricata_eve_ids_attack "alert.signature"="*"  | stats by alert.signature 
```

Viewing the `alert.signature` field in a table then counting via `stats` command shows the least Cerber malware alert:

![Splunk Bots v2-2.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v2-2.png)
Then viewing one of it's event to determine the attack signature ID.

> What fully qualified domain name (FQDN) does the Cerber ransomware attempt to direct the user to at the end of its encryption phase?

- Sysmon EventID: 22 for dns query is useful as well. (not available since the OS is most likely windows 7)

```C
index=botsv1 sourcetype="stream:dns" src_ip=192.168.250.100 "query_type{}"=A | table queries
```

Out of the result this is the domain that most stand out and VirusTotal flagged it as malicious: (cerberhhyed5frqa.xmfir0.win)

- "At the end of encryption phase" we can sort this via `| sort by -_time` reverse older events go first but in this case it is simple.

![Splunk Bots v2-7.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v2-7.png)
plus if you look at the first bit of the domain name it starts with 'cerber'.

> What was the first suspicious domain visited by we8105desk on 24AUG2016?

solidaritedeproximite.org using the same query as the above - ==it does not mean VT did not flagged it does not mean it is not the answer.==

- It is the second suspicious domain, why contact another `.org`? 
- If you look at the time this is the first suspicious domain visited.

My initial assumption is most likely going to be a website and it will make a GET request but there is a lot of ways to contact a domain without using a browser, so the best way is to query for dns queries.

> During the initial Cerber infection a VB script is run. The entire script from this execution, pre-pended by the name of the launching .exe, can be found in a field in Splunk. What is the length of the value of this field?

- Sysmon EventID 1 is good here if we don't know the file name or extension.

Since we already know that the executed document has a macro indicated by the `m` in `.docm` we can query via the malicious filename and look at the cmdline:

```bash
index="botsv1" Miranda_Tate_unveiled.dotm | table cmdline | eval cmd=len(cmdline)
```

![Splunk Bots v2-3.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v2-3.png)
> What is the name of the USB key inserted by Bob Smith?

- Registry "Artifacts"
- Then friendlyname keyword and action=modifieid


> Bob Smith's workstation (we8105desk) was connected to a file server during the ransomware outbreak. What is the IPv4 address of the file server?

445 for smb

```C
index="botsv1" we8105desk dest_port=445
```

Why is victim IP address as `dest_port` ? because communication works back and forth the same as http protocol.

![Splunk Bots v2-4.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v2-4.png)
> How many distinct PDFs did the ransomware encrypt on the remote file server?

- Windows event logs are the way to go for this one not the smb protocol as event logs are much more verbosed
- Including `dest_ip` and `src_ip` ideal

Starting with this filter and then looking at the event code, we can see 5145 meaning it checkes whether the client have enough permission to access the object:
```bash
index=botsv1 sourcetype="WinEventLog:*" "*.pdf" 
```

Then viewing one of the events, I noticed the fileshare path then added it as filter:

![Splunk Bots v2-8.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v2-8.png)
```C
index=botsv1 sourcetype="WinEventLog:*" "*.pdf" Share_Path="\\??\\C:\\fileshare"
```

Then at the sidebar I noticed this field, these are access rights the first two terms are self explanatory, the `createpipeinstance` is commonly used for communication :

![Splunk Bots v2-9.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v2-9.png)
```C
index=botsv1 sourcetype="WinEventLog:*" "*.pdf" Share_Path="\\??\\C:\\fileshare" AppendData__or_AddSubdirectory_or_CreatePipeInstance_="Granted by	D:(A;OICI;FA;;;WD)"
```

Then the returned events are **257**.

> The VBscript found in question 204 launches 121214.tmp. What is the ParentProcessId of this initial launch?

- Add some fields including **image**, **pid**, **ppid**, and **cmdline** and use `sort _time`

Since Sysmon is available, it can log almost everything including process creation:
```C
index=botsv1 sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" 121214.tmp EventCode=1
```

The search query will return 7 events and starting from the bottom viewing the last event shows this:

![Splunk Bots v2-10.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v2-10.png)
We can see that the image **cmd.exe** started **121214.tmp** making it as it's parent process id.

> The Cerber ransomware encrypts files located in Bob Smith's Windows profile. How many .txt files does it encrypt?

- Sysmon Event ID 2 - a process modified a file.

Windows Profile are located on `C:\Users\bob.smith`: so I used the workstaion name and the `.txt` keyword:

```C
index=botsv1 sourcetype="*" we8105desk "*.txt" 
```

I noticed that there is a large amount of events by the `osk.exe` to `.txt` files:

![Splunk Bots v2-5.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v2-5.png)
So I used the `app` value seen in the above image as filter and at the sidebar `file_path` field:

![Splunk Bots v2-6.png|350](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v2-6.png)
So I removed every `.txt` file that does not start with bob smith windows profile:

```C
index=botsv1 sourcetype="*" we8105desk "*.txt" app="C:\\Users\\bob.smith.WAYNECORPINC\\AppData\\Roaming\\{35ACA89F-933F-6A5D-2776-A3589FB99832}\\osk.exe" NOT "C:\\Sysmon\\" | table file_path
```

Then got the answer of `406`.

> The malware downloads a file that contains the Cerber ransomware cryptor code. What is the name of that file?

- It's a good idea to use the recently found domain as `dest_ip`. (in this case it's not, a technique to make the domain not malicious or 'fill the logs' type)

```C
index=botsv1 sourcetype="suricata" src_ip="192.168.250.100" url="*" | table dest, dest_ip, url
```

The most suspicious and standing out is the unresolved IP address:

![Splunk Bots v2-11.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v2-11.png)
VirusTotal flagged **92.222.104.182** as malicious while the **solidarite...org** is clean:

![Splunk Bots v2-12.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v2-12.png)
> Now that you know the name of the ransomware's encryptor file, what obfuscation technique does it likely use?

Steganography






