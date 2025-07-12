---
{"dg-publish":true,"permalink":"/cards/red-team/process-hollowing/","tags":["red-team/host-evasion"]}
---

~ [[cards/red-team/Abusing Windows Internals\|Abusing Windows Internals]]
### Introduction
---
Allows the attackers to inject an entire malicious file into a process, this is accomplished by "hollowing" or un-mapping the process and injecting specific Portable Executable (PE) data and sections into the process.

Think of a process as a coconut - we scoop out the insides of the coconut and replace it with something else.

## At a high level
---

![Abusing Windows Internals2.png](/img/user/cards/red-team/images/Abusing%20Windows%20Internals2.png)

1. **Creating the Target Process (Suspended State)**

	- A **legitimate process (Suspended)** is created using the `CreateProcess` API with the `CREATE_SUSPENDED` flag. (Yellow Circle)
	- This paused process provides a _"shell"_ (legitimate process like `svchost.exe`) for later hollowing.
		- The "shell" is kept, this is the legitimate process name, PID, appearance but the contents is replaced. (remember the analogy)

2. **Reading the Malicious File**

	- The **Malicious Process** uses the `ReadFile` API to load a **Malicious File** (a custom PE executable) from disk. (Bottom Left Red Rectangle).
	- This file is stored in memory for later injection.

3. **Allocating Space in Local Memory**

	- The **Malicious Process** uses `VirtualAlloc` to allocate memory for the malicious file. (Lower path red rectangle the same line with `Malicious File`)
	- This memory stores the file in preparation for injection into the target process.
	 - Malicious Process (MP) -> Malicious File (MF) -> `VirtualAlloc` is used to make 'space' for the incoming file that we are going to read using `ReadFile` -> MP (Back to yellow circle) tells the 'target' process to Allocate memory for the MF -- the staged malicious file is copied into the target process using `WriteProcessMemory`.
	-  But before writing the MF we need to carve out the original code - Step 4 must happen first.

4. **Unmapping Legitimate Code**

	- The **Suspended Process's** memory contains legitimate sections: **DLLs**, **Registers**, **Process Heap**, **Thread Stack**, and a **Process Memory Region**.
	- To hollow it out, the attacker uses `ZwUnmapViewOfSection` to unmap (remove) the original executable from the **Process Memory Region**, resulting (Blue node) in a **Hollowed Memory Region**.

5. **Allocating Space in Target Process**

	- A new **Malicious Memory Region** is allocated in the suspended process using `VirtualAlloc` again.
	- This space is where the malicious code will be injected.

6. **Writing the Malicious Code**

	- The **Malicious File** is written into the **Malicious Memory Region** using `WriteProcessMemory`.
	    
7. **Adjusting Execution Context**

	- The attacker retrieves the thread’s current context using `GetThreadContext`, which includes the **EBX/EAX Register**.
	- The register (typically EAX for 32-bit or RCX for 64-bit) is updated to point to the **Malicious Memory Region’s** entry point in other words tell the Instruction pointer **(EIP for 32-bit or RIP for 64-bit)** to execute the malicious code when the suspended thread is resumed.
	- This update is applied using `SetThreadContext`.

8. **Resuming Execution**

	- The process is resumed using `ResumeThread`.
	- Now, instead of executing the original (legitimate) binary, the process starts executing the injected **Malicious File**.

Enumerating for process:

![Abusing Windows Internals (copy).png](/img/user/cards/red-team/images/Abusing%20Windows%20Internals%20(copy).png)

Then using the malicious binary

![Abusing Windows Internals-1 (copy).png](/img/user/cards/red-team/images/Abusing%20Windows%20Internals-1%20(copy).png)
