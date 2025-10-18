### Performance: loss, delay, throughput
How do packet delay and loss occur?
- packets queue in router buffers, waiting for turn for transmission
	- queue length grows when arrival rate to link termporarily exceedds output link capacity
- packet loss occurs when memory to hold queued packets fills up

**Packet delay**
Four sources:
1. nodal processing $d_{proc}$ - checking bit errors, finding output link, usually in microseconds
2. queueing delay $d_{queue}$ - time waiting at output link for transmision, depends on congestion level
3. transmission delay $d_{trans} = \frac{L}{R}$ - time to send transmission
4. Propagation delay $d_{prop} = \frac{d}{s}$ - length of physical link / propagation speed (~2x10^8 m/sec)
![[Pasted image 20251017175326.png]]

$$
\begin{matrix}
a = \text{average packet arrival rate} \\
L = \text{packet length (bits)} \\
R = \text{link bandwidth (bit transmission rate)} \\
\text{avg. queueing delay} = \text{traffic intensity} = \frac{aL}{R} = \frac{arrival\ rate}{service\ rate} \\\\
\text{if around 0, avg queueing delay small}\\
\text{if around 1, avg queueing delay large}\\
\text{if > 1, avg queueing delay infinite}
\end{matrix}
$$

Real delay and loss can be seen using the `traceroute` program. It sends 3 packets which are then returned to sender. The sender measures the time interval between transmission + reply

**Packet loss**
- queue (aka buffer) preceding link inn buffer has finite capacity
- when a packet arrives to a full queue it is dropped
- lost packet may be resent by prev node or by source end system, but possibly not at all

**Throughput**
- the rate (bits/time) at which bits are being send to the receiver
	- instantaneous: at a given point in time
	- average: over long period
	![[Pasted image 20251017180348.png | 400]]
- Note: throughput is like fluid, there exist bottlenecks and we have to account for them. We don't calculate by doing the average of all but instead the bottleneck

### Protocol Layers
Protocol layers allow us to standardize and organize the structure of our network
- Allows identification, relationship of the pieces that make up the structure
- Enables modularization which eases maintenance annd updating of system (change in layer service doesn't affect other services' implementation)

**Layers**
1. Application - support network applications (HTTP, DNS, etc.)
2. Transport - process-process data transfer (TCP, UDP)
3. Network - routing of datagrams from source to destination
4. Link - data transfer between neighboring network elements
5. Physical - bits "on the wire"
![[Pasted image 20251017180858.png]]

**Encapsulation**
- The design of the protocol layers are modularized so taht changes in one layer do not need changes in another layer
- Protects the data of upper layers
- But, an overhead in the form of transmitting $H$ is incurred
![[Pasted image 20251017181050.png]]
