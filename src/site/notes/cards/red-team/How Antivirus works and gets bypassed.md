---
{"dg-publish":true,"permalink":"/cards/red-team/how-antivirus-works-and-gets-bypassed/","tags":["red-team/host-evasion"]}
---

~ [[map-of-contents/red-team\|red-team]]
### Introduction 
---
It's a host-based security solution designed to detect and prevent the execution of malicious files on the targeted operating system. It is running in the background by default.

> [!question]+ Antivirus vs Endpoint Detection Response (EDR)
> Antivirus looks for predefined patterns or signatures, while an EDR can do what most Antivirus can do plus can perform security checks on file activities, memory, network connections, Windows registry, processes, and more.
#### Key Topics
---
 - [[#Features|Core components of antivirus systems, including scanning, detection methods, unpackers, and emulators]]
    
- [[#Detection Techniques|Overview of detection strategies such as signature-based, heuristic, dynamic analysis, and YARA rules]]
    
- [[#What the Red Teamer should do|Red team tactics for antivirus evasion, fingerprinting installed AVs, and using external testing platforms]]
## Features
---

**Scanner**

- Scans the whole disk in real-time or on-demand.
- Good Antivirus should have the ability to scan for vulnerabilities, emails, Windows memory and Windows Registry.

**Detection Techniques**

- **Signature-based detection**: Detects known threats by matching files against a database of malware signatures.  
    - _Example: A virus file with a known hash is flagged instantly._

- **Heuristic detection**: Analyzes file structure and behavior to identify suspicious traits, even if the malware is unknown.  
    - _Example: A file tries to modify system files or registry settings._

- **Dynamic detection**: Runs the file in a sandbox or monitors its real-time activity to catch malicious behavior.  
    - _Example: A script opens a network connection and downloads additional payloads during execution._

**Compressors and Archives**

One of the ways of bypassing host-based security solutions is using compressed or archived files, an Antivirus should be able to decompress and scan through all the files before the user opens a file within the archive.

**Portable Executable Parsing and Unpackers**

- **Packing**: Malware often compresses or encrypts its code using packers (e.g., UPX, Armadillo, ASPack) to avoid detection and static analysis.  A **packer** is a tool or software that **compresses or encrypts** a program's code (usually an executable like `.exe`) to **obfuscate** or hide its real content

    - Section: `.UPX0`, `.UPX1` instead of standard `.text`, `.data`
	- Compressed code that doesn't reveal any readable strings or functions.
    
- **Unpacking capability**: Antivirus software must be able to detect and unpack known packers to reveal the original code before runtime.  
    - _Example: An AV unpacks a compressed .exe file to scan its actual contents._
    
- **PE header parsing**: AVs should parse Windows Portable Executable (PE) headers to analyze executable files (.exe, .dll). This helps distinguish legitimate from suspicious files based on structure and metadata.  
    - _Example: A PE file with a missing or unusual import table might indicate tampering or malware._

**Emulators**

It runs the suspected file (exe, DLL, PDF, etc) in a virtualized and controlled environment. It monitors the executable files' behavior during the execution, including the Windows APIs calls, Registry, and other Windows files.

## Detection Techniques
---

**A database of known malicious signatures and patterns.**

**Yara rules**

See how to create [[cards/blue-team/malware-analysis/YARA\|Yet Another Ridiculous Acronym (YARA)]] for more information.

**Custom rules**

**Viewing the strings (static analysis)**

See how to perform [[cards/blue-team/malware-analysis/Basic Static Analysis\|Basic Static Analysis]] and [[cards/blue-team/malware-analysis/Advanced Static Analysis\|Advanced Static Analysis]].

**Dynamic Analysis**

- Analyze the use of Windows API.
- Sandboxing: executing the malware inside an isolated environment with the goal of analyzing how the malicious software acts in the system, once confirmed malicious, a unique signature and rule will be created based on the characteristic of the binary, and finally an update will be pushed in the cloud database.
- See [[cards/blue-team/malware-analysis/Basic Dynamic Analysis\|Basic Dynamic Analysis]] and [[cards/blue-team/malware-analysis/Advanced Dynamic Analysis\|Advanced Dynamic Analysis]].

**Heuristic and Behavioral Detection**

1. **Static Heuristic Analysis** is a process of decompiling (if possible) and extracting the source code of the malicious software. Then, the extracted source code is compared to other well-known virus source codes. These source codes are previously known and predefined in a heuristic database. If a match meets or exceeds a threshold percentage, the code is flagged as malicious.
2. **Dynamic Heuristic Analysis** is based on predefined behavioral rules. Security researchers analyzed suspicious software in isolated and secured environments. Based on their findings, they flagged the software as malicious. Then, behavioral rules are created to match the software's malicious activities within a target machine.

## What the Red Teamer should do
---

- Platforms such as VirusTotal or any Antivirus engine should be used to test the payload that we are going to use to determine if our payload is **well known** or is too 'obvious' based on patterns and signatures.
	- Tools such as [AntiscanMe](https://antiscan.me/) (6 free scans a day) and [Virus Scan Jotti's malware scan](https://virusscan.jotti.org/) is recommended as they don't have **sharing policy** unlike VirusTotal.

- **Fingerprinting:** determining what Antivirus is installed, this is useful in creating the same environment to test bypass techniques.

| **Antivirus Name** | **Service Name**                  | **Process Name**                   |
| ------------------ | --------------------------------- | ---------------------------------- |
| Microsoft Defender | WinDefend                         | MSMpEng.exe                        |
| Trend Micro        | TMBMSRV                           | TMBMSRV.exe                        |
| Avira              | AntivirService, Avira.ServiceHost | avguard.exe, Avira.ServiceHost.exe |
| Bitdefender        | VSSERV                            | bdagent.exe, vsserv.exe            |
| Kaspersky          | AVP<Version #>                    | avp.exe, ksde.exe                  |
| AVG                | AVG Antivirus                     | AVGSvc.exe                         |
| Norton             | Norton Security                   | NortonSecurity.exe                 |
| McAfee             | McAPExe, Mfemms                   | MCAPExe.exe, mfemms.exe            |
| Panda              | PavPrSvr                          | PavPrSvr.exe                       |
| Avast              | Avast Antivirus                   | afwServ.exe, AvastSvc.exe          |

**Note:** we can use the binary listed here and create a program to fingerprint antivirus, here is one example made in C# however uploading to VirusTotal flags it.

```csharp
using System;
using System.Management;

internal class Program
{
    static void Main(string[] args)
    {
        var status = false;
        Console.WriteLine("[+] Antivirus check is running .. ");
        string[] AV_Check = { 
            "MsMpEng.exe", "AdAwareService.exe", "afwServ.exe", "avguard.exe", "AVGSvc.exe", 
            "bdagent.exe", "BullGuardCore.exe", "ekrn.exe", "fshoster32.exe", "GDScan.exe", 
            "avp.exe", "K7CrvSvc.exe", "McAPExe.exe", "NortonSecurity.exe", "PavFnSvr.exe", 
            "SavService.exe", "EnterpriseService.exe", "WRSA.exe", "ZAPrivacyService.exe" 
        };
        var searcher = new ManagementObjectSearcher("select * from win32_process");
        var processList = searcher.Get();
        int i = 0;
        foreach (var process in processList)
        {
            int _index = Array.IndexOf(AV_Check, process["Name"].ToString());
            if (_index > -1)
            {
                Console.WriteLine("--AV Found: {0}", process["Name"].ToString());
                status = true;
            }
            i++;
        }
        if (!status) { Console.WriteLine("--AV software is not found!");  }
    }
}
```

Time to use your knowledge and recreate the script in other language with different patterns to avoid detection.


