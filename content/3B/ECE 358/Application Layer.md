The application layer gives us the ability to write programs that run on different end systems and communicate over the network.

## Overview
#### **Client-server paradigm**:
- Server
	- always-on host
	- permanent IP address
	- often in data centers for scaling
- Clients
	- contact + communicate with the server
	- may be connected intermittently 
	- may have dynamic IP addresses
	- do *not* communicate directly with each other
- Example of this paradigm: HHTP, IMAP, FTP

#### **Peer - peer architecture**
- **No** always-on server
- arbitrary end systems directly communicate
- peers request service from other peers + provide service in return to other peers
- peers are intermittently connected and change IP addresses
- Example is P2P file sharing

#### **Process communication**
Note: a process is a program running within a host
- within the same host, two processes communicate using **inter-process communication** - defined by OS
- processes in different hosts communicate by sending **messages**
	- **client process** initiates communication
	- **server process** waits to be contacted
*Note: P2P apps have both client and server processes*

##### Sockets
- process sends/receives messages through its socket (like a door)
	- sending process shoves message out the door
	- sending process relies on outside infrastructure to deliver
	- The receiving process will get the message through its socket

##### Process Addressing
- to receive messages, process needs a unique identifier
	- The host device usually has a **unique 32-bit IP address**
- The **identifier** includes both the IP address and port numbers associated with the process
	- e.g. 129.97.56.100:8000 (8000 is port, other part is IP)

**Application Layer Definitions**
The application layer has many things it defines:
- types of messages exchanged (req, res)
- message syntax
- messages semantics
- rules for when + how processes send + respond to messages
- open protocols, e.g. HTTP, STMP
- proprietary protocols - e.g. skype, zoom

**Transport services required by an app**
- data integrity
	- some need 100% reliable data transfer
	- others (audio) can tolerate some loss
- timing
	- some need low delay (multiplayer games)
- throughput
	- multimedia doesn't need much, but elastic apps use as much as possible
- security
	- encryption, data integrity, etc.


## Web + HTTP
**Reminder**:
- Web page has objects stored on web servers (HTML file, JPEG img, etc.)
- each referenced object is addressable by a uniform resource locator (URL)

#### HTTP 
Hypertext transfer protocol
- Web's application-layer protocol
- Stateless - server doesn't maintain information about past requests
- client-server model:
	- client: browser that requests, receives, and displays web objects
	- server: web server sends objects in response to requests

Uses **TCP** as a transport protocol
- client initiates TCP connection (creates socket)
- server accepts TCP connection form client
- HTTP messages exchanged between browser and server
- TCP closed

| Non-persistent HTTP                                     | Persistent HTTP                                                   |
| ------------------------------------------------------- | ----------------------------------------------------------------- |
| 1. TCP Opened                                           | TCP opened                                                        |
| 2. at most one object sent over TCP                     | multiple objects can be sent                                      |
| 3. TCP closed                                           | TCP closed                                                        |
| RTT: time for a small packet to travel to server + back |                                                                   |
| **HTTP response time = 2RTT + file transmission time**  |                                                                   |
| issues: requires 2RTT's per object                      | server leaves connection open                                     |
| OS overhead for each TCP connection                     | subsequent HTTP between same client/server                        |
| browsers often open multiple connections in parallel    | as little as one RTT for all objects (cuts response time in half) |

#### HTTP Request Methods

- **GET** – Retrieves data from the server.  
    Example: `www.somesite.com/animalsearch?monkeys&banana`  
    → Sends parameters in the **URL** after `?`.
    
- **POST** – Sends data to the server in the **request body** (often from form input).
    
- **HEAD** – Requests only the **headers** of a resource (no body).  
    Useful for checking metadata (e.g., file size, last modified).
    
- **PUT** – Uploads or **replaces** a resource at the specified URL.  
    The content of the resource is sent in the **entity body**.
    

---

#### HTTP Response Message

```
HTTP/1.1 200 OK Date: Tue, 08 Sep 2020 00:53:20 GMT Server: Apache/2.4.6 (CentOS) Last-Modified: Tue, 01 Mar 2016 18:57:50 GMT ETag: "a5b-52d015789ee9e" Content-Length: 2651 Content-Type: text/html; charset=UTF-8  <data ... e.g., HTML file>`

```

| Code                               | Meaning      | Description               |
| ---------------------------------- | ------------ | ------------------------- |
| **200 OK**                         | Success      | Request succeeded         |
| **301 Moved Permanently**          | Redirect     | Resource moved to new URL |
| **400 Bad Request**                | Client Error | Malformed request         |
| **404 Not Found**                  | Client Error | Resource doesn’t exist    |
| **505 HTTP Version Not Supported** | Server Error | Version not supported     |


## Domain Name System (DNS)
Distributed database implemented in the hierarchy of many name servers
- An application-layer protocol in hosts, DNS servers communicate to resolve names (address -> name)
- DNS is a core internet function 

**DNS Services**:
- hostname to IP translation
- host aliasing (google.com vs IP address)
- main server aliasing
- load distribution - many IP addresses correspond to one name

*Note: Not centralized due to volume, failures, distance, and maintenance*
- e.g. Comcast DNS servers take 600B DNS queries per day

**DNS Servers handle trillions of queries/day that are mostly reads and prioritize performance first**

DNS is a distributed hierarchal database
- If a client wants an IP (ie. www.amazon.com), starts with root DNS servers to find .com server
- Then finds the amazon.com DNS server
- Then queries the amazon DNS server to get IP addresses that it could server out 
![[Pasted image 20251017215108.png]]

 **Root DNS Servers:**
- official contact of last resort 
- Incredibly important for internet function
- Handled by ICANN, which manages the root DNS domain
**Top Level Domain (TLD) and authoritative servers:**
- responsible for things like .com, .net, .org, etc. 
**Local DNS servers:**
- when host makes DNS query, it is sent to local NDS server
- Local server returns reply, answering from its local cache or by forwarding to DNS heirarchy

#### DNS Name Resolution
e.g. host at ece.uwaterloo.ca wants IP for gaia.cs.umass.edu
##### Iterated query
- contacted server replies with name of server to contact
- forwards to another server 
![[Pasted image 20251017215702.png | 300]]

##### Recursive query
- Puts burden of name resolution on contacted server
- heavy load at upper levels of heirarchy
![[Pasted image 20251017215816.png | 300]]

#### Caching DNS Information
Once any name server learns a mapping, it caches the mapping and immediately returns a cached mapping in response to a query.
- Improves response time
- Caches get removed after some time (TTL)
- TLD servers typically cached in local name servers

#### DNS Records
DNS being a distributed database must store records. It does this with resource records (RR) in the form: `(name, value, type, ttl)`
**Type = A**
- `name` is hostname + `value` is IP address `(relay1.bar.foo.com,145.37.93.126,A)`
**Type = NS**
- `name` is domain + `value` is hostname of name server for this domain (forwarded)
- `(foo.com, dns.foo.com, NS)`
**Type = CNAME**
- `name` is alias for some real name (`www.ibm.com is really servereast.backup2.ibm.com`)
- `value` is canonical name (a type A DNS record)
**Type = MX**
- `name ` is like normal
- `value` is the name of the SMTP mail server associated with `name`
- `(foo.com, mail.bar.foo.com, MX)`

#### DNS Protocol Messages
DNS *query* and *reply* messages both have the same format

message header:
- **identification:** 16 bit # for query, reply uses the same #
- **flags**
	- query or reply
	- recursion desired
	- recursion available
	- reply is authoritative
- **questions**
	- name, type fields for a query
- **answers**
	- RR's in response to a query
- **authority**
	- records for authoritative servers
- **additional info**
	![[Pasted image 20251017221000.png | 300]]

#### Getting info into the DNS
- register name at a DNS registrar (e.g. Network Solutions)
	- provide names, IP addresses or authoritative name server (primary and secondary)
	- Registrar adds NS and A RR's into the TLD server
- create authoritative server locally with IP address `212.212.212.1`
	- type A record for your domain
	- type MX record for the domain (minus www.)
