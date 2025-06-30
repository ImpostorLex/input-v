---
{"dg-publish":true,"permalink":"/cards/windows/windows-internals/","tags":["windows","red-team/windows/ad/host-evasion"]}
---

~ [[map-of-contents/windows\|windows]]
### Introduction
---
To be a better red-teamer, it's important to understand what is Windows Internals and how they can be (ab)used such as using it to evade security controls and crafting offensive tools for exploitation.
### Key Topics
---
- [[#Processes|Processes – Structure of a process, its components, and why it matters for execution]]
- [[#Threads|Threads – How threads control execution and how they are manipulated in attacks]]
- [[#Virtual Memory|Virtual Memory – Isolation, memory management, and its role in injection techniques]]
- [[#DLL|DLL – How DLLs are loaded, and common abuse techniques like injection and hijacking]]
### Prerequisites
---
- Windows Fundamental
## Processes
---
It represents an execution of a program, an application or program can contain one or more processes and the process provides the resources needed to execute a program.

~ related [[cards/blue-team/endpoint-security/Windows Process Analysis\|Windows Process Analysis]] and how Windows internals is abused [[cards/red-team/Abusing Windows Internals\|here]].

| **Process Component**                              | **Description**                                                                                                                                                                                               |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [[#Virtual Memory\|Private Virtual Address Space]] | Memory allocated exclusively for the process, including code, data, stack, etc.                                                                                                                               |
| Executable Program                                 | The compiled code and data loaded into memory for execution.                                                                                                                                                  |
| Open Handles                                       | References to system resources like files, sockets, or registry keys. basically a handle to system resources accessible by the program. In python example file handle such as `open("example.txt, "r")` code. |
| Security Context                                   | Access token defining user identity, groups, and privileges.                                                                                                                                                  |
| Process ID (PID)                                   | Unique number assigned by the OS to identify the process.                                                                                                                                                     |
| Threads                                            | Smallest unit of execution within a process -- basically allows multiple functions/instructions to be executed by the CPU at the same time inside one process - shares the same memory and resource.          |
## Threads
---
- It's a smallest unit of execution within a process, it represents an instruction that can be scheduled to be executed by the CPU.
- Threads can execute any part of the code that belongs to a process this allows threads to execute different functions or procedures at the same time.

**Instruction Execution** (How threads control the execution of a process)

- Threads hold the **CPU context**, including:

    - Instruction Pointer (what to execute next)
    - Registers (temporary data)
    - Stack (function calls, local variables)

This means the thread literally **controls what code is being executed at any given moment**.

**Thread Context**

- Each thread has its **context** (CPU state), which can be paused and resumed by the OS.
- When a thread is scheduled out, its context is saved; when it’s scheduled back in, that context is restored.

```C
Process -> contains code
↓
Thread -> picks code to execute
↓
OS Scheduler -> assigns CPU time to the thread
↓
CPU -> executes instructions based on thread's context
```

In ProcMon:

![Pasted image 20250531094103.png](/img/user/cards/windows/images/Pasted%20image%2020250531094103.png)

**Security Analysis & Thread Abuse**

- Malicious code often uses thread manipulation to execute payloads (e.g., via `CreateRemoteThread`, `NtCreateThreadEx`).
- Threads can be injected into running processes to evade detection.
- Attackers often abuse thread creation APIs and use thread context manipulation (like `SetThreadContext`) to hijack execution flow.

## Virtual Memory
---
Virtual memory is a **core part of how Windows handles memory**. It gives each process its own **private address space**, so it *appears* like it has full access to memory — but without actually letting one process interfere with another.

This makes memory usage **safer**, **more efficient**, and **more flexible**.

#### How Virtual Memory Works
---
- Every process gets its **own virtual address space** — completely isolated from others.

- The **Memory Manager** handles translating these virtual addresses into actual **physical RAM** or **disk** if needed.
	- When Process A writes to memory address `0x12340000`, and Process B writes to the _same_ address — they’re actually writing to **different physical locations**.

- Windows uses **paging** to swap memory between RAM and disk when necessary, basically move infrequently used parts of a program's virtual memory from RAM to disk, freeing up RAM for active processes while preserving data for later use.

- This abstraction keeps processes from overwriting each other’s memory. 

**Injection, Execution, and Evasion:**

- Process injection targets a process’s **virtual memory space**.
- Shellcode gets written to **virtual memory** before execution.
- APIs like `VirtualAllocEx` and `WriteProcessMemory` directly interact with this space often times requiring elevated priviliges (not just being admin) specific privileges such as `SeDebugPrivileges` for dumping `lsass.exe`
## DLL
---
Stands for Dynamic Link Library, a library that contains code and data that can be used by one or more program at the same time. They help modularize programs and reduce redundancy.

When a DLL is loaded into a process, the application becomes **dependent on that DLL**. If attackers can control or manipulate that DLL, they may be able to:

- Hijack the application’s execution flow
- Load **malicious code** instead of the legitimate DLL
- Inject themselves into a **trusted process**

**Common DLL-Based Attack Techniques**

| Technique | MITRE ID | Description |
|----------|----------|-------------|
| **DLL Hijacking** | T1574.001 | Replaces a missing or mislocated DLL with a malicious one by placing it in a location where the app loads it first. |
| **DLL Side-Loading** | T1574.002 | Places a malicious DLL alongside a legitimate executable that **expects** a DLL in that location. |
| **DLL Injection** | T1055.001 | Injects a DLL into the memory space of another process to execute malicious code under that process’s context. |
They are loaded into a program:

**Load-Time Dynamic Linking**

```cpp

#include "sampleDLL.h"

int main() {
    HelloWorld(); // Calls DLL function at runtime, linked at compile time
    return 0;
}
```


**Run-Time Dynamic Linking**

```cpp
#include <windows.h>

typedef void (*DLLFunc)();
HINSTANCE hDLL = LoadLibrary("sampleDLL.dll");

if (hDLL) {
    DLLFunc HelloWorld = (DLLFunc)GetProcAddress(hDLL, "HelloWorld");
    if (HelloWorld) HelloWorld();
    FreeLibrary(hDLL);
}
```

ProcMon:

![Pasted image 20250531101852.png](/img/user/cards/windows/images/Pasted%20image%2020250531101852.png)





