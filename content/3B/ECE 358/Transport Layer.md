The transport layer's goal is to provide logical communication between the application processes running on different hosts. The sender will break application messages into segments, which are passed to the network layer. The receiver reassembles the segmenets into messages, passing them to the application layer

**We have two transport protocols available to internet applications - TCP and UDP**

### Transport Control Protocol (TCP)
- reliable in order delivery
- congestion control
- flow control 
- connection setup

### User Datagram Protocol (UDP)
- unreliable unordered delivery
- no-frills extension of "best-effort" IP

**Note: neither TCP or UDP guarantee delay or bandwidth**

## Multiplexing + Demultiplexing
Multiplexing at the sender handles data from multiple sockets and adds transport header. Demultiplexing at the receiveer uses header info to deliver received segments to correct socket.
![[Pasted image 20251017222608.png | 500]]

#### Demultiplexing
To demultiplex:
- host receives IP datagrams
	- each datagram has source + destination IP addresses
	- each datagram carries one transport-layer segment
	- each segment has source + destination port number
	- ![[Pasted image 20251017222807.png | 200]]
**Two kinds of ports:**
- Reseved ports (i.e. 80/HTTP)
- Free ports (anything else, need to talk to admins)

**Connectionless Demultiplexing**
Remember that when creating a socket, we had to specify the host-local port number
- when creating datagram to send into the UDP socket, we need to specify the destination IP address and port number

Then when the receiving host gets the UDP segment:
- checks destination port number in segment
- directs UDP segment to socket with that port number

**Note: IP/UDP datagrams with same dest. port number but different IP addresses and/or source port numbers will be redirected to the same socket at the receiving host**
![[Pasted image 20251017223321.png]]

**Connection-Oriented Demultiplexing**
In Connection-Oriented Demultiplexing, the TCP socket is identified by a 4-tuple:
1. source IP address
2. source port number
3. destination IP address
4. destination port number
The demux receiver uses **all four values** to direct segment to appropriate socket
![[Pasted image 20251017223513.png]]
##### Summary
-  Multiplexing, demultiplexing: based on segment, datagram header field values
- UDP: demultiplexing using destination port number (only)  
- TCP: demultiplexing using 4-tuple: source and destination IP addresses, and port numbers  
- Multiplexing/demultiplexing happen at all layers

## UDP: User Datagram Protocol
UDP is the "no frills", "barebones" internet transport protocol that gives us a "best effort" despite segments possibly being lost or out of order. It does not do any handshaking and each segment is handled independently.

**Benefits:**
- no connection establishment (which would add delay)
- no connection state at sender, receiver
- small header size
- no congestion control - it can shoot off messages as fast as desired
- can function in the face of congestion

**Common uses**:
- streaming multimedia apps
- DNS
- SNMP

**UDP segment header**
![[Pasted image 20251017224137.png | 400]]

## Principles of reliable data transfer
Packets are transmitted over unreliable channels which can corrupt the packets. Packets may be duplicated, sent out of order, or even be lost

Automatic Repeat Request (ARQ) protocols are used to provide reliable data transfer 

Reliable data transfer requires feedback from the receiver to the sender (Acknowledgement, aka ACK)

Sequence numbers are used to solve the problem of out of order and duplicate packets

Timers are used to solve the problem of lost packets

##### Stop-and-wait operation
The main idea is to send a packet, start a timer, and wait for an acknowledgement (ACK). If the timer times out before receiving the ACK, retransmit
![[Pasted image 20251017231715.png | 300]]

**Performance**: $U_{sender} (utilization) = \text{fraction of time sender busy sending}$
e.g. 1Gbps link, 15ms prop. delay, 8000 bit packet
$D_{trans} = \frac{L}{R} = \frac{8000 bits}{10^9 \text{bits/sec}} = 8\ \text{microsecs}$

![[Pasted image 20251017232130.png |300]]
- notice, this makes a 270Kbps throughput on a 1Gbps link - this sucks
- Protocol limits performance of underlying infrastructure - only 0.027% busy which is bad we want high utilization

#### Pipelined protocols operation
**Pipelining** allows the sender to send multiple "in-flight", yet-to-be-ACKed packets
- range of sequence numbers must be increased
- buggering at sender and/or receiver
![[Pasted image 20251017232538.png|400]]

##### Methods of pipelined protocols

**Go-Back-N sender**
- The sender can have a "window" of up to N, consecutive transmitted but unACKed packets
	- the packet header holds a k-bit sequence number
- **culmulative ACK**: ACK(n): ACKs all packets up to, including seq number $n$
	- On receiving ACK(n): move window forward to begin at n+1
- timer for oldest in-flight packet
- timeout(n): retransmit packet n and all higher seq number packets in window
- ACK-only: always send ACK for correctly-received packet so far, with highest in-order sequence number (might generate duplicate ACKs)
- on out-of-order packet
	- can discard or buffer: implementation decision
	- re-ACK packet with highest in-order sequence number
	- 
![[Pasted image 20251018152849.png]]


**Selective repeat**
- receiver individually acknowledges all correctly received packets
	- buffers packets as needed to eventually deliver in-order to upper layer
- sender times out/retransmits individually for unACKed packets
	- sender keeps timer for each packer
- Window: N consecutive seq numbers, limits seq numebrs of sent but unACKED packets

| sender                                                                           | Receiver                                                                                  |
| -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| if next available seq number in window, send packet                              | if packet n in order, send ACK(n) and also ones in buffer with m < n, else, add to buffer |
| on timeout(n), resend packet n + restart timer                                   | if packet n with seq number in the prev window, send ACK(n)                               |
| ACK(n) marks packet n as received + moves window if n is smallest unACKed packet | otherwise ignore                                                                          |
![[Pasted image 20251018153534.png]]
**Note**: the window size should be equal to or less than half of the sequence number space

**Performance of sliding window**
Say that $Tx$ and $Rx$ have agreed on a window size$ $W$
Let $L$ be the frame size

If $t_{T} = RTT + \frac{L}{R}$ is the total time to send a data frame and receive the corresponding ACK

Then the sender will send:
- $W$ frames if $W\frac{L}{R} < t_{T}$
- an upper bound of $W_{upper-bound} = \frac{t_T}{t_I}$ if $W\frac{L}{R} > t_{T}$
	- $t_I$ is the time to transmit one frame ($\frac{L}{R}$)
So the utilization is $U = min(1, W\frac{t_I}{t_T})$

Ideal size for 100% allocation: $W_{upper-bound} = \frac{t_T}{t_I}$


