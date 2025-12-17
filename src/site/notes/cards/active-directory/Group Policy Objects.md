---
{"dg-publish":true,"permalink":"/cards/active-directory/group-policy-objects/","tags":["windows/ad"]}
---

[[cards/active-directory/windows active directory\|windows active directory]]
### Introduction 
---
A virtual collection of policy settings that is stored in the SYSVOL directory to be replicated to domain-joined machines. Each GPOs is uniquely named called GUID.

Each Windows computer has a local policy configuration such as:

- Application configuration for services such as the Firewall, Anti-Virus, and Applocker.  
- Local Group membership such as the Administrator or Remote Desktop Users groups.
- Startup configuration such as scripts that should be executed.
- And more.

##### Group Policy Management
---
It is used to manage group policy of thousands of workstations in an organization in a Active Directory environment.

Domain-joined computers would then pull all policies from SYSVOL periodically and apply the relevant ones. By default, policies are replicated every 15 minutes through the gpupdate application.
## Sample Usage
Enforcing password policy:

- Using the Run window, open **Group Policy Management** from your server by typing `gpmc.msc`.
- Right-click your domain and select **"Create a GPO in this domain, and Link it here"**. Name the new GPO **"Password Policy"**.
- Edit the GPO by navigating to **Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Account Policies -> Password Policy**.
- Configure the following settings:
    - Minimum password length: 12 characters
    - Enforce password history: 10 passwords
    - Maximum password age: 90 days
    - Password must meet complexity requirements: Enabled
- Click **OK**, then link this GPO to the domain or specific OUs you want to targe
## Sample Investigation

As said before GPO can be a great way to spread malicious scripts to multiple devices, to list all GPOs installed on the domain controller:

```
Get-GPO -All
```

Output:

![Group Policy Objects.png](/img/user/cards/active-directory/images/Group%20Policy%20Objects.png)

Then once a out of place GPO is found, we can see the overview by exporting it to HTML:

```powershell
Get-GPOReport -Name "Malicious GPO - Glitch_Malware Persistence" -ReportType HTML -Path ".\set.html"
```

Output:

![Group Policy Objects-1.png](/img/user/cards/active-directory/images/Group%20Policy%20Objects-1.png)
And then look for **Computer/User Configuration** to see if it's enabled however most of the time domains have naturally many GPOs, in this snippet we can see only modified GPOs:

```powershell
Get-GPO -All | Where-Object { $_.ModificationTime } | Select-Object DisplayName, ModificationTime
```

**Event Viewer** is a great tool investigating system activity and user auditing:

View users present on the domain and their group membership:

```powershell
Get-ADUser -Filter * -Properties MemberOf | Select-Object Name, SamAccountName, @{Name="Groups";Expression={$_.MemberOf}}
```

Output:

![Group Policy Objects-2.png](/img/user/cards/active-directory/images/Group%20Policy%20Objects-2.png)

Viewing powershell history and logs:

```
APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

Output:

![Group Policy Objects-3.png](/img/user/cards/active-directory/images/Group%20Policy%20Objects-3.png)

For Event Viewer under `Application and Services Logs -> Microsoft -> Windows -> PowerShell -> Operational` or also under `Application and Service Logs -> Windows PowerShell`

## Red Teaming

To setup a group policy that executes on startup or shutdown we can do this by clicking the Show Files and placing it there.
![Pasted image 20241216184820.png](/img/user/cards/active-directory/images/Pasted%20image%2020241216184820.png)



### Questions and Problems
---
## Conclusion


