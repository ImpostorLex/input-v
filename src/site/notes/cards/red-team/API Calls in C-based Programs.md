---
{"dg-publish":true,"permalink":"/cards/red-team/api-calls-in-c-based-programs/","tags":["concept","red-team/concept"]}
---

~ [[cards/red-team/Signature Evasion#Behavioral Signatures\|back]]
### Introduction
---
1. The API needs two things, to use Windows API like `OpenProcess`:

	- The **address** of the function in memory (called a **pointer**)
	- And the **structure** that describes the function (data type, parameters, etc.).

2. These functions live in system libraries:

	- Functions such as `OpenProcess` live in DLLs such as `kernel32.dll` or `ntdll.dll`.
	- These DLLs export functions so programs can call them.

3. How Programs Know Where Functions are (Despite ASLR):

- Windows uses **ASLR** (Address Space Layout Randomization) — so function addresses change each run.
    
- So how does your program know where `OpenProcess` is? Answer is Windows Loader takes care of it using the _IAT_.

#### Import Address Table
---
The **IAT** is a table inside the binary’s Portable Executable (PE) file format.

- It holds **placeholders** for the addresses of imported functions.
    
- When the program loads, **Windows fills in the actual addresses** of the functions into the IAT.
    
- So later when your program calls `OpenProcess()`, it doesn’t look it up — it just calls the pointer in the IAT.

- The IAT lives in a section of the PE file under `IMAGE_OPTIONAL_HEADER`.

It works like this:

1. The Windows loader looks at the PE header.
    
2. It sees which DLLs and functions your binary wants.
    
3. It loads those DLLs into memory.
    
4. It looks up each function.
    
5. It fills in the **real memory addresses** into the IAT.
    
6. Your program can now call `OpenProcess` through that pointer.