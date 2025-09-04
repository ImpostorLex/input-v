---
{"dg-publish":true,"permalink":"/cards/red-team/signature-evasion/","tags":["red-team/host-evasion"]}
---

~ [[atlas/red-team\|red-team]] | [[cards/red-team/Obfuscation Principles\|Obfuscation Principles]]
### Introduction 
---
This covers how to identify and defeat malware signatures that persist even after obfuscation. You’ll learn how AV/EDR detections work, how to spot static and heuristic signatures, and apply advanced coding techniques to evade them using a tool-agnostic approach.
#### Key Topics
---

-  [[#Signature Identification|Binary signature isolation through iterative file splitting]]  

- [[#Automating Signature Identification|Tools like ThreatCheck and Find-AVSignature for speeding up signature detection]]

-  [[#AMSITrigger|Identifying AMSI-triggered detections in PowerShell scripts]]  

- [[#Static Code-Based Signatures|Obfuscation strategies to break code-based detections]]  

- [[#Splitting/Merging Functionality|Code restructuring via method/class merging or splitting to evade static scans]] 

- [[#Removing and Obscuring Identifiable Information|Masking strings and symbols tied to known signatures]]  

- [[#Static Property-Based Signatures|Techniques to alter hash, entropy, and metadata to evade detection]]  

- [[#File hashes|How to mutate file hashes using bit-flipping while preserving functionality]]  

- [[#Entropy|Lowering entropy to reduce suspicion from obfuscated scripts]]  

- [[#Behavioral Signatures|Evading runtime detection through API call manipulation]]  

- [[#Performing Dynamic Loading|Hiding imports by resolving APIs dynamically at runtime]]  

- [[#Limitations and Counter Measures|AV evasion strategies like unhooking and position-independent code]]  

- [[#Putting it All Together|A phased strategy to apply and layer evasion techniques effectively]]  

## Signature Identification
---
Signatures are used by anti-virus engines to track and identify possible suspicious and/or malicious programs. Essentially we want to find out what part of the binary (compiled malware or payload) is triggering the antivirus (AV) detection known as **Binary splitting**.

The steps can be broken down:

1. Determine file size:

```C
ls -l malware.bin
```

2. **Split the file in half** (using `head`, `dd`, or `split`)

**Method 1:**
```C
dd if=malware.bin of=malware_part1.bin bs=1 count=50000
dd if=malware.bin of=malware_part2.bin bs=1 skip=50000
```

- `bs=1` sets the read block size to 1 byte
- `count=50000` copies the first 50,000 bytes
- `skip=50000` skips the first 50,000 bytes and copies the rest

**Method 2**

```C
head -c 50000 malware.bin > malware_part1.bin
tail -c +50001 malware.bin > malware_part2.bin
```

- `head -c 50000`: get the first 50,000 bytes
- `tail -c +50001`: get from byte 50,001 to the end

3. Upload each half to a test machine with antivirus installed
 
4. If AV **detects** it → the signature is in that half  
	- if AV **doesn't detect** it → the signature is in the other half
    
5. Take the "bad half" and **split it again** — repeat the test
    
6. Keep doing this until the chunk is very small (kilobytes or less)
    
7. Open the small chunk in a **hex editor** to see the suspicious bytes or strings

**Example:**

The malicious file size is 73802:

```C
dd if=shell.exe of=shell_part1.exe bs=1 count=36901
dd if=shell.exe of=shell_part2.exe bs=1 skip=36901
```

**Note:** Screenshot are different e.g `.bin` and `.exe` - I was testing something should not confuse this.

But before doing so let's upload shell.exe (the original) to a protected folder and see what happens:

![Signature Evasion.png](/img/user/cards/red-team/images/Signature%20Evasion.png)

Uploading the first part triggers an alert -- then after a few second the file dissapears:

![Signature Evasion-1.png](/img/user/cards/red-team/images/Signature%20Evasion-1.png)

Next we upload the second part and no alerts shows:

![Signature Evasion-2.png](/img/user/cards/red-team/images/Signature%20Evasion-2.png)
## Automating Signature Identification
---
This tool: [Find-AVSignature](https://github.com/PowerShellMafia/PowerSploit/blob/master/AntivirusBypass/Find-AVSignature.ps1) will split a provided range of bytes through a given interval.

However it is not reliable.

### ThreatCheck
---
ThreatCheck leverages several anti-virus engines against split compiled binaries and reports where it believes bad bytes are present.

It requires one thing: the file and optionally an engine; primarily we want to use AMSITrigger when dealing with AMSI (**A**nti-**M**alware **S**can **I**nterface).

```C
ThreatCheck.exe -f Downloads\Grunt.bin -e AMSI
```

![Signature Evasion-4.png](/img/user/cards/red-team/images/Signature%20Evasion-4.png)

Then it will reveal the bad bytes in your code:

![Signature Evasion-5.png](/img/user/cards/red-team/images/Signature%20Evasion-5.png)

The bad bytes is 50461.

Just keep breaking and breaking until no signatures are identified, do note there may be instances of false positives, in which the tool will report no bad bytes, this will require analyst to observe and solve; more [here].
### AMSITrigger
---
Read first: [[cards/red-team/AMSI (Anti-Malware Scan Interface)\|AMSI (Anti-Malware Scan Interface)]] 

**AMSITrigger** is a tool that:

- Emulates how AMSI would scan a **PowerShell (.ps1)** script.
    
- Helps identify **exact lines or chunks** of PowerShell that trigger detection.

```C
.\amsitrigger.exe -i bypass.ps1 -f 3
```

## Static Code-Based Signatures
---
Once the bad signature is identified, it may be broken using obfuscation techniques covered here [[cards/red-team/Obfuscation Principles\|Obfuscation Principles]], or it requires specific investigation and remedy.

As always the layered obfuscation taxonomy covers the most reliable solution.

**Obfuscating methods:**

![Signature Evasion-6.png|450](/img/user/cards/red-team/images/Signature%20Evasion-6.png)

**Obfuscating Classes**

![Signature Evasion-7.png|450](/img/user/cards/red-team/images/Signature%20Evasion-7.png)

Both tables describes different obfuscation techniques used in programming to hide the true behavior of a program. Even though the techniques have different names, they all fall into **two main ideas**:
### Splitting/Merging Functionality
---
The goal is  to break apart or combine code so it's hard to follow **while maintaining previous functionality**, this is very similar to [[cards/red-team/Obfuscation Principles#Object Concatenation\|object concatenation from Obfuscation Principles]].

These techniques take **functions or classes** and either:

- **Split them** into multiple smaller parts spread across the program, OR
- **Merge multiple pieces** into one confusing chunk.

**Original String** static, recognizable string pattern in a variable.

```csharp
string MessageFormat = @"{{""GUID"":""{0}"",""Type"":{1},""Meta"":""{2},""IV"":""{3}"",""EncryptedMessage"":""{4}"",""HMAC"":""{5}""}}";
```

- Antivirus or defenders could search for this exact format
- It’s easy to detect and understand what this string is used for

**Obfuscated Method** makes the string much harder to detect using signature-based scanning.

Below is the new class used to replace and concatenate the string.

```csharp
public static string GetMessageFormat // Format the public method
{
    get // Return the property value
    {
        var sb = new StringBuilder(@"{{""GUID"":""{0}"","); // Start the built-in concatenation method
        sb.Append(@"""Type"":{1},"); // Append substrings onto the string
        sb.Append(@"""Meta"":""{2}"",");
        sb.Append(@"""IV"":""{3}"",");
        sb.Append(@"""EncryptedMessage"":""{4}"",");
        sb.Append(@"""HMAC"":""{5}""}}");
        return sb.ToString(); // Return the concatenated string to the class
    }
}

string MessageFormat = GetMessageFormat
```

- **Built piece-by-piece** using `StringBuilder.Append()`

- Spread across multiple lines (scattered into `Append` calls)

- Returned by a method instead of being directly assigned  (move inside a class)

This makes the string **much harder to detect** using signature-based scanning because:

- The final pattern doesn’t exist in one place
    
- It’s built at **runtime**, not stored statically

### Removing and Obscuring Identifiable Information
---
This is similar to [[cards/red-team/Obfuscation Principles#Object Names\|obscuring variable names here]]. We are taking it one step further by specifically applying it **to identified signatures in any objects** including methods and classes.

Hiding anything that makes your code recognizable, such as:

- Known **string literals** (like `"wdigest.dll"`)
    
- **Method names**, **class names**, or other **unique identifiers**
    
- **Access modifiers** (like `public`, `private`) define how visible or accessible classes tools such as static analyzer relies on access modifiers to map out class heirarchies, map out dependencies, and undestand program flow.
	- Removing modifiers flattend the code - there is no hierarchy.
#### Example Mimikatz
---
Defender or antivirus tools might scan for suspicious things, such as:

```C
string lib = "wdigest.dll";
```

A dead giveaway, this is much better:

```C
string part1 = "wdi";
string part2 = "gest";
string part3 = ".dll";
string lib = part1 + part2 + part3;
```

Challenge [[cards/red-team/Signature Evasion Code Blocks#Static Code-Based Signature Code Block\|get code here]], identify bad signature using AMSITrigger until no signature is detected.

## Static Property-Based Signatures
---
Various detection engines or analysts may consider different indicators rather than strings or static signatures to contribute to their hypothesis. Signatures can be attached to several file properties, including file hash, entropy, author, name, or other identifiable information to be used individually or in conjunction. These properties are often used in rule sets such as **YARA** or **Sigma**.

### File hashes
---
A file hash is used to tag/identify a unique file, commonly used to verify file's authenticity or it's known purpose.

If we have access to the source application changing the file hash is trivial but what if we dont?

Enter _bit-flipping_, is the act of **manually flipping individual bits (0 ↔ 1)** in a file to alter it _very slightly_ — just enough to:

- Produce a **new hash** (bypass static detection)
- Not break the program
#### How General Bit-flipping Works
---
1. Load a binary file.
2. Iterate over the file byte by byte.
3. Flip a single bit (or modify in a predictable way).
4. Save each mutated version to disk in other words every single bit you flip, you save that new version of the file as a seperate `.exe`.
5. Repeat for many bytes to generate many _variants_, one bit flipped = one variant.
6. Each variant has a **different hash**, but may still function the same.

```python
import sys

# Supply the executable as argument
orig = list(open(sys.argv[1], "rb").read())

i = 0
while i < len(orig):
	current = list(orig) # Copy the original list (put each bit as list)
	# XOR the i-th byte with 0xDE (bit-level mutation)
	current[i] = chr(ord(current[i]) ^ 0xde)
	# Save the file as 0.exe, 1.exe, and so on.
	path = "%d.exe" % i
	
	output = "".join(str(e) for e in current)
	open(path, "wb").write(output)
	i += 1
	
print("done")
```


**Next steps:**
Checking for functional variants, including both:

- Some may crash
- Retain their digital signature (some bits are part of the signature block)

```bash
FOR /L %%A IN (1,1,10000) DO (
	signtool verify /v /a flipped\\%%A.exe
)
```

- Loops from 1 to 10,000 (or however many variants you generated)
- Checks each flipped file like `flipped\1.exe`, `flipped\2.exe`, etc.
- Uses `signtool` to:
    
    - Verify if the **digital signature** is still valid
    - Confirm that the **file is trusted and unchanged in critical parts**

The goal: **Find a bit-flipped version that still passes verification** — meaning it will still be trusted by Windows or AV systems.

This is a great technique but can take a long time and will only have limited period until hash is discovered.
### Entropy
---
Is the randomness of the data in a file used to determine whether a file contains hidden data or suspicious scripts.

- **Low entropy** means the data is **predictable or repetitive** (e.g., plain text, uncompressed data).
	- Normal script will have readable commands `Get-Process`, `Invoke-Request`.
- **High entropy** means the data is **random-looking** (e.g., encrypted, compressed, packed malware, embedded content, or obfuscated data).

Entropy can be problematic for obfuscated scripts, specifically when obscuring identifiable information such as variables or functions.

To lower entropy, we can replace random identifiers with randomly selected English words. For example, we may change a variable from `q234uf` to `nature`.

Below is the Shannon entropy scale for a standard English paragraph:

![Signature Evasion-8.png](/img/user/cards/red-team/images/Signature%20Evasion-8.png)

Below is the Shannon entropy scale for a small script with random identifiers:

![Signature Evasion-9.png](/img/user/cards/red-team/images/Signature%20Evasion-9.png)

- Depending on the EDR employed, a “suspicious” entropy value is ~ greater than 6.8.
- Differences between a random value and English text will become amplified with a larger file and more occurrences.
- Note we can use CyberChef for this one
## Behavioral Signatures
---
Modern engines may still observer the behavior or functionality of the binary even after breaking static signature, this presents numerous problems for attackers that cannot be solved with simple obfuscation.

In [[cards/red-team/How Antivirus works and gets bypassed\|introduction to antivirus]], modern engines will employ two common methods to detect behavior: observing imports and [[cards/windows/hook\|hooking]] known malicious calls.

It especially watches **which Windows API functions the program is trying to use**. Two common ways antivirus detects malicious behavior.

1. **Observing API Imports**

	- If a binary imports sensitive APIs (like `OpenProcessToken`, `VirtualAlloc`, etc.), that raises suspicion.

2. **Hooking Function Calls**

	-  The antivirus may "hook" a function (like `CreateRemoteThread`) and see who’s calling it during runtime.

	- This is more advanced and harder to bypass.

Before that let's understand how API calls in C-based program works:

[[cards/red-team/API Calls in C-based Programs\|API Calls in C-based Programs]]
### Blue and Red Team Perspective on this
---
**Antivirus Perspective:**

- If the binary imports `VirtualAllocEx`, `WriteProcessMemory`, and `CreateRemoteThread`, it’s probably suspicious — even if it’s not actively malicious.

- These are all imported via the IAT, which is **easy for AV to scan**.

**Obfuscation Perspective:**

- A malware author may try to **avoid the IAT entirely** by dynamically resolving functions.
    
    - E.g., using `LoadLibrary()` + `GetProcAddress()` at runtime.
        
- Or encrypt the import table and decrypt it only at runtime.
    
- These are techniques to **evade AV that only looks at static imports**.
### Performing Dynamic Loading
---
The problem if the red teamer imports dangerous APIs, they get flagged by AV during scanning, the solution is don't import the function at compile time but resolve it dynamically at runtime using `LoadLibrary` and `GetProcAddress`.

Dynamic loading means you **manually** find the memory address of a function at runtime, this done by:

- Manually loading the DLL (e.g., `kernel32.dll`)
- Manually finding the function’s address inside it
- Calling it using a function pointer you define

**Breaking it down:**

**1. Define the Function Pointer Structure** - we can use official Microsoft Documentation [here](https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-getcomputernamea) to determine it's structure.

Example for `GetComputerNameA`:

```C
typedef BOOL (WINAPI* myNotGetComputerNameA)(
    LPSTR lpBuffer,
    LPDWORD nSize
);
```

This creates a new type `myNotGetComputerNameA` (note the name can be anything), which is a pointer to the same function signature as `GetComputerNameA`.

**2. Load the DLL Module That Contains the API**

Use `LoadLibraryA()` to load the DLL where the API lives.

```C
HMODULE hkernel32 = LoadLibraryA("kernel32.dll");
```

This loads the DLL into your process and gives you a handle to it. You’ll need this to get the function address.

**3. Get the Function Address with GetProcAddress**

```C
myNotGetComputerNameA notGetComputerNameA =
    (myNotGetComputerNameA) GetProcAddress(hkernel32, "GetComputerNameA");
```

You are casting the result of `GetProcAddress` to your function pointer type (`myNotGetComputerNameA`).

1. `myNotGetComputerNameA notGetComputerNameA` -- this line is the equivalent of storing the result of a class into a variable which in this case is `notGetComputerNameA`. This essentially creating a variable that holds a pointer to a function with this **exact**
2. `(myNotGetComputerNameA)` -- tells the compiler "treat this generic address as if it's a function that takes `char*` and `DWORD*`, and returns a BOOL." 

	- Without casting this tells the compiler its a data pointer (points to data) not pointing to a function, this could lead to:
		- Incorrect assumptions
		- Compiler warnings or outright errors.
		- Runtime crashes.

	- Without casting, the compiler **has no idea** how to treat the function call: it doesn’t know what parameters the function takes, or even if it returns anything.

**4. Use the Function Normally**

```C
char buffer[MAX_COMPUTERNAME_LENGTH + 1];
DWORD size = sizeof(buffer);
notGetComputerNameA(buffer, &size);
```

Challenge: [[cards/red-team/Signature Evasion Code Blocks#Behavioral Signatures\|obfuscating API]]
#### Limitations and Counter Measures
---
Antivirus might still flagged this:

- The presence of `LoadLibrary` and `GetProcAddress` in the IAT.
    
- The API behavior when it runs (behavioral analysis, not static).

Counter this with:

todo:: requires more knowledge about below topic but for now it is enough since the purpose is mal dev.

- **Position Independent Code (PIC):** Avoid importing even `LoadLibraryA`/`GetProcAddress` by resolving those dynamically too (manually walking the PEB).
    
- **API Unhooking:** Restore clean versions of functions like `NtOpenProcess` from disk or PEB to bypass AV hooks. 

## Putting it All Together
---
No one method is 100% effective or reliable and the order you apply each technique metters.

**Step 1. Break the most obvious or dangerous static signatures first**

These are the things AV engines or scanners catch right away — things like:

- Hardcoded strings (e.g., `wdigest.dll`, `GetProcAddress`)
- Known function names
- Recognizable class structures or method signatures
    
Use techniques like:

- **Method proxy**
    
- [[#How General Bit-flipping Works|Bit flipping]]
    
- **String encoding**
    
- **Renaming identifiers manually**

Why this instead of using automatic obfuscator first, it may:

- Rename everything randomly
- Flatten your class hierarchy
- Minify and split methods
    
...which makes it **much harder for you to find and surgically edit** the parts you need to change.

**Step 2. Run general/automated obfuscation**

- Control flow flattening
    
- Class hierarchy flattening
    
- String splitting/aggregation
    
- Modifier dropping
    
- Instruction reordering

**These help us red teamers:**

- Confuse reverse engineers
- Throw off heuristic AV engines
- Mask overall intent of code
### Before starting obfuscation, here are some questions to ask yourself
---

- What are the _known signatures_ that will trigger detection? (e.g., class names, API calls, DLLs)
    
- Which functions are most suspicious or easy to detect?
    
- What tools will I use for automation, and what parts will they touch?

The ideal step is have it go with: ThreatCheck.exe then check Import Address Table, then finally obfuscate class names and such.

Challenge [[cards/red-team/Signature Evasion Code Blocks#All of them at once\|lets go]]
### Questions and Problems
---
## Conclusion


