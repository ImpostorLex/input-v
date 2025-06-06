---
{"dg-publish":true,"permalink":"/cards/windows/windows-api/","tags":["ad/windows"]}
---

~ [[map-of-contents/windows\|windows]]
### Introduction
---
The Windows API allows programs, or functions to interact with key components of the Windows Operating system, it is used by all red teamers, threat actors, blue teamers, software developers, and solution providers.
#### Key Topics
---

- [[cards/windows/Windows API#Subsystem and Hardware Interaction\|How Win32 API enables interaction with subsystems and hardware through user-mode and kernel-mode.]]
    
- [[cards/windows/Windows API#Components of the Windows API\|Overview of the key components like header files, core DLLs, and how API calls are structured.]]
    
- [[cards/windows/Windows API#OS Libraries\|Explaining how the Windows header file and P/Invoke help with address resolution in the context of ASLR.]]
    
- [[cards/windows/Windows API#API Call Structure\|Details on the naming conventions and character encoding distinctions in Win32 API calls.]]
    
- [[cards/windows/Windows API#Commonly Abused API calls\|Examples of commonly abused API calls and their potential malicious use.]]

## Subsystem and Hardware Interaction
---
Microsoft released Win32 API to allow programs to access or modify subsystem or hardware using two modes user-mode applications and the kernel mode rather than allowing any program to access any subsystem or hardware that may cause machine instability.

| **User mode** -- _where apps run with limited privileges and isolation to protect the system._ | **Kernel mode** -- _is where the OS itself runs, with full access to the hardware and memory_ |
| ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| No direct hardware access - access. Must go through APIs (e.g., Win32).                        | Direct hardware access                                                                        |
| Access to "owned" memory locations                                                             | Access to entire physical memory                                                              |

Visual representation of how a user applicaiton uses API calls to modify kernel components:

![Pasted image 20250601092123.png|300](/img/user/cards/windows/images/Pasted%20image%2020250601092123.png)

Running app as administrator does not open in **kernel** mode.
## Components of the Windows API
---
Assuming the API is the top layer and the parameters that make up a specific call is the bottom layer:

| **Layer**               | **Explanation**                                                                                                                                                                                                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| API                     | A top-level/general term or theory used to describe any call found in the win32 API structure. It stands for _Application Programming Interface_ allows program to use another's service/program's functionality.                                                                           |
| Header files or imports | Defines libraries to be imported at run-time, defined by header files or library imports. **Uses pointers to retrieve the memory address of a function that has been loaded into the program’s memory space**.                                                                              |
| Core DLLs               | These four DLLs (KERNEL32, USER32, and ADVAPI32) help programs interact with the operating system by providing a set of **core functions** that aren’t all bundled into one big chunk (subsystem), but are separated based on their functionality (system, user interface, security, etc.). |
| Supplemental DLLs       | Other DLLs defined as part of the Windows API. Controls separate subsystems of the Windows OS. ~36 other defined DLLs. (NTDLL, COM, FVEAPI, etc.)                                                                                                                                           |
| Call Structures         | Defines the API call itself and parameters of the call.                                                                                                                                                                                                                                     |
| API Calls               | The API call used within a program, with function addresses obtained from pointers. the part where the pointer retrieves the address of the WANTED API.                                                                                                                                     |
| In/Out Parameters       | The parameter values that are defined by the call structures. the required/optional parameters in order for the API                                                                                                                                                                         |
## OS Libraries
---
In Windows determining the address of an API's function is impossible thanks to Address Space Layout Randomization (ASLR) implementations.

Thankfully P/Invoke and the Windows header file exists.
### Windows Header File
---
Once the `windows.h` header is included in an unmanaged program (C, C++, Assembly), it allows access to all Win32 API functions. The program can make API calls, but at runtime, the Windows loader resolves the function addresses using the Import Address Table (IAT). If ASLR is enabled, the loader randomizes the memory addresses of the DLLs, and pointers to the functions are adjusted accordingly.

### P/Invoke
---
- **P/Invoke (Platform Invocation)** is a feature in .NET that allows **managed code** (e.g., C#) to call **unmanaged functions** in native libraries (like Win32 APIs) written in languages like C or C++.
- The **runtime** takes care of calling the unmanaged functions, so you don’t have to worry about memory management or low-level details.

#### It works like this
---

- **Import the DLL**:    
    - Use the `[DllImport]` attribute to specify the DLL and configure options (like character encoding or error handling).
    - Example:
    
```C#
[DllImport("user32.dll", CharSet = CharSet.Unicode, SetLastError = true)]
```
    
- **Define the Managed Method**:
    - The `extern` keyword tells the runtime this method exists outside the current code and will link to the unmanaged DLL.
    - Example:
    
```C
private static extern int MessageBox(IntPtr hWnd, string lpText, string lpCaption, uint uType);
```
    
- **Call the Unmanaged Function**:
    - You can now call `MessageBox` as if it's a normal C# method, but it actually calls the unmanaged `MessageBox` function from the `user32.dll`.

## API Call Structure
---
Microsoft uses specific characters in the names of API calls to distinguish between different types of encoding and functionality. Here's what each character means:

|**Character**|**Explanation**|
|---|---|
|**A**|**ANSI encoding**: This indicates that the function uses an **8-bit character set** for text (older, simpler encoding).|
|**W**|**Unicode encoding**: This indicates that the function uses **Unicode encoding** for text, which supports a wider range of characters (more modern, international).|
|**Ex**|**Extended functionality**: This shows that the function has **extended functionality**, like handling more complex input/output (I/O) parameters or providing extra options.|
### Why it matters
---
For example, a function like `MessageBox` might have both an ANSI version (8-bit) and a Unicode version (16-bit) depending on how the program handles text. This is why you might see two versions of the same function:

- `MessageBoxA` — Uses **ANSI** encoding (older, simpler text).
- `MessageBoxW` — Uses **Unicode** encoding (modern, supports more characters).

The **Ex** version might indicate an enhanced version of a function, such as adding extra parameters or options.

Each API call has specific parameters that tell it what to do, these parameter can be:

- **Input parameters [in]**: Data that the function needs to do its work (e.g., a file path to open).
- **Output parameters [out]**: Data that the function will return or modify (e.g., how many bytes were written to a file).

```C
BOOL WriteProcessMemory(
  [in]  HANDLE  hProcess,                  // Input: Handle to the target process
  [in]  LPVOID  lpBaseAddress,             // Input: Address in the target process to write to
  [in]  LPCVOID lpBuffer,                  // Input: Data to write
  [in]  SIZE_T  nSize,                     // Input: Number of bytes to write
  [out] SIZE_T  *lpNumberOfBytesWritten    // Output: Number of bytes actually written
);
```

## Commonly Abused API calls
---
Several entities have attempted to document and organize all available API calls with malicious vectors, including [SANs](https://www.sans.org/white-papers/33649/) and [MalAPI.io](http://malapi.io/).

|   |   |
|---|---|
|**API Call**|**Explanation**|
|LoadLibraryA|Maps a specified DLL  into the address space of the calling process|
|GetUserNameA|Retrieves the name of the user associated with the current thread|
|GetComputerNameA|Retrieves a NetBIOS or DNS  name of the local computer|
|GetVersionExA|Obtains information about the version of the operating system currently running|
|GetModuleFileNameA|Retrieves the fully qualified path for the file of the specified module and process|
|GetStartupInfoA|Retrieves contents of STARTUPINFO structure (window station, desktop, standard handles, and appearance of a process)|
|GetModuleHandle|Returns a module handle for the specified module if mapped into the calling process's address space|
|GetProcAddress|Returns the address of a specified exported DLL  function|
|VirtualProtect|Changes the protection on a region of memory in the virtual address space of the calling process|






















