---
{"dg-publish":true,"permalink":"/cards/windows/windows-hardening/"}
---

[[atlas/windows\|windows]]
### Introduction 
---
Hardening endpoints is as important as establishing security policies and implementing security controls in one organization. Although hardening one endpoint is okay but a thousand is not thankfully we can manage this by using software like Active Directory.
## Prerequisites
---
- Windows Endpoint with Administrative Privileges
- Active Directory Installed
## General Concepts
---
- `services.msc` is responsible for creating and managing important functions such as network connectivity, storage, memory, sound, and more that runs automatically in the background.
- `regedit` or Windows Registry it is a container database for important configuration settings, essential keys, and shared preferences for Windos and third-party applications.
- `eventvwr` It is an application that shows log details of any events that occured in your computer including driver updates, hardware failures, invalid authentication attempts and application crash logs.
	- Bad actors make use of the logs for 'getting to know the victim better' and as well as leaving traces of their action by deleting the logs.
- Telemetryis a data collection system used by Microsoft to enhance the user experience by sending crash logs, application specifics to Microsoft without you knowing. It uses the Universal Telemtry Client (UTC) service and runs `diagtrack.dll` to send to Microsoft.
	- `Services` -> "Connected User Experiences and Telemetry"
	- Services from Task Manager shows less information than the dedicated `Services` panel.
	- Location `%ProgramData%\Microsoft\Diagnosis`
## Step 1 - Identity & Access Management
---
It is the implementation of best practices to ensure that only authenticated and authorized users can access a system.

- **Administrator** should only be used for software installations that are verified with trusted source, accessing the registry editor, service and control panel.
- **Regular** for accessing regular applications such as MS Office, Browser and more.

We should always keep in mind the [[Least Privilege\|Least Privilege]] method.
### Best Practices
---
- **Navigate to "Control Panel -> User Accounts"** and create a administrator account and a standard account.
- **Select a sign-in method**: 'Settings -> Accounts -> Sign-in Options' and choose Password however ideally Windows Hello is recommended for more advance sign in method.
- **User Account Control** is a feature that enforces enhanced access control making sure that all services and applications execute in a non-administrator account.
	- To access UAC, go to `Control Panel -> User Accounts -> User Accounts` and click on `Change User Account Control Setting`.
	- Put the slider on "Always Notify" this tracks any app that install or make any changes to computer and the changes you made in Windows settings is notified.
- **Local Policy and Group Policies** ideally configured on Active Directory however can still be used without AD: [[Configuring Password Policy in Active Directory\|Configuring Password Policy in Active Directory]]
## Step 2: Network Connections
---
- **Windows Defender Firewall** is a built-in application that decides what is allowed to come in and come out from your network. 
	- Run `WF.msc` and have the three profiles configured with "Blocked Incoming connections". It is important to note unless configured Firewall block what is not normal.
- **Disable unused networking devices** - routers, ethernet cards, WiFi adapters or any device that enables data sharing must be disabled if not used.
	- To disable the unused Networking Devices, go to the `Control panel > System and Security Setting > System > Device Manager`
- **Disable SMB protocol** - it is a file sharing protocol use for file sharing in a network

```bash
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

- **Protecting Local DNS** attacker could modify with for example: `192.168.1.100 www.example.com` this IP address is controlled by the attacker - the impact everytime the user go to the domain it is redirected to the malicious actor.
	- Open `C:\Windows\System32\Drivers\etc\hosts` and select **Properties** > **Security** tab and **edit** permission ensuring administrator and SYSTEM have full control.
	- In Active Directory or group policy editor -> Navigate to **Computer Configuration > Windows Settings > Security Settings > File System**. Add File and select the **host** file and configure permission to restrict access.
- **Disabling Remote Access to Machine** "Settings > Remote Desktop" then at sidebar "For Developers > Change settings to allow remote connections to this computer" click show settings and tick "Don't allow remote connections to this computer". 

![Windows Hardening.png](/img/user/cards/windows/images/Windows%20Hardening.png)

## Conclusion 
---



