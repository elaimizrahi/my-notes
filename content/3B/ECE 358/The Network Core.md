The network core is a mesh of interconnected routers. The hosts break the application-layer messages into packets, which are then forwarded from one router to the next. These packets travel across links from the source to destination.

**Functions**
1. **Forwarding** *aka switching*
	- **local action:** moves arriving packets from the router's input link to the appropriate router's output link
	- uses a local forwarding table to figure out where to forward packet 
2. **Routing** 
	- **global action**: determines source-destination paths takes by packets
	- Uses routing algorithms to achieve this
				![[Pasted image 20251017170200.png | 300]]

#### Packet-switching
**Store-and-forward**
- wait for and stores all packets at the current location until done then sends to destination
$$ packet\ transmission\ delay\ = \frac{L}{R}seconds$$
- Example: 
$$
\begin{matrix}
L = 10\ Kbits, R = 100\ Mbps \\
one-hop\ transmission\ delay = 0.1\ ms
\end{matrix}
$$
**Queueing**
- Occurs when work arrives faster than it can be serviced
	![[Pasted image 20251017171218.png | 500]]
- Queueing and loss: 
	- if arrival rate (in bps) to link exceeds transmission rate (bps):
		- packets will queue, waiting to be transmitted on output link
		- packets can be dropped (lost) if memory buffer in router fills up

#### Circuit switching
- Circuit switching is an alternative to packet switching 
- end to end, resources allocated to and reseved for "call" between source and destination
- The call gets assigned a "circuit" at each link and this cannot be shared with other calls
- Guaranteed performance, but if the call is idle then it gets left unused
- Used in traditional phone networks

**Frequency Division Multiplexing (FDM)**
- optical/electromagnetic frequencies are divided into narrow frequency bands
- Each call gets allocated its own band and can transmit at the max rate of that narrow band
				**split:** 	======
						======

**Time Division Multiplexing (TDM)**
- time divided into slots
- each call allocated periodic slot(s), cal transmit at maximum rate of this wider frequency band only during its time slots
				**split**: | | | | | | | | 
#### Packet Switching vs. Circuit switching
**Scenario:**  
Host A sends a $640{,}000\ \text{bit}$ file to Host B.  

**Circuit-switched network:**

$$
\begin{matrix}
\text{Setup delay} = 2\ \text{sec}\\

\text{Transmission delay} = \frac{640{,}000\ \text{bits}}{1.5\ \\ \text{Mbps}} \approx 0.427\ \text{sec} \\

\text{Total delay} \approx 2.43\ \text{sec}\\
\end{matrix}
$$

Guaranteed bandwidth, but idle time between bursts.  

**Packet-switched network:**

- No setup delay.  
- If each packet is $1{,}000\ \text{bits}$, sent at $1.5\ \text{Mbps}$:

$$
\text{Per-packet delay} = \frac{1{,}000}{1.5 \times 10^6} \approx 0.00067\ \text{s}
$$  

- Multiple packets can interleave with other users — higher efficiency, but **variable arrival times**.

Packet switching is usually better, especially for "bursty" data. 
- simpler, no call setup, resource sharing

But, excessive congestion is possible which causes packet delay and loss due to buffer overflow. Protocols are needed for reliable data transfer and congestion control.

#### Internet Structure
The structure is a network of networks. It has millions of access ISP's that need to be connected. 
- global ISP's connect these access nets, and each global ISP is connected by an internet exchange point (IXP)
- If one ISP wants to access another quickly (without IXP) **peering links** are used to connect these (red line between ISP C and ISP B)
![[Pasted image 20251017173442.png]]

**Tier 1 commercial ISPs**: large ISP's like Sprint, AT&T, NTT, etc.
**Content provider network:** Private network that bypasses tier-1 or regional ISP's to access the internet (Google, Facebook)

