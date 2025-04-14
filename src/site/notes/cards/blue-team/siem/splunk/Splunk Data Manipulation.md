---
{"dg-publish":true,"permalink":"/cards/blue-team/siem/splunk/splunk-data-manipulation/","tags":["splunk"]}
---

[[cards/blue-team/siem/Splunk Map\|Splunk Map]]
### Introduction 
---
Processing, parsing and manipulation on data is very important in order to generate meaningful insights and enabling effective analysis for security threats, investigating incidents, and monitoring system health.
## How Splunk Process Data

1. Define the format of the data source and identify the relevant fields you want to extract.
2. Identify the sourcetype so Splunk can apply specific parsing rules, if not create your own one in `props.conf`. It resides on `$SPLUNK_HOME/etc/system/local`.
3. Example:

```c
[source::/path/to/your/data] // Path to data source
sourcetype = your_sourcetype // Name for the sourcetype
```

4. Next is define what fields are extracted:

```c
[your_sourcetype] // name of the source type from step 3.
EXTRACT-fieldname1 = regular_expression1 
EXTRACT-fieldname2 = regular_expression2
```

`[your_sourcetype]` is the name previously assigned to the new data source.
5. Save, Restart and Verify the changes.
## Stanzas in Splunk

Configuration files are organized into sections called "stanzas" which defines various settings and parameters for how data is processed, indexed, and managed. Each stanza is identified in a configuration file with square brackerts `[]` and it's parameters:

Here's one example of a configuration file `inputs.conf` which defines how data is ingested into Splunk:

```C
[monitor:///var/log/syslog] 
disabled = false 
index = main 
sourcetype = syslog
```

1. Read and ingest new data as it appears similar to `tail -f`.
2. not disabled, keep monitoring, it is great for managing multiple configurations.
3. which index to send the data.
4. Tell splunk to use the sourcetype `syslog` which is built-in.

Here is a step by step:

1. `inputs.conf` collect data.
2. `props.conf` parse the data. 
3. `transforms.conf` enrich fields, route to a different index, alias and more. (Optional) 

### Creating a Simple App
---
Splunk let's you use pre-packaged software modules or extensions to enhance the functionality of the Splunk platform, to view:

![cards/blue-team/siem/images/Splunk Data Manipulation.png](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation.png)

The new apps created will be placed on `/opt/splunk/etc/apps`, after filling the details we can launch the app but of course no data or event is shown (Steps and images not shown because it's basic).

Here is a sample app with it's default files:

![Splunk Data Manipulation-1.png](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation-1.png)

- `app.conf` Metadata file defining the app's name, version, and more.
- `bin` Holds custom scripts or binaries required by the app.
- `default` Contains XML files defining app dashboards and views.
- `local` Optionally used for overriding default UI configurations.

#### Step 2. Python script for the /bin directory (Logs)
---
- A simple print statement 
- Note full path to the python script.
- `/opt/splunk/etc/apps/DataApp/bin/sample.py`
#### Step 2.1 Creating Inputs.conf and the other stanzas in the default directory
---
Starting with `inputs.conf`:

```c
[script:///opt/splunk/etc/apps/DataApp/bin/sample.py] 
index = main
source = test_log
sourcetype = testing
interval = 5
```

1. This stanza tells Splunk to execute the specified script (`samplelogs.py`)  - great for logs that requires scripting first.
2. `source = test_log` used to identify the origin of data appears in the `source` field.
3. `sourcetype = testing`  Uses the `testing` parser made in `pros.conf` fike.

Executes the scripts and then sends to index **main** every 5 seconds, and after this it's important to restart splunk.

![cards/blue-team/siem/images/Splunk Data Manipulation-2.png](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation-2.png)
### Event Boundaries
---
Splunk breaks raw data into individual events using a specified rule this helps Splunk to identify where an event begins and where it ends.

Here is an example of a `vpnlogs`:

![Splunk Data Manipulation-3.png](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation-3.png)

At the `DataApp/defaults` directory `inputs.conf` and copy the `vpnlogs` script to the bin directory (Since looking at the image it's currently located in the **Downloads/scripts**):

```c
[script:///opt/splunk/etc/apps/DataApp/bin/vpnlogs]
index = main
source = vpn
sourcetype = vpn_logs
interval = 5
```

Restart and view using **source_type**:

```
sudo $SPLUNK_HOME/bin/splunk restart
```

![Splunk Data Manipulation-4.png](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation-4.png)
The problem is it consider multiple events as a single events, to fix this it requires making changes to the `props.conf` and using regex to determine the end of the event:

![Splunk Data Manipulation-5.png|450](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation-5.png)

Creating `props.conf` in the default directory (`$SPLUNK_HOME/etc/system/local` or for better organization (try either if one does not work): `$SPLUNK_HOME/etc/apps/your_app/local/`) with the regex above that tells it must break after the pattern matches:

```c
[vpn_logs]
SHOULD_LINEMERGE = true
MUST_BREAK_AFTER = (DISCONNECT|CONNECT)
```

- **SHOULD_LINEMERGE** determine if lines will be merged into a single event or treated as seperate events. (for cases a single event has multiple lines)

Output after restarting:
![Splunk Data Manipulation-6.png](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation-6.png)
## Parsing multi-line log sources

Sample log generated:

```c
[Authentication]:A login attempt was observed from the user Michael Brown and machine MAC_01
at: Mon Jul 17 08:10:12 2023 which belongs to the Custom department. The login attempt looks suspicious.
```

The `inputs.conf` configuration (Again **authenticaton_logs** is a custom script.):

```c
[script:///opt/splunk/etc/apps/DataApp/bin/authentication_logs]
interval = 5
index = main
sourcetype= auth_logs
host = auth_server
```

The problem is Splunk is breaking the 2-line Event into 2 different events and is unable to determine the boundaries.:

![Splunk Data Manipulation-7.png](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation-7.png)
The solution is the event always starts with `[AUTHENTICATION]`, we can use this as a regex pattern with `BREAK_ONLY_BEFORE` in the `props.conf`:

```c
[auth_logs]
SHOULD_LINEMERGE = true
BREAK_ONLY_BEFORE = \[Authentication\]
```

### Masking Sensitive Data for compliance with standards such as PCI DSS and HIPAA
---

Sample generated events:

```c
User William made a purchase with credit card 3714-4963-5398-4313.
User John Boy made a purchase with credit card 3530-1113-3330-0000.
User Alice Johnson made a purchase with credit card 6011-1234-5678-9012.
```

The `inputs.conf` (**purchase-details** is a script.):

```c
[script:///opt/splunk/etc/apps/DataApp/bin/purchase-details]
interval = 5
index = main
source = purchase_logs
sourcetype= purchase_logs
host = order_server
```

- **host** tells which hostname of the machibne generated the log

In the Splunk interface the credit card are not masked and multiple events are treated as single event:

![Splunk Data Manipulation-8.png|450](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation-8.png)

The regex pattern to identify the **end of the boundary** for `props.conf`:

![Splunk Data Manipulation-9.png|450](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation-9.png)

Adding the event boundary into `props.conf`:

```C
[purchase_logs]
SHOULD_LINEMERGE = true
MUST_BREAK_AFTER = \d{4}\.
```

Save the file, restart splunk, and see the output:

![Splunk Data Manipulation-10.png](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation-10.png)
### SEDCMD for data transformation during indexing
---
It is similar to the Unix `sed` command and here how it works:

1. Open the `props.conf` file in your Splunk configuration directory.
2. Locate or create a stanza for the data source you want to modify.
3. Add the `sedcmd` setting under the stanza.
4. Specify the regular expression pattern and the replacement string using the `s/` syntax similar to the `sed` command.

```c
[source::/path/to/your/data]
SEDCMD-myField = s/oldValue/newValue/g
```

In this example, the `sedcmd` setting is applied to the data from a specific source path. It uses the regular expression pattern `oldValue` and replaces it globally with newValue using the `g` flag in the `myField` field. 
#### Masking the CC information
---
The regex:

![Splunk Data Manipulation-11.png](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation-11.png)

Back in the `props.conf` then restart splunk:

```c
[purchase_logs]
SHOULD_LINEMERGE = true
MUST_BREAK_AFTER = \d{4}\.
SEDCMD-cc = s/-\d{4}-\d{4}-\d{4}/-XXXX-XXXX-XXXX/g
```

## Extracting Custom Fields

The current log  great if we don't want to perform analysis on **username**, **server**, and **action**  fields, otherwise it's not:

![Splunk Data Manipulation-12.png|450](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation-12.png)
### Capturing Username
---
Sample Event:

```c
User: John Doe, Server: Server C, Action: CONNECT
User: John Doe, Server: Server A, Action: DISCONNECT
```

![Splunk Data Manipulation 444.png|450](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation%20444.png)

Create a file `transforms.conf` - this file allows you to perform field transformations:

```c
[vpn_custom_fields]
REGEX = User:\s([\w\s]+) ..... (and so on)
FORMAT = Username::$1 Server::$2 Action::$3
WRITE_META = true
```

This is a custom identifier `vpn_custom_fields`, used the regex pattern to pull the **usernames**, **server**, and **action** from the logs, mentioned the field name as Username, Server, and Action, and asked to capture the first group by referring to it as `$1` and `$2` so on for the other fields.

The next step is updating the `props.conf`:

```C
[vpn_logs]
SHOULD_LINEMERGE = true
MUST_BREAK_AFTER = (DISCONNECT|CONNECT)
TRANSFORM-vpn = vpn_custom_fields // name of stanza from transform.conf
```

Creating and updating the `fields.conf` (app's local or system's local directory latter works) - this allows you to specify which what field we are going to extract from the logs basically telling Splunk to extract this field at indexed time but not making it as seperate events:

```c
[Username] // name of the field in transform.conf
INDEXED = true

[Server]
INDEX = true

....
```

Note you have to sometimes manually extract the fields in the interface:

![Splunk Data Manipulation-14.png](/img/user/cards/blue-team/siem/images/Splunk%20Data%20Manipulation-14.png)

That's it you are done.
### Example
---
Line not breaking properly:
![SplunkManipulation.png](/img/user/cards/blue-team/siem/images/SplunkManipulation.png)

- /opt/splunk/etc/apps/fixit/bin/network-logs - location of the script.

Fixing event boundaries by creating a `props.conf` at the system's local directory:

```C
[network_logs]
SHOULD_LINEMERGE = true
BREAK_ONLY_BEFORE = \[Network-log\]
TRANSFORM-network = network_custom_fields
```

Restart splunk with:

```C
sudo $SPLUNK_HOME/bin/splunk restart
```

After verifying and successfully establishing boundaries, next up is extracting custom fields:

```C
[network_custom_fields]
REGEX = User named (.+?) from (.+?) accessed the resource (.+?) from the source IP (\d{1,3}(?:\.\d{1,3}){3}) and country\s*\n(.+?) at:
FORMAT = Username::$1 Department::$2 Domain::$3 SRCIP::$4 Country::$5
WRITE_META = true
```

-  (.+?) - **Non-greedy** — avoids over-capturing when similar words appear later.

Then create `fields.conf`:

```C
[Username] 
INDEXED = true

[Department]
INDEX = true

[Domain]
INDEX = true

[SRCIP]
INDEX = true

[Country]
INDEX = true
```

