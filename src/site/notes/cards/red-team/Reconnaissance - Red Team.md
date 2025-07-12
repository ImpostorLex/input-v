---
{"dg-publish":true,"permalink":"/cards/red-team/reconnaissance-red-team/","tags":["red-team/ad","red-team/initial-access"]}
---

~ [[map-of-contents/red-team\|red-team]]
### Introduction
---
> _"Know your enemy, know his sword"_
> - Miyamoto Musashi

A proven strategy (not a stategy but a standard really) -- to have a successful penetration testing or red teaming, we must know our target well enough.

There are two types of reconnaissance:

1. **Passive** -- does not directly interacting with the target.
2. **Active** -- opposite of passive.
	- **External Recon**: conducting reconnaissance outside of the target's network such as running scan on externally faced assets.
	- **Internal Recon:** conducted from within the target's company's network. The pentester might be physically located inside the company or the pentester already compromised a machine.

#### Key Topics
---

- [[#Built-in Tools|Use whois, traceroute to find out IP addresses and other details.]]
    
- [[#Advanced Searching|Leverage advanced search techniques for OSINT gathering.]]
    
- [[#Specialized Search Engines|Use specialized search engines like Censys for detailed domain and IP data.]]
    
- [[#Recon-ng|Automate OSINT collection and organize results with Recon-ng.]]
    
- [[#Maltego|Map relationships and visualize data with Maltego’s transforms.]]


## Built-in Tools
---

**whois**

It is a request and response protocol tha listens on TCP port 43 for incoming request, the domain registrat is responsible for maintaining the WHOIS records. the `whois` will query the WHOIS server for information.

```C
whois thmredteam.com
```

- Registrar WHOIS server
- Registrar URL
- Record creation date
- Record update date
- Registrant contact info and address (unless withheld for privacy)
- Admin contact info and address (unless withheld for privacy)
- Tech contact info and address (unless withheld for privacy)

**nslookup**

Perform a DNS query returning IPv4(6) addresses

```C
nslookup cafe.thmredteam.com
```

**dig**

Does the same with a flexibility for our query such as querying DNS record using a specific DNS server:

```C
dig cafe.thmredteam.com @1.1.1.1
```

**host**

```C
host cafe.thmredteam.com
```

**traceroute**

Shows the path taken by our packet to get to the target. It's important to know that some routers does not respond with packet sent by `traceroute` indicated by ' * '.

```C
traceroute cafe.thmredteam.com
```

## Advanced Searching
---
Some commands here work for other search engine as well:

|Symbol / Syntax|Function|
|---|---|
|`"search phrase"`|Find results with exact search phrase|
|`OSINT filetype:pdf`|Find files of type `PDF` related to a certain term.|
|`salary site:blog.tryhackme.com`|Limit search results to a specific site.|
|`pentest -site:example.com`|Exclude a specific site from results|
|`walkthrough intitle:TryHackMe`|Find pages with a specific term in the page title.|
|`challenge inurl:tryhackme`|Find pages with a specific term in the page URL.|

Additionally due to the nature of web crawlers they might indexed sensitive information with improper access control or due to misconfiguration: Websites such as [Google Hacking Database](https://www.exploit-db.com/google-hacking-database) (GHDB) collect such search terms and are publicly available

Some examples:

- **Footholds**  
    Consider [GHDB-ID: 6364](https://www.exploit-db.com/ghdb/6364) as it uses the query `intitle:"index of" "nginx.log"` to discover Nginx logs and might reveal server misconfigurations that can be exploited.
- **Files Containing Usernames**  
    For example, [GHDB-ID: 7047](https://www.exploit-db.com/ghdb/7047) uses the search term `intitle:"index of" "contacts.txt"` to discover files that leak juicy information.
- **Sensitive Directories**  
    For example, consider [GHDB-ID: 6768](https://www.exploit-db.com/ghdb/6768), which uses the search term `inurl:/certs/server.key` to find out if a private RSA key is exposed.
- **Web Server Detection**  
    Consider [GHDB-ID: 6876](https://www.exploit-db.com/ghdb/6876), which detects GlassFish Server information using the query `intitle:"GlassFish Server - Server Running"`.
- **Vulnerable Files**  
    For example, we can try to locate PHP files using the query `intitle:"index of" "*.php"`, as provided by [GHDB-ID: 7786](https://www.exploit-db.com/ghdb/7786).
- **Vulnerable Servers**  
    For instance, to discover SolarWinds Orion web consoles, [GHDB-ID: 6728](https://www.exploit-db.com/ghdb/6728) uses the query `intext:"user name" intext:"orion core" -solarwinds.com`.
- **Error Messages**  
    Plenty of useful information can be extracted from error messages. One example is [GHDB-ID: 5963](https://www.exploit-db.com/ghdb/5963), which uses the query `intitle:"index of" errors.log` to find log files related to errors.

Don't forget about social media, job ads, and wayback machine.
## Specialized Search Engines
---
There are some paid tools that allows us to see WHOIS history -- this could be useful to find organization who did not secure there WHOIS privacy by including email address and more.

**ViewDNS.info** -- https://viewdns.info/

Traditionally, each web server would have its own unique IP address with the rise of _shared hosting_, many websites can share the same IP address.

![Pasted image 20250601072445.png](/img/user/cards/red-team/images/Pasted%20image%2020250601072445.png)

**Censys**

It's a tool that helps you find detailed information about IP addresses and domains.


In this example, we can see that the IP address is owned by CloudFare and the IP address is used by other websites as well:

![Pasted image 20250601073007.png](/img/user/cards/red-team/images/Pasted%20image%2020250601073007.png)
## Recon-ng
---
Is a framework that helps automating Open Source Intelligence (OSINT) using various modules from authors

1. **Workspaces**:
    
    - Workspaces allow you to organize your investigations by creating isolated environments for different targets or projects. Each workspace contains its own database and modules.
    - You can create a workspace using the command:

```C
workspaces create <WORKSPACE_NAME>
```

- You can start Recon-ng in a specific workspace by using:

```C
recon-ng -w <WORKSPACE_NAME>
```
        
2. **Database**:
    
    - Recon-ng allows you to insert, manage, and query data within a database. This is crucial for organizing and storing information you gather during your investigation.
    - The database stores various data points such as domain names, IPs, hosts, contacts, etc.
    - To insert a domain into the database:
         
```C
db insert domains <DOMAIN_NAME
```
        
3. **Modules**:
    
    - **Modules** are the building blocks of Recon-ng. They allow you to gather specific types of data or perform specific reconnaissance actions.
        
    - Modules are organized into categories like **discovery**, **import**, **recon**, and **reporting**. They can transform one type of information into another (e.g., domains to hosts, emails to accounts, etc.).
        
4. Marketplace

- **Marketplace** is where you can search for, install, and manage modules that can extend the capabilities of Recon-ng.

**Useful Commands for Working with the Marketplace:**

- **Search for modules**:
```C
marketplace search <KEYWORD>
```

- **Install a module**:

```C
marketplace install <module_name>
```

- simply replace `install` with `remove` to remove a module.

5. **Running Modules**

- To load a module into memory:

```C
modules load <MODULE_NAME>
```

- To see what options are available for the loaded module:

```C
options list
```

- To Set an option for the module

```C
options set <OPTION_NAME> <VALUE>
```

Then hit run

6. Keys

Some modules require an **API key** to interact with external services like Google, Whois databases, etc.

You can manage API keys within Recon-ng using:

List keys:

```C
keys list
```

Add a key:

```C
keys add <KEY_NAME> <KEY_VALUE>
```

Simply remove `add` and `key value` with `remove` to remove a key

## Maltego
---
Maltego is a tool that combines mind-mapping and OSINT. You start with an entity like a domain name or company and apply transforms to gather related information.

### Entities and Transforms
---
Each block on a Maltego graph is an **entity**. A **transform** queries an API to retrieve new related entities. The results help build your investigation.

![Pasted image 20250601075342.png](/img/user/cards/red-team/images/Pasted%20image%2020250601075342.png)

### Passive vs Active Reconnaissance
---

Some transforms might actively query the target system, so it's important to choose **passive reconnaissance** transforms if needed.

### Example Workflow
---
Starting with a DNS Name like `cafe.thmredteam.com`, applying a "Resolve to IP" transform gives IP addresses.

- **Standard Transforms → Resolve to IP → To IP Address (DNS)**

Once IPs are obtained, you can apply another transform:

- **DNS from IP → To DNS Name from passive DNS (Robtex)**

This will populate the graph with new DNS names.

![Pasted image 20250601075410.png|450](/img/user/cards/red-team/images/Pasted%20image%2020250601075410.png)

### Maltego Workflow Summary
---
Maltego organizes your data efficiently, letting you gather information with just a few clicks, compared to querying individual websites.

### WHOIS and nslookup
---
We experimented with **whois** and **nslookup** previously. Maltego transforms organize WHOIS data like names, emails, and IPs, visually presenting the results.

![Pasted image 20250601075455.png](/img/user/cards/red-team/images/Pasted%20image%2020250601075455.png)

### Adding New Transforms
---
You can enhance Maltego's functionality by adding new transforms. Some are free in the **Community Edition**, while others require a paid subscription.

![Pasted image 20250601075459.png](/img/user/cards/red-team/images/Pasted%20image%2020250601075459.png)
