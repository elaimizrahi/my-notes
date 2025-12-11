There are two key network layer functions
- **Forwarding**: Moving packets from a router's input link to appropriate router output link
- **Routing**: Determine the route taken by packets from source to destination using routing algorithms

**Data plane:** Local, per router functions that determine how datagrams arriving on the router input are forwarded
**Control Plane**: Network-wide login that determines how a datagram is routed among routers along end-end paths from source host to destination hosts
- Two Approaches: traditional routing algorithms and software defined networking

**Per-router control plane:** Individual routing algorithm components in each and every router interact in the control plane 

![[Pasted image 20251204165642.png]]
**Note: For the software-defined networking control plane, a remote controller above the control plane handles the forwarding tables**


### Router Architecture

**High-level overview**:

![[Pasted image 20251204165758.png]]
![[Pasted image 20251204165853.png]]

**Input port:**
- Physical layer does bit-level reception
- Link layer is for things like ethernet
- Decentralized switching:
	- Using header field values, lookup output port using forwarding table in input port memory
	- **Input port queueing**: if datagrams arrive faster than forwarding rate into switch fabric
	- **Destination-based forwarding**: forward based only on destination IP address (traditional)

#### Switching Fabrics
- Transfer packet from input link to appropriate output link
- **Switching rate:** rate at which packets can be transferred from inputs to outputs
	- Often measured as a multiple of input/output line rate
	- N inputs: switching rate N times line rate desirable

![[Pasted image 20251204170206.png]]
**Switching Via Memory:**
- First generation routers:
	- traditional computers with switching under direct control of CPU
	- packet copied to system's memory
	- speed limited by memory bandwidth (2 bus crossings per datagram)

![[Pasted image 20251204170906.png]]

**Switching via a bus:**
- datagram from input port memory to output port memory via a shared bus
- **bus contention:** switching speed limited by bus bandwidth

**Switching via interconnection network:**
- Crossbar, Clos networks, other interconnection nets initially developed to connect processors in mulitproccessor
- **Multistage switch**: nxn switch from multiple stages of smaller switches
- **exploiting parallelism**: 
	- fragment datagram into fixed length cells on entry
	- switch cells through the fabric, reassemble datagram at exit

**Input port Queueing**:
- If switch fabric is slower than input ports combined, then queueing may occur at input queues
	- queueing delay and loss due to input buffer overflow
- **Head-of-the-Line (HOL) Blocking**:
	- queued datagram at front of queue prevents others in queue from moving forward

![[Pasted image 20251204171545.png]]

## **Output port queueing**:
- **Buffering** is required when datagrams arrive from fabric faster than link transmission rate
	- Which datagrams do we drop if there's no free buffers
	- Datagrams can be lost due to congestion and lack of buffers
- **Scheduling Discipline** chooses among queued datagrams for transmission
	- Priority scheduling - who gets best performance, network neutrality

![[Pasted image 20251204171825.png]]
- Buffering when arrival rate via switch exceeds output line speed
- **queueing delay and loss due to output buffer overflow**

# Internet Protocol (IP)
![[Pasted image 20251204171940.png]]

IP Datagram format:
![[Pasted image 20251204172137.png]]
- Overhead: 20 Bytes of TCP, 20 Bytes of IP = 40 Bytes + application layer overhead for TCP + IP
IP Packet format: 
![[Pasted image 20251204172303.png]]
- **Version** (4 bits)
- **Header length** (4 bits): unit is 4-bytes (e.g. a 20 byte header is represented by 5 = 0101)
- **DSCP** (Differentiated Services Code Point)
	- Type of data carried - 46 = High priority, 0 = Low
- **Length (16 bits)**: Packet length in bytes
- **16-bit ID:** a long IP packet is fragmented into smaller packets - all those small packets carry the **same** 16-bit ID
- **3-bit flags**
- Fragment offset times 2^3 gives the position of the fragment in the original packet
- **TTL** = time to live
	- Max # of remaining hops
	- TTL is decremented by 1 at each router - once it's 0 the router discards the packet
- Upper layer: Upper layer proptocol to deliver payload to (ex. TCP = 6, UDP = 17)
- header Checksum: to detect bit errors
- **Source IP Address:** 32-bit IP of the node that created the packet
- **Destination IP address**: 32-bit IP of the node that will receive the packet

**IP Fragmentation/Reassembly**
- Network links have MTU (max transfer size) - largest possible link-level frame
- Large IP datagram divided within the net
	- one datagram becomes several datagrams
	- "reassembled" only at destination
	- IP header bits used to identify, order related fragments

![[Pasted image 20251204172939.png]]

### IP Addressing
- IP Addresses are 32-bit identifiers associated with each host or router **interface**
- **interface:** connection between host/router and physical link 
	- router's typically have multiple interfaces
	- host typically has one or two interfaces

**Subnets:**
- Subnets are device interfaces that can physically reach each other without passing through an intervening router
- IP addresses have structure:
	- **Subnet part:** devices in same subnet have common high order bits
	- **host part:** remaining low-order bits
- **Recipe for defining subnets:**
	- detach each interface from its host or router, creating "islands" of isolated networks
	- each isolated network is called a subnet
![[Pasted image 20251204173416.png | 500]]

**Classful Addressing (1981-1993):** Characterized by fixed-length prefixes 
- Class A: Address begins with 0 and is of the form: prefix.host.host.host
	- 8 bit prefix length
- Class B: Address begins with 10 and is of the form Prefix.Prefix.host.host
	- 16 bit prefix length
- Class C: Same but prefix length is 24 bits, prefix.prefix.prefix.host
- **Drawback**: Fixed number of networks and fixed number of addresses per network

**CIDR (Classless InterDomain Routing) (1993--)**:
- Subnet portion of address of arbitrary length
- address format: **a.b.c./x** where x is the number of bits in the subnet portion of the address

![[Pasted image 20251204173757.png]]
![[Pasted image 20251204173811.png]]

**Network ID and broadcast address:**
- Network/subnet ID appears in routing tables
	- Individual host IP addresses do NOT
- Broadcast address is used to perform IP-level broadcast
	- You send an IP packet with a broadcast address as the destination, the IP packed is delivered to ALL the nodes on the network

![[Pasted image 20251204174028.png]]

**How to get an IP address?**
- How does a host get an IP address in its network?
	- hard-coded by sysadmin in config file (/etc/rc.config in UNIX)
	- **DHCP (Dynamic Host Configuration protocol** dynamically get addresses from a server - "plug and play"
		- Goal: Host dynamically gets IP address from network server when it joins network
			- can renew its lease on address in use
			- allows reuse of addresses (only hold address while connected/on)
			- support for mobile users who join/leave network
- How does network get subnet part of IP address? 
	- Gets allocated portion of its provider ISP's address space 
	- ISP can then allocate out its address space in 8 blocks
	- **Hierarchical addressing** allows efficient advertisement of routing information
	- ![[Pasted image 20251204182528.png]]
	- Address aggregation (aka supernetting):
		- ![[Pasted image 20251204182553.png | 400]]

**DHCP Overview:**
- Host broadcasts DHCP discover msg (optional)
- DHCP server responds with DHCP offer msg (optional) 
- host requests IP address: DHCP request msg
- DHCP server sends address: DHCP ack msg
![[Pasted image 20251204180935.png]]
- DHCP can also return:
	- address of first-hop router for client
	- name and IP address of DNS server
	- network mask (indicating network versus host portion of address)


## Packet Forwarding 
- Packet forwarding is the relaying of packets from one physical interface to another by routers on the internet
- Next hop = f(destination address, routing table)
	- this is called table-driven routing

![[Pasted image 20251204181720.png]]

![[Pasted image 20251204181756.png]]

**Matching and forwarding algorithm**
- Inputs: IP address from packet header (call it IP1) and routing table (call it RT)
- Processing:
	- if (matching occurs between IP1 and RT) 
		- find the matching entry with the **longest prefix** 
		- forward the packet via the appropriate interface
	- else if (default entry exists in RT)  // default is 0.0.0.0/0
		- forward the packet via the appropriate interface
	- else
		- send error message (ICMP message) to the source of the IP packet

**Longest Prefix matching**
- When looking for forwarding table entry for given destination address, use the **longest** address prefix that matches destination address
#### Hierarchal Addressing
- Allows efficient advertisement of routing information
- How does the network get the subnet part of the IP address?
	- It gets allocated a portion of its provider ISP's address space
	- e.g. ISP's block **11001000 00010111 0001**0000 00000000 200.23.16.0/20
		- ISP can then allocate out its address space in 8 blocks:
			- Organization 0 **11001000 00010111 0001000**0 00000000 200.23.16.0/23
- **Address Aggregation**: Supernetting
	- Without this, the router would advertise 8 routers. With it, the router only advertises one route
	- The router ends up only advertising the /20 that it has, and ignores the last three bits so that it doesn't have to advertise 8 routers

![[Pasted image 20251209161644.png]]


### NAT (Network Address Translation) 
- All devices in the local network share just ONE IPv4 address as far as the outside world is concerned
- all datagrams leaving the local network have the same source NAT IP address but different source port numbers 
- all devices have 32-bit addresses in a private IP address space (10/8, 172.16/12, 192.168/16 prefixes) that can only be used locally
- advantages: 
	- just one IP address needed from provider ISP for all devices
	- can change addresses of host in local network without notifying outside world
	- can change ISP without changing addresses of devices in local network
	- security: devices inside local net not directly addressable or visible by outside world
- Implementation - NAT router must (transparently):
	- outgoing datagrams: replace (source IP address, port #) of every outgoing datagram to (NAT IP address, new port #)
		- remote clients/servers will respond using (NAT IP address, new port # as designation address)
	- remember (in Nap translation table) every (source IP address, port #) to (NAT IP address, new port #) translation pair
	- incoming datagrams: replace (NAT IP address, new port #) in destination fields of every incoming datagram with corresponding (source IP address, port #) stored in NAT table

![[Pasted image 20251204184119.png]]
- NAT is controversial:
	- routers "should" only process up to layer 3
	- address "shortage" should be solved by IPv6
	- violates end-to-end argument (port # manipulation by network layer device)
	- NAT traversal: what if client wants to connect to server behind NAT
- But nat is going to stay.
	- It's extensively used in home and institutional nets, 4G/5G cellular nets

## Routing Protocols
- The goal with routing protocols is to determine the good paths (routes) from sending hosts to the receiving host, through a network of routers
- Graph abstraction: link costs
	- we denote $c_{a,b}$ as the cost of a direct link connecting a and b
		- If no link exists we set it as infinity
	- graph $G = (N, E)$
		- N = the set of routers
		- E = the set of links

![[Pasted image 20251209162141.png]]

### Link-state routing algorithms
 - Dijkstra's link-state routing algorithm
	 - **centralized**: network topology, link costs known to all nodes
		 - accomplished via link state broadcast
		 - all nodes have same info
	- Computes least cost paths form one node (source) to all other nodes
		- gives forwarding table for that node
	- **iterative**: after k iterations, know least cost path to k destinations
- Notation:
	- $c_{x,y}$: direct link cost from node x to y, infinity if it doesn't exist
	- $D(v)$: current estimate of cost of least-cost-path from source to destination v
	- $p(v)$ predecessor node along path from source to v
	- $N'$: set of nodes whose least-cost-path definitively known

![[Pasted image 20251209162706.png]]

### Distance Vector Algorithm
- Based on Bellman-Ford equation (dynamic programming)
- Let Dx(y) = cost of least-cost-path from x to y
	- Then: 
		- $D_x(y) = min_v\{c_{x,v} + D_v(y)\}$
		-  note: min_v is the min taken over all neighbours v of x
- Key idea:
	- from time-to-time, each node sends its own distance vector estimate to neighbours
	- When x receives new DV estimate from any neighbor, it updates its own DB using B-F equation
	- under minor, natural conditions, the estimate Dx(y) converge to the actual least cost dx(y)

![[Pasted image 20251209163115.png]]

- When a link cost changes: 
	- Node detects local link cost change
	- Updates routing info, recalculates local DV (distance vector)
	- if DV changes, notify neighbours
	- **BUT** When a path becomes worse or unreachable, the bad news may take many iteration to propagate - and during that time routers can accidentally create routing loops
	- **The Count to Infinity Problem** - take X - A - B
		- **Before Failure:** all have correct routes (A -> X = 1, B -> X = 2 (through a))
		- **After link X - A fails:** A updates its table (A -> X = ∞), but B still thinks A has a good path to X
			- So B's table says B -> X = 2
		- **A gets B's bad information**: A sees B claiming it can reach X with cost 2 so A incorrectly updates to (A -> X = 3) 
		- **Cycle:** This continues (increments B by one, then A by one) until infinite
	- **Solutions:**
		- Split horizon: if node B learns about a path to X from A, subsequently, B does not tell A about this
		- Poisoned Reverse: If A tries to route to X via B, A tells B that it has an infinity cost path to X when the (X, A) link is broken
			- This enables B to not try to forward the packet to X via A

## Scalable Routing
- Can't store all billions of destinations in routing tables - the exchange alone would swamp links
- Aggregate routers into regions known as "Autonomous Systems (a.k.a domains)" 
	- **Intra-AS**: aka intra-domain, routing among within same AS (network)
		- All routers in AS must run same intra-domain protocol
		- routers in different AS can run different intra-domain routing protocols
		- **gateway router:** at edge of its own AS, has link to routers in other AS'es
	- **Inter-AS** aka inter-domain: routing among AS'es
		- gateways perform inter-domain routing as well as intra-domain routing

**Most Common Intra-AS routing protocols:**
- **RIP:** Routing Information Protocol - DV's exchanged every 30 secs, not used anymore
- **EIGRP:** Enhanced Interior Gateway Routing Protocol - DV based, formerly Cisco proprietary for decades
- **OSPF**: Open Shortest Path First, link-state routing, IS-IS protocol 

**OSPF:** Open Shortest Path First Routing
- "Open" = publicly available
- classic linl-state
	- each router floods OSPF link-state advertisements (directly over IP) to all other routers in entire AS
	- multiple link costs metrics possible: bandwidth, delay
	- Each router has full topology, uses Dijkstra's algorithm to compute forwarding table
- **security:** all OSPF messages authenticated

**Hierarchal OSPF:**
- Two-level hierarchy: local area, backbone
	- link-state advertisements flooded only in area, or backbone
	- each node has detailed area topology; only knows direction to reach other destinations

![[Pasted image 20251209170030.png]]



## BGP - Routing Among ISP's
- BGP (Border Gateway Protocol): *The* de facto inter-domain routing protocol
	- Glue that holds the internet together
- allows subnet to advertise its existence, and the destinations it can reach, to rest of internet
	- "I am here, here is who I can reach, and how"
- BGP provides each AS a means to:
	- **eBGP:** obtain subnet reachability information from neighbouring AS's
	- **iBGP**: propagate reachability information to all AS-internal routers
	- determine good routes to other networks based on reachability information and policy

![[Pasted image 20251209170424.png]]

- BGP Session: Two BGP routers (peers) exchange BGP messages over semi-permanent TCP connection
	- advertising paths to different destination network prefixes 
- When an AS advertises its path to X, it promises it will forward the datagrams towards X

**Path Attributes and BGP routes**:
- BGP advertised route: prefix and attributes
	- prefix: destination being advertised
	- two important attributes
		- **AS-PATH**: list of ASes through which profix adveritsement has passed
		- **NEXT-HOP**: indicates specific internal AS router to next-hop AS
- Policy-based routing:
	- Gateway receiving route advertisement uses import policy to accept/decline path
	- AS policy also determines whether to advertise path to other neighbouring AS's

**BGP Route Selection:**
- router may learn about more than one route to destination AS, selects route based on:
	1. local preference value attribute: policy decision
	2. shortest AS-PATH
	3. closest NEXT-HOP router
	4. additional criteria


![[Pasted image 20251209171055.png]]

**BGP Messages:**
 - BGP messages are exhanged between peers over a TCP connection
 - message:
	 - **OPEN**: opens TCP connection and authenticates
	 - **UPDATE:** advertises new path (or withdraws old) 
	 - **KEEPALIVE**: keeps connection alive in absence of UPDATES, also ACKs OPEN request
	 - **NOTIFICATION:** reports errors in previous msg, also used to close connections


**Why different Intra and Inter AS routing:**
- Policy:
	- inter-AS: admin wants control over how its traffic is routed, who routes through its network
	- intra-AS: single admin, so policy is less of an issue
- Scale:
	- hierarchical routing saves table size, reduced update traffic
- Performance:
	- intra-AS: can focus on performance
	- inter-AS: policy dominates over performance

![[Pasted image 20251209171453.png]]

![[Pasted image 20251209171529.png]]

![[Pasted image 20251209171533.png]]

