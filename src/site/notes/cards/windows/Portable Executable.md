---
{"dg-publish":true,"permalink":"/cards/windows/portable-executable/","tags":["windows"]}
---

~ [[atlas/windows\|windows]]
### Introduction
---
It is the standard file format for Windows executables such as `.exe`, `.dll`, `.sys`, and many more. It defines how binary code, libraries, and resources are organized on disk and loaded into memory by the Windows loader.

![Portable Executable.png|400](/img/user/cards/red-team/images/Portable%20Executable.png)
**PE Headers**

- **Image Base** the sport in the memory where the program _wants_ to live. (If the space is taken it moves to a different location to work.)
	- In the context of **host evasion** the default memory address for executable is 0x00400000 - malware authors changes this to avoid rules (pseudo code) such as: _"If the ImageBase is 0x00400000 and it contains Meterpreter shellcode, block it."_
	- The value of this header is also considered in hashing.
- **Entry Point** is the **first instruction (code)** that runs when a Windows executable (like `.exe`) is launched.

**Data Section (Sections)**:

- **`.text`**: Contains the **executable code** of the program.

- **`.data`**: Contains **initialized global and static variables**.

- **`.bss`**: Stores **uninitialized data** (declared variables with no assigned values)
    
- **`.rdata`**: Read-only data such as constants and import/export tables.
    
- **`.idata`**: Information about **imported functions and DLLs**.
    
- **`.edata`**: Contains information about **functions this binary exports**.
    
- **`.reloc`**: Used if the binary needs to be relocated from its preferred ImageBase.
    
- **`.rsrc`**: Resources like icons, images, dialogs, and embedded files.



### Why is PE important for Host Evasion?

Understanding PE structure allows red teamers and malware developers to:

- **Embed shellcode** into specific sections (.text, .data, .rsrc) to evade detection.
    
- **Modify headers** to alter execution behavior or spoof legitimacy.
    
- **Bypass AVs** by exploiting how they parse or scan PE files.
    
- **Craft loaders or packers** that execute payloads without triggering standard heuristics.