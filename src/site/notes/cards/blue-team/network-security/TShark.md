---
{"dg-publish":true,"permalink":"/cards/blue-team/network-security/t-shark/","tags":["blue-team/network-analysis/tshark"]}
---

[[atlas/Network-Traffic Analysis\|Network-Traffic Analysis]]

A command line version of Wireshark that is suitable for data carving, in-depth packet analysis, and automation with scripts.

- `tshark -D` - shows available interface to sniff.
- `tshark -i 1 or ens5` - sniff a specific interface.
- `-r demo.pcapng` - read from a capture file.
- `-c 10` - stop after capturing a specified number of packets / stop after capturing/filtering/reading 10 packets.
- `-w sample-capture.pcap` - Write/output function. Write the sniffed traffic to a file.
	- Great for seperating specific packets from filters.
- `-V` - Verbosity is great after filtering the packet.
- `-q` - Supress the packets outputs on the terminal.
- `-x` - Show packet details in hex and ASCII dump for each packet.
- `| nl` to replace packets with filters with an actual count

#### Capture Conditions
---
- `tshark -w test.pcap -a duration:1` - stop after X seconds.
	- `-a filesize:10` - stop after X kb.
	- `-a files:3` stops after writing X amount of output files
- `-b` - infinite loop

| Filters             | Descriptions                                                                                                                                                          |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Capture Filters** | Live filtering options. The purpose is to **save** only a specific part of the traffic. It is set before capturing traffic and is not changeable during live capture. |
| **Display Filters** | Post-capture filtering options. The purpose is to investigate packets by **reducing the number of visible packets**, which is changeable during the investigation.    |
Again, Tshark is simply a command line version of Wireshark.
## Capture Filters

- Filtering a host
	- `tshark -f "host 10.10.10.10"`
- Filtering a network range 
	- `tshark -f "net 10.10.10.0/24"`
- Filtering a Port
	- `tshark -f "port 80"`
- Filtering a port range
	- `tshark -f "portrange 80-100"`
- Filtering source address
	- `tshark -f "src host 10.10.10.10"`
- Filtering destination address
	- `tshark -f "dst host 10.10.10.10"`
- arp | ether | icmp | ip | ip6 | tcp | udp
	- Filtering TCP
	- `tshark -f "tcp"`
- Filtering MAC address
	- `tshark -f "ether host F8:DB:C5:A2:5D:81"`

We can use `netcat`, `curl` and other network related tools to practice or test our filters and you can read more on capture filter syntax [here](https://www.wireshark.org/docs/man-pages/pcap-filter.html) and [here](https://gitlab.com/wireshark/wireshark/-/wikis/CaptureFilters#useful-filters).
## Display Filters

- [Display filter reference](https://www.wireshark.org/docs/dfref/) here.
- Ideally single quotes for filter is recommended.

- Filtering an IP without specifying a direction.  
	- `tshark -Y 'ip.addr == 10.10.10.10'`
- Filtering a network range 
	- `tshark -Y 'ip.addr == 10.10.10.0/24'
- Filtering a source IP
	- `tshark -Y 'ip.src == 10.10.10.10'`
- Filtering a destination IP
	- `tshark -Y 'ip.dst == 10.10.10.10'`
- Filtering TCP port  
	- `tshark -Y 'tcp.port == 80'`
- Filtering source TCP port
	- `tshark -Y 'tcp.srcport == 80'`
- Filtering HTTP packets  
	- `tshark -Y 'http'`
- Filtering HTTP packets with response code "200"
	- `tshark -Y "http.response.code == 200"`
- Filtering DNS packets  
	- `tshark -Y 'dns'`
- Filtering all DNS "A" packets (DNS queries)
	- `tshark -Y 'dns.qry.type == 1'`

### Implementing Wireshark Features
---
- `tshark --color` colorised output.

**Statistics**
-  Supress packets only show statistics
	- `-q` 
-  Show protocol hierarchy.
	- `tshark -z io,phs -q -r file`
	- focus on specific protocol `-z io,phs,udp` in this case **udp**.
- Shows the distrubtion of packets in a tree view - detect big and small packets at a glance:
	- `-z plen, tree`
- Overview of unique endpoints and number of packets associated with them.
	- `-z endpoints,ip or (eth, ip, ipv6, tcp, udp, wlan)` 
- Shows traffic flow or which IP is communicating to which.
	- `-z conv,ip or (...)` 
- Expert view by wireshark.
	- `-z expert` 

**Tree view**

| Key     | Description                                                                                                                                                                                        |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Rate    | Rate of events per milisecond                                                                                                                                                                      |
| Average | The average response time or latency                                                                                                                                                               |
| Percent | The percentage of the total traffic or packets that each IP address represents. if an IP address has 10% in this column, it means that 10% of all packets observed were related to that IP address |

- Shows protocol statistics (tcp/udp).
	- `-z ptype,tree -q` 
- Shows summary of hosts in a single view.
	- `-z ip(v6)_hosts,tree -q` 
- Summary v4,6 of src and destination:
	- `-z ip(v6)_srcdst,tree -q`
- Show outgoing traffic and ports used
	- `-z dests,tree -q`
	- `-ipv6_dests,tree -q`
- Packet and status counter for HTTP:
	- `-z http,tree -q`
- Statistics for DNS:
	- `-z dns,tree -q`
- Packet and status counter for HTTP2:
	- `-z http2,tree -q`
- Load distribution: 
	- `-z http_srv,tree -q`
- Requests:
	- `-z http_req,tree -q`
- Requests and responses: 
	- `-z http_seq,tree -q`

**Streams, objects, and credentials**:

| **Main Parameter** | **Protocol**                        | **View Mode**    | **Stream/Packet Number** | **Additional Parameter** |
| ------------------ | ----------------------------------- | ---------------- | ------------------------ | ------------------------ |
| -z follow          | - TCP<br>- UDP<br>- HTTP<br>- HTTP2 | - HEX<br>- ASCII | 0 \| 1 \| 2 \| 3 ...     | -q                       |

- Follow Stream (refer to table above) (remember it starts with 0)
	- `-z follow,tcp,ascii,0 -q`
- Export Objects
	- Available protocols: (DICOM, HTTP, IMF, SMB, TFTP)
	- `--export-objects http,/home/ubuntu/folder1 -q`
- View credentials:
	- Available protocols:  (FTP, HTTP, IMAP, POP and SMTP)
	- `-z credentials -q`

**Contains, matches, and extract fields**:

- Using hex and regex values instead of ASCII has a better chance.
- Do not use `-q`.
- Extract fields and show field name.
	- `-T fields -e ip.src -e ip.st -E header=y`
- View http body with form
- ` -Y 'http' -e http.file_data -e urlencoded-form.key -e urlencoded-form.value`
- Search a value inside a packet, case sensitive
	- `-Y 'http.server contains "Apache"`
- Search a pattern of regular expression, case-insenstive
	- `-Y 'http.request.method matches "(GET|POST)"`

## Use Cases
When investigating a case, a security analyst should know how to extract hostnames, DNS queries, and user agents to hunt low-hanging fruits after viewing the statistics and creating an investigation plan

- Extract Hostnames (With linux utilities for better format)
	- `-T fields -e dhcp.option.hostname`
- Extract DNS queries
	- `-T fields -e dns.qry.name`
- Extract User Agents 
	- `-T fields -e http.user_agent`

Better formatting add this:
- `| awk NF` - removes empty lines.
- `| sort -r` - sort recursively before handling values.
- `| uniq -c` - Show unique values and number of occurences.
- `| sort -f` - Sort from high occurences to less.

**Something that I would start off:**
- Shows summary of hosts in a single view.
	- `-z ip(v6)_hosts,tree -q` 
- Viewing specific packet details
	- ` -Y 'frame.number == 11' -V`
- View all port numbers used:

```bash
tshark -nlr "web server.pcap" -Y "tcp.flags.syn==1 && tcp.flags.ack==0" -T fields -e ip.src -e ip.dst -e tcp.dstport | sort | uniq 
```

