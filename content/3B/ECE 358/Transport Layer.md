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


## Transmission Control Protocol (TCP)
Features of TCP:
- point-to-point (one sender + one receiver)
- reliable in-order **byte** stream
- cumulative ACK's
- pipelining - TCP congestion + flow control set window size
- full duplex data (bi-directional and max segment size)
- connection-oriented (uses handshaking)
- flow controlled - sender will not overwhelm receiver

![[Pasted image 20251018160853.png]]

Sequence numbers come as a byte stream "number" 
Acknowledgements (ACKs) holds the sequence number of the next byte it expects (cumulative ACK)

**Note:** TCP doesn't specify how to handle out-of-order segments, this is up to the implementor if they want to discard or buffer
- usually buffer is used

![[Pasted image 20251018161257.png]]

**ACK Generation:**

| Event at receiver                                                     | TCP receiver action                                                        |
| --------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| in-order arrival with expected seq #, all data up to # ACKed          | Delayed ACK, waits up to 500ms for next segment, if not received sends ACK |
| in-order arrival with expected seq #, another segment has ACK pending | immediately send single cumulative ACK (ack both in-order segments)        |
| out-of-order arrival, higher seq # than expected (Gap)                | immediately send duplicate ACK indicating seq # of next expected byte      |
| arrival of segment that partially or fully fills gap                  | immediately send ACK provided that segment is at lower end of gap          |
**TCP fast retransmit** - if sender gets 3 additional ACKs for same data (4 total), resend unACKed segment with smallest seq #

**Flow Control**
Q: How do we handle it if the network layer delivers faster than application layer removes from socket buffers?

A: *The receiver controls the sender*

- The receiver advertises free buffer space in the `rwnd` field in the TCP header
	- `RcvBuffer` sets size via socket options (default 4096 bytes)
	- many OS's autoadjust to RcvBuffer
- Sender limits amount of unACKed (in-flight) data to received `rwnd` 
- guarantees will not overflow

**Connection Establishment**
Connection establishment hinges on these fields:
- Connection Request **(SYN)**
- Initial Sequence Number **(ISN)**
- Acknowledgement **(ACK)**
- Receive window size
![[Pasted image 20251018163505.png]]

1. SYN segment from client to server
	- SYN = 1
	- A random ISN
	- RWND is undefined
	- some options
2. SYN + ACK segment from server to client
	- SYN = 1
	- A random ISN for client
	- ACK = 1
	- ack seq number is the sequence # of the first data byte to be received
	- RWND - receive window size
3. ACK from client to server
	- ACKs the second SYN segment
	- RWND - receive window size for server

To close, server and client each close their side, send TCP segment with FIN bit = 1

## Congestion Control
How we control the case where too many sources are sending too much data too fast for the network to handle.
- creates long delays, packet loss, and is **different from flow control**

**End-End congestion control**
- no explicit feedback from the network
- congestion gets inferred from observed loss + delay

**Network-assisted congestion control 
- routers provide direct feedback to sending/receiving hosts with flows passing through congested routers 
- may tell server/client the congestion levels, might just give a sending rate to follow
- TCP ECN, ATM, DECbit protocols

## TCP Congestions Control (AIMD)
*Approach*: senders can increase sending rate until packet loss (congestion) occurs. Then a decreasing rate is sent on a loss event

**Additive Increase:** increase sending rate by 1 maximum segment size every RTT until loss detected (when all segments in previous were ACKed)
**Multiplicative Decrease**: cut sending rate in half at each loss event (triple duplicate ACK), cut to 1 maximum segment size when loss from timeout

**Behaviour**: roughly send `cwnd` bytes, wait RTT for ACKS, then send more bytes
$$ TCP\ rate = \frac{cwnd}{RTT} bytes/sec$$
$$ LastByteSent - LastByteACKed \leq min(cwnd, rwnd)$$
`cwnd` is dynamically adjusted in response to observed congestion

By using the AIMD algorithm, the connection starts at a slow rate then ramps up exponentially fast

**Congestion avoidance**
- when `cwnd` gets to 1/2 of its value before timeout, we should switch the exponential increase to linear
	- Uses a variable `ssthresh` which is set to 1/2 of `cwnd` just before loss event

**Note:** there is TCP RENO which follows this, but TCP Tahoe always sets cwnd to 1 (doesn't cut in half on 3 duplicate ACKs)

**TCP Throughput**
When calculating this we ignore the slow start, assume there is always data to send

$$ avg.\ throughput = \frac{3}{4}W\ per\ RTT$$
**TCP CUBIC**
We can increase the amount of time being sent in usable bandwidth 

If $W_{max}$ is the sending rate where loss was detected:
- cut the rate/window in half on loss, ramp up to $W_{max}$ faster, but then approach it slower
How?
- Set $K$ as the point in time when the window size will reach $W_{max}$ (tuneable)
- Increase W as a function of the **cube** of the distance between current time and $K$ (larger increase when farther from K)
**Note:** TCP cubic is the most popular for web servers, and is used in linux