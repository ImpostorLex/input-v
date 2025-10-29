---
{"dg-publish":true,"permalink":"/cards/blue-team/digital-forensics/windows-program-execution-artifacts-lnk-prefetch-jump-lists/"}
---

[[atlas/blue-team\|blue-team]]
### Introduction
---
These are other ways to determine what files are executed and how many times other than viewing the [[cards/blue-team/digital-forensics/Common Windows Forensics Artifacts#Program Execution\|registry]].

- `.lnk` has a specific signature of (4C 00 00 00) and has metadata of creation, modification, access time, and local path.
-  `.pf` - **prefetch files** are frequently used files to speed up loading, it includes all necessary dependency such as `.dll` and more.
- **jump list** are recently accessed files from the taskbar and start menu

#### Key Topics
---

- **Program Execution Artifacts (non-registry)** — identify files executed and frequency without relying on registry keys.
    
- **.LNK Files (Shortcuts)** — reveal metadata (created, modified, accessed times, local paths, even deleted files).
    
- **Prefetch Files (.pf)** — system-wide artifacts showing execution counts, dependencies (e.g., `.dll`), and binary locations.
    
- **Rogue Binary Detection** — compare legitimate and copied executables via prefetch data.
    
- **Jump Lists** — track recently accessed or pinned files via `AutomaticDestinations` and `CustomDestinations`.

## Investigations
---
**Recent Items (.lnk)** will include recent files **even deleted files** and their local path, we can view this via:

```C
%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Recent Items
```

Then we can use `exiftool` against a directory or specific filename to view the metadata:

```Javascript
exiftool(-k).exe C:\Users\tcm\AppData\Roaming\Microsoft\Windows\Recent Items\DeleteMe
```

![LNK, Prefetch Files, and Jump Lists.png](/img/user/cards/blue-team/digital-forensics/images/LNK,%20Prefetch%20Files,%20and%20Jump%20Lists.png)
[exiftool output]: 

**Prefetch** are files on disk meaning it is system wide:

```C
C:\Windows\Prefetch
```

The format consist of the application name followed by a hash, and the hash value is prefixed and determined via disk location, this is a **great way to determine rogue binaries.**
![LNK, Prefetch Files, and Jump Lists-1.png](/img/user/cards/blue-team/digital-forensics/images/LNK,%20Prefetch%20Files,%20and%20Jump%20Lists-1.png)

We can use `PECmd.exe` followed by a `-f` for file or `-d` switch for directory and see output of file run last and other run times, and many more:

![LNK, Prefetch Files, and Jump Lists-2.png](/img/user/cards/blue-team/digital-forensics/images/LNK,%20Prefetch%20Files,%20and%20Jump%20Lists-2.png)

- These will include file (`.dlls`) and directories references.

Demonstrating **rogue binaries** by copying `calc.exe` into user's desktop as `conhost.exe`:

![LNK, Prefetch Files, and Jump Lists-3.png](/img/user/cards/blue-team/digital-forensics/images/LNK,%20Prefetch%20Files,%20and%20Jump%20Lists-3.png)

Back to the prefetch folder:

![LNK, Prefetch Files, and Jump Lists-4.png](/img/user/cards/blue-team/digital-forensics/images/LNK,%20Prefetch%20Files,%20and%20Jump%20Lists-4.png)

Then run against both `.exe`:

```C
PECmd.exe -f C:\Windows\Prefetch\CONHOST.EXE // hit tab to auto
```

Then look for the **files referenced section (file path)** then compare both.

**Jump list** can be found on:

**note:** viewing on the GUI will not show the two folders.

```C
%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Recent
```

Then open up a command prompt then type `dir /ad`:

![LNK, Prefetch Files, and Jump Lists-5.png](/img/user/cards/blue-team/digital-forensics/images/LNK,%20Prefetch%20Files,%20and%20Jump%20Lists-5.png)
- **AutomaticDestinations** shows recently accessed files. (this is what we want.)
- **CustomDestinations** shows manually pinned apps by apps or users.

These are unique IDs for recently accessed files and we can use this [list]([https://github.com/EricZimmerman/JumpList/blob/master/JumpList/Resources/AppIDs.txt](https://github.com/EricZimmerman/JumpList/blob/master/JumpList/Resources/AppIDs.txt)) to compare the IDs and determine what application is it.
![LNK, Prefetch Files, and Jump Lists-6.png](/img/user/cards/blue-team/digital-forensics/images/LNK,%20Prefetch%20Files,%20and%20Jump%20Lists-6.png)
Or a much better way is to install `JumpList Explorer`:

![LNK, Prefetch Files, and Jump Lists-7.png](/img/user/cards/blue-team/digital-forensics/images/LNK,%20Prefetch%20Files,%20and%20Jump%20Lists-7.png)
- It shows what are the applications or files recently accessed under a specific app, in this case it shows all recently accessed files **including** opened `.txt` files in `notepad.exe` and such.
- It can also shows deleted files.




