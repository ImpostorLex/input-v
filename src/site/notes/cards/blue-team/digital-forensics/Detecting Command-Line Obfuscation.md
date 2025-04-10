---
{"dg-publish":true,"permalink":"/cards/blue-team/digital-forensics/detecting-command-line-obfuscation/"}
---

[[cards/blue-team/digital-forensics/Digital Forensics\|Digital Forensics]]

Source: https://www.wietzebeukema.nl/blog/bypassing-detections-with-command-line-obfuscation

### Introduction
---
_Malware Intrusions_, a successful compromise without the use of malicious binaries, such as `.exe`, `.dll` for Windows or `.elf` files for linux based operating systems, instead threat actors used the following:

1. **System native scripting languages** such as PowerShell or bash.
2. **System-native executables** these are executables that are built-in or comes in an operating system this is also known as _Living-of-the-Land_ binaries.
3. **Legitimate/trusted third-party executables** tools that allows remote management such as TeamViewer and Anydesk.

Why write a very complicated malware, when you can use tools that is trusted or legitimate that comes with the operating system: Introducing _command-line obsfucation_, it's a technique used to mislead or confused the analysis by **'masquerading' the true intention of the command**, it's different from other types of obfuscation such as DOSfuscation or PowerShell obfuscation:

**Shell-dependent obfuscation** - some technique depends on the behavior of the shell to interpret the obfuscation before the execution

In Windows
```C
cmd /c "po^wer^sh^ell -nop -exec bypass"
```

![Bypassing Detections with Command-Line Obfuscation-3.png](/img/user/cards/blue-team/digital-forensics/images/Bypassing%20Detections%20with%20Command-Line%20Obfuscation-3.png)

In LimaCharlie (EDR) although it detected the original code this demonstrate the attack that depends on shell's ability to interpret the obfuscation:

![Bypassing Detections with Command-Line Obfuscation-2.png](/img/user/cards/blue-team/digital-forensics/images/Bypassing%20Detections%20with%20Command-Line%20Obfuscation-2.png)
**Shell-independent obfuscation** - obfuscation happens at the **argument level**, it does not depend on the shell but instead it depends on how the executable interprets it

In Windows (aQBlAHgA = iex or Invoke-Expression):
```C
powershell.exe -ExecutionPolicy Bypass -EncodedCommand aQBlAHgA 
```

![Bypassing Detections with Command-Line Obfuscation-4.png](/img/user/cards/blue-team/digital-forensics/images/Bypassing%20Detections%20with%20Command-Line%20Obfuscation-4.png)

In LimaCharlie:

![Bypassing Detections with Command-Line Obfuscation-5.png](/img/user/cards/blue-team/digital-forensics/images/Bypassing%20Detections%20with%20Command-Line%20Obfuscation-5.png)

## Creating the rule

```
event: NEW_PROCESS
op: matches
path: event/COMMAND_LINE
case sensitive: false
re: .[^\x00-\x7F]
```

1. We are looking for a **new process** - as the attack focuses on built-in executables.
2. `matches` since we are using regular expression.
3. We want to look in the COMMAND_LINE field or key.
4. Regular expression: matches any non-ASCII character such as [this](http://kestrel.nmt.edu/~raymond/software/howtos/greekscape.xhtml) greek unicode letters commonly used in typosquatting or obfuscation. 

```
# Response
- action: report
  name: Potential Command Line Obfuscation
```
### Testing the rule
---
I am going to use the code example in the article with argfuscator[.]net:
![Bypassing Detections with Command-Line Obfuscation-6.png](/img/user/cards/blue-team/digital-forensics/images/Bypassing%20Detections%20with%20Command-Line%20Obfuscation-6.png)

```C
reg export HKLM\SAM example.reg
```

In Windows:

![Bypassing Detections with Command-Line Obfuscation-7.png](/img/user/cards/blue-team/digital-forensics/images/Bypassing%20Detections%20with%20Command-Line%20Obfuscation-7.png)

In LimaCharlie:

![Bypassing Detections with Command-Line Obfuscation-8.png](/img/user/cards/blue-team/digital-forensics/images/Bypassing%20Detections%20with%20Command-Line%20Obfuscation-8.png)

Of course this will not work however it also detected high unicode characters:

```
Κ (U+039A) Greek Capital Letter Kappa
```

=

```
reg export HΚLM\SAM example.reg # modified with the Greek Capital Letter
```

In Windows:

![Bypassing Detections with Command-Line Obfuscation-9.png](/img/user/cards/blue-team/digital-forensics/images/Bypassing%20Detections%20with%20Command-Line%20Obfuscation-9.png)
In LimaCharlie:

![Bypassing Detections with Command-Line Obfuscation-10.png](/img/user/cards/blue-team/digital-forensics/images/Bypassing%20Detections%20with%20Command-Line%20Obfuscation-10.png)
#### Detecting for multiple quotes usage
---
Modifying our previous rule with this one:

```C
event: NEW_PROCESS
op: or
rules:
  - op: matches
    path: event/COMMAND_LINE
    re: .[^\x00-\x7F]
    case sensitive: false
  - op: matches
    path: event/COMMAND_LINE
    case sensitive: false
    re: .(?:.*['\"].*){5,}
```

Changes:
```C
op: matches
    path: event/COMMAND_LINE
    case sensitive: false
    re: .(?:.*['\"].*){5,}
```

- `(?: ... )` → **Non-capturing group** (avoids unnecessary capturing).
    
- `.*['\"].*` → Looks for **at least one quote (`'` or `"`)** in any part of the string.
	- `.*` zero or more occurrences of any character - looking for quotes regardless of what comes first or after them.
	- `['\"]` a set of character to look for in this case it's single and double quote with double quote escaped.
- `{5,}` → Requires **this pattern to happen at least 5 times** or more.

https://regex101.com/ no matches since it only contains four quotes.

![Bypassing Detections with Command-Line Obfuscation-11.png](/img/user/cards/blue-team/digital-forensics/images/Bypassing%20Detections%20with%20Command-Line%20Obfuscation-11.png)

##### True Positive
---
1.
```C
reg export HKLM\SAM "r""e""g""o""u""t".reg
```

2.
```
powershell -Command "& { Write-Output """"""Hello"""""" }"
```

![Bypassing Detections with Command-Line Obfuscation-12.png](/img/user/cards/blue-team/digital-forensics/images/Bypassing%20Detections%20with%20Command-Line%20Obfuscation-12.png)
##### False Positive
---
1.
```C
powershell -Command "Write-Output 'Hello'"
```

2.
```C
reg export HKLM\SAM out.reg
```

### Conclusion
---
However to a Security Information & Event Management (SIEM) or any monitoring tools, these obfuscation techniques can be easily identified by the analyst, in Splunk with sysmon installed:

![Detecting Command-Line Obfuscation.png](/img/user/cards/blue-team/digital-forensics/images/Detecting%20Command-Line%20Obfuscation.png)

Command obfuscated by argfuscator[.]net:
![Detecting Command-Line Obfuscation-1.png](/img/user/cards/blue-team/digital-forensics/images/Detecting%20Command-Line%20Obfuscation-1.png)

However the problem with SIEM - it cannot do the 'responding' part automatically, it requires analyst to actually analyze the obfuscation, and correlate with other data sources, which cost a lot of time.







