---
{"dg-publish":true,"permalink":"/cards/red-team/runtime-detection-evasion/","tags":["red-team/host-evasion"]}
---

~ [[map-of-contents/red-team\|red-team]]
### Introduction 
---
With the release of PowerShell, Microsoft released [[cards/red-team/AMSI (Anti-Malware Scan Interface)\|AMSI (Anti-Malware Scan Interface)]].

Runtime detection measure intercepts and takes a look at the malicious code then it looks for indicators of malicious code before allowing the execution of the script.
#### Key Topics
---
- [[#Runtime Detections|Explains how code is scanned before execution in memory or the runtime and how this differs from standard antivirus scanning.]]
    
- [[#AMSI|Overview of the Anti-Malware Scan Interface, its workflow, integration with interpreters like PowerShell, and interaction with providers such as Windows Defender.]]
    
- [[#PowerShell Downgrade|Technique for bypassing PowerShell security by switching to version 2.0, which lacks modern protections, and mitigation methods.]]
    
- [[#PowerShell Reflection|Use of .NET reflection to access and modify internal PowerShell classes and fields to disable AMSI scanning.]]
    
- [[#Patching AMSI|Direct memory patching of amsi.dll’s AmsiScanBuffer function to always return a clean scan result.]]
    
- [[#Automation|Automated AMSI bypass approaches such as amsi.fail and amsiTrigger to identify and modify flagged signatures.]]
## Runtime Detections
---
When executing code or applications, it will almost always flow through a [[cards/red-team/runtime\|runtime]], no matter the interpreter. This is most commonly seen when using Windows API calls and interacting with .NET.

A runtime detection measure will scan code before execution in the runtime, the detection measure applied depends on the technology used. **If the code is suspected of being malicious**, it will be assigned a value, and if within a specified range, the execution will stop, quarantine, or delete the file.

Its different from a standard antivirus as it scans directly from memory and the runtime, at the same time anti-virus products can also employ these runtime detections to give more insights.
## AMSI
---
Read: [[cards/red-team/AMSI (Anti-Malware Scan Interface)\|AMSI (Anti-Malware Scan Interface)]]
### How AMSI works and instrumented
---
Its an interface that acts as a bridge between applications (like PowerShell, VBScript, or Office macros) and anti-malware solutions (like Windows Defender or a third-party AV).

**How AMSI Works:**
![AMSI (Anti-Malware Scan Interface).png](/img/user/cards/red-team/images/AMSI%20(Anti-Malware%20Scan%20Interface).png)

**1. PowerShell / VBScript / Other Application**  
These are the interpreters or applications executing scripts or commands. These components call the AMSI API functions.

**2. AMSI API**

- Functions like `AmsiScanBuffer()` and `AmsiScanString()` are used to send code/content to AMSI for scanning.
- These are implemented in `amsi.dll`, using headers like `amsi.h`.

**3. AMSI COM Interface** `IAntimalware::Scan()`

The API calls are abstracted into a COM interface for flexibility and interaction with various providers. COM is a flexible way to define interaces in Windows, allowing different components such as antivirus products to interact with AMSI without needing to modify the API code itself.

In other words, it allows different components to make use of AMSI features.

**4. Provider Registration Layer**

- This is where antivirus engines register themselves as AMSI providers. 

Whole flow:

```C
Script runs in an interpreter (e.g., PowerShell)
      ↓
Interpreter submits code to AMSI using AmsiScanBuffer()
      ↓
AMSI calls the registered scan engine (e.g., Defender or McAfee)
      ↓
AV engine analyzes the code and returns a scan result
      ↓
AMSI sends result back to the interpreter (e.g., clean, malware)
      ↓
Interpreter acts accordingly (e.g., executes or blocks script)
```

- **Windows Defender** is the default provider, but **third-party AVs** can also plug into AMSI here.

**5. AV Provider Class**

- Windows Defender uses `MpEngine.dll` (Scan Engine) and `MpSvc.dll` (RPC Server).

	- `MpEngine.dll`: So when AMSI wants to scan a suspicious script, and Defender is the registered provider, **`MpEngine.dll` will do the actual malware detection**.
	- **MsMpEng.exe** (Windows Defender’s main process) runs with higher privileges in a protected context. To allow other processes (like PowerShell or Excel) to **send scan requests** to it securely, Windows uses **RPC (Remote Procedure Call)**. this is the purpse of this .dll

- Third-party providers can also hook in at this stage with their scanning engines.

**6. RPC (Remote Procedure Call) Layer**

This allows cross-process communication. When a scan is triggered, data may be sent to another process (like MsMpEng.exe for Defender) for analysis.

Breaking it down into core components:

![AMSI (Anti-Malware Scan Interface)-1.png](/img/user/cards/red-team/images/AMSI%20(Anti-Malware%20Scan%20Interface)-1.png)

**Note**: AMSI is only instrumented when loaded from memory when executed from the **Common Language Runtime** (execute .NET apps such as powershell). It is assumed that if on disk MsMpEng.exe (Windows Defender) is already being instrumented.

```cpp
var scriptExtent = scriptBlockAst.Extent;
 if (AmsiUtils.ScanContent(scriptExtent.Text, scriptExtent.File) == AmsiUtils.AmsiNativeMethods.AMSI_RESULT.AMSI_RESULT_DETECTED)
 {
  var parseError = new ParseError(scriptExtent, "ScriptContainedMaliciousContent", ParserStrings.ScriptContainedMaliciousContent);
  throw new ParseException(new[] { parseError });
 }

 if (ScriptBlock.CheckSuspiciousContent(scriptBlockAst) != null)
 {
  HasSuspiciousContent = true;
 }
```

This 12 lines of code basically represent PowerShell being connected to AMSI which in this case the default provider is Windows Defender.
## PowerShell Downgrade
---
It is an attack (low hanging fruit) that allows attackers to modify the current PowerShell version to remove security features.

PowerShell sessions will start with the most recent PowerShell engine (most of them) but the attacker can manually change them to 2.0 as security features were implemented in version 5.0, we can bypass this with one liner.

Launch PowerShell Version 2:

```powershell
PowerShell -Version 2
```

Or much more (not really) stealthier version:

```C
full_attack = '''powershell /w 1 /C "sv {0} -;sv {1} ec;sv {2} ((gv {3}).value.toString()+(gv {4}).value.toString());powershell (gv {5}).value.toString() (\''''.format(ran1, ran2, ran3, ran1, ran2, ran3) + haha_av + ")" + '"'
```

**Detection and Mitigations:**

1. Remove PowerShell 2.0 engine from device
2. Deny access to PowerShell 2.0 via application blocklisting.

## PowerShell Reflection
---
**Reflection** is a feature in .NET that allows code to **inspect, interact with, and even modify** assemblies (compiled code), types, methods, and properties at runtime.

A **.NET assembly** is just a compiled piece of code in formats like:

- `.exe` – Executable files
    
- `.dll` – Dynamic Link Libraries

These are **the core building blocks of .NET applications** — they contain the types, methods, and resources needed for the application to run.

In short: assemblies are packaged code libraries that .NET uses to run programs securely and modularly.

**So whats the big deal?**

**PowerShell**, being built on top of .NET, **can use reflection** to do things like:

- Look at which methods exist in a DLL.
- Access private or internal methods (not normally accessible).
- Modify or override functionality at runtime.

Resulting into attackers:

- Bypassing security features, like AMSI.
- Invoke hidden methods in DLLs.

A one liner to do this:

```powershell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

- `[Ref].Assembly` gives us the ability to interact with **loaded .NET classes**, including internal/private ones.

- `G.etType('System.Management.Automation.AmsiUtils')` -
	- This looks into the loaded assembly for the **class** `System.Management.Automation.AmsiUtils`.
    - This class handles how **PowerShell connects to AMSI**.
    - Specifically, it contains **internal fields** that indicate whether AMSI is active or failed.

- `.GetField('amsiInitFailed','NonPublic,Static')` You are accessing a private setting that **disables AMSI silently**.

- `.SetValue($null,$true)` PowerShell will **think AMSI is not available**, and it will **not scan scripts or commands for malware**.

## Patching AMSI
---
Unlike previous technique were we disable AMSI this time we are going straight into memory and rewriting AMSI's scanning function so it always returns a "clean" result.

**1. Obtain handle of** `amsi.dll`

- `amsi.dll` is the actual DLL file where AMSI’s scanning functions live.

- When PowerShell starts, it loads `amsi.dll` into its process memory.

- We can ask Windows: _“Hey, what’s the memory handle for `amsi.dll`?”_

- This is done with `GetModuleHandle("amsi.dll")`.

The goal is to know where in memory `amsi.dll` so we can tamper it.

**2. Get the memory address of** `AmsiScanBuffer`

- `AmsiScanBuffer` is the function inside `amsi.dll` that receives suspicious script data, scans it, and returns a detection result.
    
- We use `GetProcAddress()` to find _exactly where in memory_ this function’s machine code lives.

We can modify or tell this function instead of scanning just say 'its all good'.

**3. Change memory protection**

- Normally, Windows mark code in memoy as read-only for safety.
- The goal is to change AMSI's function memory protection settings.
- This is where `VirtualProtect()` comes in — it changes the region of memory where `AmsiScanBuffer` is stored to **read/write/execute** mode.

**Overwrite the function with our "clean" response**

- The real trick: instead of letting `AmsiScanBuffer` scan anything, we overwrite its first few instructions with new opcodes (machine code).
    
- These opcodes force the function to **always return `AMSI_RESULT_CLEAN` (0)** no matter what’s scanned.
    
The patch buffer in the example:

```cpp
$buf = [Byte[]](
    0xB8, 0x57, 0x00, 0x07, 0x80,  # MOV EAX, 0x80070057  (clean return code)
    0xC3                            # RET
);
```

- `0xB8` — opcode for _“move this number into EAX”_ (EAX is the register where return values go)

- `0x80070057` — the constant for "all good, nothing detected"
    
- `0xC3` — opcode for _“return from function”_


[[cards/red-team/Runtime Detection Evasion Code Blocks\|Runtime Detection Evasion Code Blocks]] see script here

## Automation
---
Its preferred to use the previous methods shown but if automation is a must:

[amsi.fail]

[amsiTrigger] allows attackers to automatically identify strings that are flagging signatures to modify and break them. This method of bypassing AMSI is more consistent than others because you are making the file itself clean.



### Questions and Problems
---
## Conclusion


