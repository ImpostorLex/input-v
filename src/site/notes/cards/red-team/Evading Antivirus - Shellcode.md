---
{"dg-publish":true,"permalink":"/cards/red-team/evading-antivirus-shellcode/","tags":["red-team/host-evasion"]}
---

~ [[map-of-contents/red-team\|red-team]]
### Introduction 
---
This focuses on evading [[cards/red-team/How Antivirus works and gets bypassed#Features\|common Antivirus Engines]] found on most Windows system but some of the topics here can also be applied to Unix systems.
#### Key Topics
---
- [[#Prerequisites|Foundational understanding needed before diving into AV evasion.]]
- [[#Portable Executable|Understand PE structure, section roles, and how storing shellcode in specific sections impacts detection.]]

- [[#Shellcode|Define, generate, and compile raw shellcode — including examples with Metasploit.]]

- [[#Encoding and Encryption|Techniques to obfuscate or hide shellcode using encoding or cryptographic methods.]]

- [[cards/red-team/Packers Against Disk-Based AV Detection\|Use tools like ConfuserEx to obfuscate and compress .NET payloads to hinder static analysis.]]

- [[cards/red-team/Shellcode Encoding and Encrypting\|Detailed techniques on encoding/encryption specific to shellcode, including runtime decryption.]]

- [[cards/red-team/Staged and Stageless\|Explanation of how payload staging affects detection and shellcode structure.]]
- [[cards/red-team/Binders - malware development\|Combining legitimate applications with malicious payloads to deceive users and bypass casual inspection.]]

- [[cards/red-team/How Antivirus works and gets bypassed\|Understand how AV engines function and identify key bypass strategies (heuristic, behavioral, signature-based, etc).]]
## Prerequisites
---
- [[cards/red-team/How Antivirus works and gets bypassed\|How Antivirus works and gets bypassed]]
- [[cards/windows/Portable Executable\|Portable Executable]]
## Portable Executable
---
Read this first: [[cards/windows/Portable Executable\|Portable Executable]]

Antiviruses and malware analysts analyze `.exe` file based on the information found in the PE header and other PE sections.

Storing shell code in the Data section since we can control it:

**`.text` section**

- Defining the shellcode as a local variable within the main function will store it in the **.TEXT** PE section.  
- This is the **default code section**, so putting shellcode here **isn’t stealthy**.
- If analysts or AV scan the `.text` section and see suspicious byte patterns (like a shellcode decoder or syscall stub), it **gets flagged** - It’s like hiding a knife in a kitchen — it might blend, but it’s the first place they'll check.

**`.data` section**

- Defining the shellcode as a _global variable_ will store it in the **.Data** section.
- A **global variable** is a variable that is defined **outside** any function, which means it is **accessible from anywhere** in your code
- _global variable_ should not be confused with `global _start` (in the below example) keyword.

In Assembly example of defining a global variable:

```C
section .data
message db "Hello", 0x0A // In python = Hello\n where in 0x0A results into newline character.
```

- `message` is a **label** — kind of like a variable name
    
- `db`  define bytes: defines the actual data bytes in memory, you can use **normal text** and **byte values** at the same time provided above


**`.rsrc` section**

- Another technique involves storing the shellcode as a raw binary in an icon image and linking it within the code, so in this case, it shows up in the **.rsrc** Data section. 
- Very common AV bypass: **store shellcode as an image, then extract it at runtime.**
- Analyst might not check resource data deeply.

**`.custom` section**

- We can add a custom data section to store the shellcode.
- Often flags by analyst for unknown section names.
- Must encrypt or obfuscate your payload then only decode it in memory.
## Shellcode
---
Shellcode is a set of crafted machine code instructions that tell the vulnerable program to run additional functions and, in most cases, provide access to a system shell or create a reverse command shell.

read first: [[cards/red-team/shell code injection\|shell code injection]] (notes are transferred to their own section due to large text.)

- [[cards/red-team/Generating a simple shellcode\|Understanding how to create a basic shellcode]]
### Generating Shellcode
---
Metasploit or any C2 frameworks already provide shellcodes however they are well known by any Antivirus products.

Step 1: 
```C
msfvenom -a x86 --platform windows -p windows/exec cmd=calc.exe -f c
```

Step 2: Compile to `.exe` file:

```C
i686-w64-mingw32-gcc calc.c -o calc-MSF.exe
```

**Output:**

![Evading Antivirus-2.png|450](/img/user/cards/red-team/images/Evading%20Antivirus-2.png)


Shellcode can be stored as `.bin` files or raw binary files, we can get the shellcode via `xxd -i` command:

Step 1:
```shell-session
msfvenom -a x86 --platform windows -p windows/exec cmd=calc.exe -f raw > /tmp/example.bin
```

Step 2: 

```C
xxd -i /tmp/example.bin
```

The output should match the previous command, next up is delivering our shellcode to the victim: [[cards/red-team/Staged and Stageless\|Staged and Stageless]].
## Encoding and Encryption
---
**Encoding** is the process of changing the data from it's original state into a specific format depending on the algorithm used such as Base64 while **encryption** is the process of changing the data into unreadable value that requires knowing the algorithm used and the encryption key.

**Why is it important?**

As said before some AV uses pattern detection or any algorithm with the sole purpose of knowing what's malicious or not. However encoding is not enough for evasion purposes as AV software is more intelligent and can analyze binary, decode an encoded string.

- Two or more encoding to make it harder for the AV.
- As for encryption well you need a good algorithm and a secret key (hardcoded secret key is not ideal must be produced in a code -- to make it harder for blue team to analyze it).

Read this before continuing [[cards/red-team/Shellcode Encoding and Encrypting\|Shellcode Encoding and Encrypting]].

Next up:

- [[cards/red-team/Packers Against Disk-Based AV Detection\|Packers Against Disk-Based AV Detection]]
- [[cards/red-team/Binders - malware development\|Binders - malware development]]
## Questions and Problems
---

> [!NOTE]-  Antivirus and malware analysts could easily view this with debugging tools, why this exactly?
>Question related to [[#Portable Executable|Portable Executable]] section
> 
> It's a cat and mouse game, it's a game of **skills**, **time**, and **imagination**.
## Conclusion


