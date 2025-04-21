---
{"dg-publish":true,"permalink":"/cards/blue-team/siem/challenges/splunk-bots-v1/"}
---

[[map-of-contents/blue-team\|blue-team]]

**Indexes:**
```C
| eventcount summarize=false index=*
```

Since we are dealing with unknown indexes and sourcetypes, it's best to learn them first:
![Splunk Bots v1-9.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-9.png)
Revealing all source types:

```C
| metadata type=sourcetypes
| fields sourcetypes
```

**Sourcetypes:**
![Splunk Bots v1-10.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-10.png)
- fgt = fortigate and utm is for incident response, web filtering and more. (Important)

> What is the likely IPv4 address of someone from the Po1s0n1vy group scanning imreallynotbatman.com for web application vulnerabilities?

Since we are dealing with a web server and the attacker is scanning, we should be expecting inbound packets, so analyze the important sourcetypes: `stream:http`, `suricata`, and `fgt*`.

```C
botsv1 sourcetype=fgt* imreallynotbatman.com
```

**Output:** the filter shows that **40.80.148.42** is the top 'talker' to our web server, now it does not mean it is malicious. (spoiler: it is.)
![Splunk Bots v1-11.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-11.png)
Adding to our suspicions by analyzing who is the top IP address being blocked by our firewall (fortigate):

```C
index=botsv1 sourcetype=fgt* imreallynotbatman.com action="blocked"
```

**Output:** 
![Splunk Bots v1-12.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-12.png)
Viewing one of the events:
![Splunk Bots v1-13.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-13.png)
- **Web Server IP**: 192.168.250.70
- **Note:** since suricata and fortigate can block traffic there is always a description why they blocked it.

```bash
index=botsv1 sourcetype=fgt* imreallynotbatman.com action="blocked"
| stats count by srcip, attack
| sort -count
| head 10
```

**Output:** 
![Splunk Bots v1-14.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-14.png)
> What is the name of the file that defaced the imreallynotbatman.com website? Please submit only the name of the file with extension?

My thoughts: is the attacker needs to somehow have access to their content management system in order to defaced their website so it is possible that the attacker submitted a file via POST request.

- However we still lacks information because this is part of the **action on objective** phase we still don't know what the attacker exploited.

> What IPv4 address is likely attempting a brute force password attack against imreallynotbatman.com?

Brute-force in web attacks typically uses POST request on uri such as '/login' pages however in cms such as joomla '/index.php' serves as login page, there are 15k events.

![Splunk Bots v1-15.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-15.png)
Then viewing the src, the same culprit appears and is responsible for 97% of the request:

![Splunk Bots v1-16.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-16.png)
Setting the dest_ip of the web server to see the inbound traffic and http method to POST:

```bash
index="botsv1" sourcetype="stream:http" dest_ip="192.168.250.70" http_method=POST
```

Output:

![Splunk Bots v1-17.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-17.png)
In the sidebar, here are the interesting fields:

- src_ip
- form_data
- http_user_agent
- dest_ip
- uri

```C
index="botsv1" sourcetype="stream:http" dest_ip="192.168.250.70" http_method=POST uri="/joomla/administrator/index.php" | table _time, form_data, uri, src_ip, http_user_agent, dest_ip
```

Output (**23.22.63.114 is responsible)**:

![Splunk Bots v1-18.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-18.png)
Then the first brute-forced password is: (12345678)

![Splunk Bots v1-19.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-19.png)
Add to the previous filter:

```bash
| rex field=form_data "passwd=(?<password>\S{1,6})" | sort password
```

 **Breakdown:**
- `"passwd="` → Matches `"passwd="` literally.
- `(?<password>\S+)` → Captures **everything after `passwd=` until the next space**.
- `\S+` → Matches **one or six non-space characters** (the password).

**Reveals james favorite coldplay song**: yellow.

> What was the correct password for admin access to the content management system running "imreallynotbatman.com"?

We are looking for a `303` redirect (see other) as `200` can indicate that the password only is incorrect in most web application:

**Things to consider:**
- One other thing is brute-force password mistake shows the same content length. adding `http_content_length` to the table.
- Brute-forcing tools will have the same user-agent so the attacker will most likely use a normal browser to login to the page.


```bash
index="botsv1" sourcetype="stream:http" dest_ip="192.168.250.70" http_method=POST uri="/joomla/administrator/index.php"
```

Output:
![Splunk Bots v1-20.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-20.png)
Then adding the same filter:

```C
| table _time, form_data, uri, src_ip, http_user_agent, dest_ip
```

Shows this (admin:batman):

![Splunk Bots v1-21.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-21.png)
or a regex of:

```C
form_data="*pass*
```

> Count all the unique passwords

The regex assumes that the password list used does not contain any special characters:

```bash
index="botsv1" sourcetype="stream:http" dest_ip="192.168.250.70" http_method=POST uri="/joomla/administrator/index.php"
| table _time, form_data, uri, src_ip, http_user_agent, dest_ip
| rex field=form_data "passwd=(?<password>\w+)" | stats count by password
```

Output:

![Splunk Bots v1-22.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-22.png)
> What is the name of the executable uploaded by Po1s0n1vy?

Since it is an executable: we can simply add the keyword `*.exe`:

```C
index="botsv1" sourcetype="*" "imreallynotbatman.com" "*.exe"
```

Then scrolling down and we can see a very strange event with:

![Splunk Bots v1-23.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-23.png)
We can go further down and be specific by adding `http_method == 'POST'` then viewing the filename sidebar: 

![Splunk Bots v1-24.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-24.png)
> What is the MD5 hash of the executable uploaded?

It will be difficult to calulate a hash based on the Splunk event you used in our previous answer. Instead Search for the file name in a different data source to find evidence of execution, including file hash values.

Remember that we have **sysmon** and **fgt_utm** as part of our sources:

```C
XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

Then at the search bar:

```C
index="botsv1" sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" "3791.exe"
```

Since every executable are counted as process we can add `EventID=1` then adding the image path found on the sidebar:

```C
index="botsv1" sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" "3791.exe" EventID=1 Image="C:\\inetpub\\wwwroot\\joomla\\3791.exe"
```

Then viewing the md5 hash sidebar again:

![Splunk Bots v1-25.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-25.png)
**Viewing with fgt_utm as source**:

![Splunk Bots v1-26.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-26.png)
- It also includes the file hash when toggled.

>  What is the name of the file that defaced the imreallynotbatman.com website? Please submit only the name of the file with extension?

- GET request method can also be used to retrieve the file from other host.

Then using the `event_type=fileinfo` from sidebar:

```bash
index="botsv1" sourcetype="*" dest_ip="192.168.250.70" AND src_ip="40.80.148.42" OR src_ip="23.22.63.114" "http.http_method"=GET event_type=fileinfo
```

Output:
![Splunk Bots v1-27.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-27.png)
> This attack used dynamic DNS to resolve to the malicious IP. What fully qualified domain name (FQDN) is associated with this attack?

The above image shows the answer in the **hostname** field.

>What IPv4 address has Po1s0n1vy tied to domains that are pre-staged to attack Wayne Enterprises?

https://otx.alienvault.com/indicator/ip/23.22.63.114 - threat intelligence tools:

**Typosquatting** because why would posion ivy have similar name to wayne enterprise?

![Splunk Bots v1-28.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-28.png)
> How many seconds elapsed between the time the brute force password scan identified the correct password and the compromised login?

Based on the previous result here is the time where the actual attacker log in:
2016-08-10 21:48:05.858

```C
index="botsv1" sourcetype="stream:http" dest_ip="192.168.250.70" http_method=POST  "passwd=batman"
| table _time, form_data, uri, src_ip, http_user_agent, dest_ip
```

Output:
![Splunk Bots v1-30.png](/img/user/cards/blue-team/siem/images/Splunk%20Bots%20v1-30.png)
2016-08-10 21:46:33.689

by appending: `| delta _time`

**92.17**

> What was the average password length used in the password brute forcing attempt?

```C
index="botsv1" sourcetype="stream:http" dest_ip="192.168.250.70" http_method=POST uri="/joomla/administrator/index.php"
| table _time, form_data, uri, src_ip, http_user_agent, dest_ip
| rex field=form_data "passwd=(?<password>\w+)" | eval password_length=len(password) 
| stats min(password_length) AS min_length, avg(password_length) AS avg_length, max(password_length) AS max_length
```

Breakdown:
- **`eval password_length=len(password)`** → Creates a new field `password_length` with the character count of `password`.
- **`stats avg(password_length) AS avg_length`** → Computes the **average length** of all passwords and renames it to `avg_length`.

