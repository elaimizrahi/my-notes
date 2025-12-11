- Link layer has the responsibility of transferring datagrams from one node to a **physically adjacent** node over a link
- Terminology:
	- hosts and routers: nodes
	- communication channels that connect adjacent nodes along communication path: links
		- wired
		- wireless
		- LANs
	- layer-2 packet: frame
		- encapsulates datagram

**Services**:
- **Framing, link access**
	- encapsulate datagram into frame, adding header and trailer
	- channel access if shared medium
	- MAC addresses in fram headers identify source and destination
- **reliable delivery**
	- already know how to do this!
  **Flow Control**
	- pacing between adjacent sending and receiving nodes
- **Error Detection**
	- errors caused by signal attenuation, noise
- **Error Correction**
	- receiver identifies and corrects bit errors without retransmission
- **Half and Full Duplex**
	- with half duplex, nodes at both ends of link can transmit, but not at same time

- Link layer is implemented in each and every host
- In a Network Interface Card (NIC) or on a chip
	- Ethernet, Wifi card, or chip
	- implements link, physical layer
- combination of hardware, software, and firmware

![[Pasted image 20251211103726.png]]

### Error Detection
- EDC: error detection and correction bits (redundancy)
	- Larger EDC gets better detection, but not 100% reliable
- D: data protected by error checking, may include header fields

**Parity Checking**
- Single bit parity - detecting single-bit errors
	- Even parity: set parity bit so there is an even number of 1's
- Two-dimensional bit parity - detect and correct single bit errors
	- Column and row parity, can find the error with this

![[Pasted image 20251211104049.png]]

**Internet Checksum**
- Goal: detect errors in transmitted segment
- Sender:
	- treat contents of UPD segment as sequence of 16-bit integers
	- checksum: addition - one's complement of sum - of segment content
	- checksum value put into UDP checksum field
- Receiver:
	- compute checksum of received segment
	- check if computed checksum equals checksum field value
	- not equal - error detected 
	- equal - maybe error still?

**Cyclic Redundancy Check (CRC)**
- more powerful error-detection coding
- D: data bits (given, think of these as a binary number)
- G: Bit pattern (generator) of r+1 bits (given)

![[Pasted image 20251211104408.png]]

![[Pasted image 20251211104440.png]]


## Multiple Access Protocols
- two types of "links"
	- point-to-point
		- P2P link between ethernet switch and host
		- PPP for dial-up access
	- broadcast (shared wire or medium)
		- old-fasioned eethernet
		- upstream HFP in cale-based access network
- single shared broadcast channel
- two or more simultaneous transmissions by nodes: interference
	- collision if node receives two or more signals at the same time
- distributed algorithm that determines how nodes share channel
- communication about channel sharing must use channel itself 
	- no out-of-band channel for coordination

**Ideally:**
- **given**: multiple access channel (MAC) of rate R bps
- **desiderata:** 
	- when one node wants to transmit, it can send at rate R
	- when M nodes want to transmit, each can send at an average rate of R/M
	- fully decentralized
		- no special node to coordinate, no clocks, synchronization
	- simple

**Taxonomy:**
- three broad classes:
	- channel partitioning
		- divide channel into smaller pieces (time, frequency, code)
		- allocate piece to ndoe for exclusive use
	- random access
		- channel not divided, allows collisions
		- recover from collisions
	- Taking turns
		- nodes take turns, but nodes with more to send can take longer turns


#### Protocols
**TDMA (Time Division Multiple Access)**
- access to channel in rounds
- each station gets fixed length slot (length = packet transmission time) in each go round
- unused slots go idle

**FDMA (Frequency division multiple access)**
- channel spectrum divided into frequency bands
- each station assigned fixed frequency band
- unused transmission time in bands go idle

**Random Access**
- When node has packet to send
	- transmit at full channel data rate R
	- no a priori coordination among nodes
- two or more transmitting nodes: collision
- random access MAC protocol specifies:
	- how to detect collisions
	- how to recover from collisions
- Exampels: ALOHA, CSMA, CSMA/CD, CSMA/CA

**Slotted ALOHA:**
- Assumptions: 
	- all frames same size
	- time divided into equal size slots
	- nodes start to transmit only slot beginning
	- nodes and synced
	- if 2+ nodes transmit in slot, all nodes detect collision
- Operation
	- When node obtains fresh frame, transmits in next slot
		- if no collision: node can send new frame
		- if collision: node retransmits frame in each subsequent slow with probability p until success

![[Pasted image 20251211105927.png]]

- Efficiency: long run fraction of successful slots
	- suppose: N nodes with many frames to send, each transmits in slow with probability p
		- prob that given node has success in a slot: $p(1-p)^{N-1}$
		- prob that **any** node has a success: $Np(1-p)^{N-1}$
		- max efficiency: find p* that maximizes: $Np(1-p)^{N-1}$
		- for many nodes, take limit of $Np*(1-p*)^{N-1}$
			- max efficiency = 1/e = .37
	- at best: channel used for useful transmissions 37% of the time

**Pure ALOHA:**
- unslotted ALOHA: simpler, no synchronizaiton
	- when first frame arrives, transmit immediately
- collision probability increases with no synchronization
	- frame sent at t0 collides with other frames sent in $[t_0-1, t_0+1]$
- Pure aloha efficiency: 18%

**CSMA (carrier sense multiple access)**
- simple CSMA: listen before transmit
	- if channel sensed idle: transmit entire frame
	- if channel busy: defer transmission
- dont interrupt others

**CSMA/CD**
- CSMA with collision detection
- collisions detected within short time
- colliding transmission aborted, reducing channel wastage
- collision detection easy in wired, difficult with wireless
- ask before 

- Collisions can still occur with carrier sensing
	- propagation delay means two nodes may not hear each other's just-started transmission
	- Collision meant the entire packet transmission time is wasted
	- Distance and propagation delay play role in determining collision probability
- 
![[Pasted image 20251211111138.png]]

**Ethernet CSMA/CD algorithm:**
1. NIC receives datagram from network layer, creates frame
2. If NIC senses channel, check if busy, if not send else wait until idle then transmit
3. If NIC transmits entire frame without collision, NIC is done with frame
4. If NIC detects another transmission while sending, abort, send jam signal
5. After aborting, NIC enters **binary backoff**:
	1. After *m*th collision, NIC chooses K at random from ${0, 1, 2, ..., 2^{m-1}}$
	2. NIC waits Kx512 bit times, returns to step 2
	3. more collisions: longer backoff interval

![[Pasted image 20251211111437.png]]
- Medium sensing is done for 96 bit-times
- Jamming signal length is 48 bits. Jamming signal creates enough energy on the medium for collision detection
- Tp is equated with 512 bit-times
- i saturates at 10

![[Pasted image 20251211111932.png]]

**Taking Turns Protocols:**
- Channel partitioning protocol:
	- share channel efficiently and fairly at high load
	- inefficient at low load, 1/N bandwidth allocated even if one active node
- Random access MAC protocol: 
	- efficient at low load: single node can fullt utilize channel
	- High load causes lots of collision

- Polling: 
	- Master node invites other nodes to transmit in turn
	- typically used with dumb devices
	- causes overhead, latency, single point of failure
- Token passing:
	- control token passed from one node to next sequentially
	- token message
	- same issues as pollling


![[Pasted image 20251211112435.png]]


## LANs
#### Addressing, ARP
- MAC addresses
	- 32-bit IP address
		- network layer address for interface
		- used for layer 3 (network) forwarding
	- MAC (or LAN or Physical or ethernet) address
		- used locally to get frame from one interface to another physically connected interface
		- 48-bit MAC address burned in NIC ROM
		- e.g. 1A-2F-BB-76-09-AD
	- each interface on LAN
		- has a 48-bit MAC address
		- has a locally unique 32-bit IP address
	- address allocation done by IEEE
	- manufacturer buys portion of MAC address space to assure uniqueness
	- Like social security number, IP is postal code
	- portable - can move interface from one LAN to another

- ARP
	- each IP node (host, router) on LAN has table
	- IP/MAC address mappings for some LAN nodes `<IP addresss, MAC address; TTL>`

![[Pasted image 20251211112954.png]]
2. B replies to A with ARP response, giving its MAC address
3. A receives B’s reply, adds B entry into its local ARP table

**Routing to another Subnet:**
- e.g. sending a datagram from A to B via R
	- A creates IP datagram with IP source A, destination B
	- A creates link-layer frame containing A-B IP datagram
		- R's MAC address is frame's destination
	- frame sent from A to R
	- frame received at R, datagram removed, passed up to IP
	- B receives frame, extracts IP datagram destination B
	- B passes datagram up protocol stack to IP

#### Ethernet
- main wired LAN tech
- 10Mbps - 400Gbps
- simpler, cheap
- two types:
	- bus: popular through mid 90's, all nodes can collide with each other
	- switched: used today
		- active link-layer 2switch in center
		- each spoke runs a separate ethernet protocol (nodes do not collide with each other)

![[Pasted image 20251211113631.png]]

![[Pasted image 20251211113639.png]]

- **addresses**: 6 bytes source, destination MAC addresses
	- if adapter receives frame with matching destination address, or with broadcast address, it passes data in frame to network layer protocol 
	- otherwise, adapter didscards frame
- **type**: indicates higher layer protocol
	- mostly IP but others possible
	- used to demultiplex up at receiver
- **CRC**: cyclic redundancy check at receiver
	- error detected: frame is dropped

Notes about ethernet:
- **Connectionless**: no handshaking between sending and receiving NICs
- **unreliable**: receiving NIC doesn't send ACK's or NAK's to sending NIC
- Ethernet's MAC protocol: unslotted CSMA/CD with binary backoff


![[Pasted image 20251211114230.png]]


#### Switches
- Switch is a link-layer device - takes an active role
	- store, forward ethernet frames
	- examine incoming frame's MAC address, selectively forward frame to one or more outgoing links when frame is to be forwarded on segment, uses CSMA/CD to access segment
- Transparent: host unaware of presence of switches
- plug and play: self-learning
	- switches do not need to be configured
- hosts have dedicated, direct connection to switch
- switches buffer packets
- Ethernet protocol used on each incoming link, so:
	- no collisions, full duplex
	- each link is its own collision domain
- switching: A to A' and B to B' can transmit simultaneously without collisions but A to A' and C to A' can't
			- ![[Pasted image 20251211114558.png | 300]]

- How does switch know A' reachable via interface 4, B' reachable via interface 5?
	- each switch has a switch table, each entry
		- `<Mac address of host, interface to reach host, time stamp>`
		- looks like a routing table

**Self-learning:**
	- Switch learns which hosts can be reached through which interfaces 
		- when frame received, switch learns location of sender: incoming LAN segment
		- records sender/location pair in switch table
![[Pasted image 20251211115154.png | 300]]
- Frame Filtering/Forwarding:
	- When frame received at switch: 
		- record incoming link, MAC address of sending host
		- index switch table using MAC destination address
		- if entry found:
			- if destination on segment from which frame arrived
				- drop frame
			- else
				- forward frame on interface indicated
		- else forward on all interfaces except arriving one
![[Pasted image 20251211115413.png]]

**Interconnecting Switches:**
- Self-learning switches can be connected together
- It works the exact same as the single-switch case for how a node knows to forward a frame

![[Pasted image 20251211115514.png]]

**Switches vs. Routers:**
- both are store-and forward
	- routers: network layer devices (examine network layer headers) 
	- switches: link-layer devices (examine link-layer headers)
- both have forwarding tables:
	- routers: computer tables using routing algorithms, IP addresses
	- switches: learn forwarding table using flooding, learning, MAC addresses


## Datacenter Networks
- 10's to 100's of thousandsd of hosts, often closely coupled and in close proximity
	- e-business (Amazon)
	- content-servers (Youtube)
	- search engines, data mining 
- challenges: 
	- multiple applications, each service massive numbers of clients
	- reliability
	- managing/balancing load, avoiding processing, networking, data bottlenecks

![[Pasted image 20251211115850.png]]

- Multipath: 
	- rich interconnection among switches and racks:
		- increased throughput between racks (multiple routing paths possible) 
		- increased reliability via redundancy

![[Pasted image 20251211120226.png]]

- Application-layer routing:
	- load balancer receives external client requests and directs the workload within the data center
- 
![[Pasted image 20251211120312.png]]
- Innovations:
	- link layer:
		- RoCE
	- transport layer:
		- ECN
		- hop-by-hop congestion control
	- routing, management:
		- SDN widely used


# Putting It All Together
- Scenario: student attaches laptop to campus network, requests/receives www.google.com
- **Client joins network → DHCP**: Laptop broadcasts DHCPDISCOVER; router replies with IP address, gateway, DNS server.
- **Client configures itself** with assigned IP, default gateway, DNS info.
- **Before DNS, ARP needed**: Client ARP-broadcasts to learn MAC of default gateway.
- **DNS query** sent via UDP → encapsulated in IP → Ethernet → forwarded to DNS server.
- **Routers forward DNS packet** using routing tables built by RIP/OSPF/BGP.
- **DNS server replies** with IP address of [www.google.com](http://www.google.com).
- **Client opens TCP connection** → sends SYN to server; server replies SYN-ACK; client sends ACK.
- **HTTP request** sent over established TCP connection.
- **Server processes request** and returns HTTP response (webpage) in TCP segments.
- **Datagrams flow back** through Google → ISP → campus → laptop, completing the request.