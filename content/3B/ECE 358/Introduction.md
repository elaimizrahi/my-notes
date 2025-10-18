#### Important terms

**Network edge:** hosts, access network, physical media
**Network Core:** packet/circuit switching, internet structure

**Performance:** loss, delay, throughput
**Packet switches:** forward packets (chunks of data)
**Communication Links:** fiber, copper, ethernet
**Networks:** collection of devices, routers, links
**Protocols:** control sending, receiving of messages

**Internet Standards:** RFC (request for comments) IETF (Internet Engineering Task Force)
**Infrastructure:** provides services to applications + programming interface to distributed applications
**Human protocols:** "what's the time", "I have a question" 
- network protocols act the same way
- define the format + order of messages sent and received among network entities

**Network Edge:** hosts: clients and servers (servers are in data centers)
**Access networks:** wired, wireless, communication links
**Network Core:** interconnected routers, network of networks
- We connect end systems to edge routers with access nets (residential, institutional, mobile)
- ![[Pasted image 20251017161504.png|200]]
**Host:** sends packets of data
- takes application message, breaks into chunks (packets) of length $L$ bits
- Transmits packet into access network at link transmission rate $R$ (aka bandwidth)
$$
\text{packet transmission delay} = \frac{L\ (\text{bits})}{R\ (\text{bits/sec})}
$$

##### Links
**bit:** propagates between transmitted/receiver pairs
**guided media:** signals propagate in solid media (wires)
**unguided media**: same as guided but free, like radio

**Types of guided media**

| Twisted Pair (TP)             | Coaxial Cable                              | Fiber Optic                                                               |
| ----------------------------- | ------------------------------------------ | ------------------------------------------------------------------------- |
| two insulated copper wires    | two concentric copper conductors           | glass fiber with light pulses                                             |
| Cat5: 100Mbps, 1Gbps Ethernet | bidirectional                              | high speed transmission: 10's-100's Gbps                                  |
| Cat6: 10Gbps Ethernet         | broadband: 100'sMbps per channel per cable | low error rate since repeaters spaced far apart, immune to electric noise |
**Types of unguided media**

| Wireless radio                                | Radio link types                                           |
| --------------------------------------------- | ---------------------------------------------------------- |
| signal carried in "bands"                     | Wireless LAN: 100Mbps, 10m                                 |
| no physical wire                              | wide-area: 10Mbps, 10Km                                    |
| broadcast, "half-duplex" (sender to receiver) | Bluetooth: cable replacement, short distance               |
| effected by reflection, obstruction, noise    | terrestrial microwave: point-to-point, 45Mbps channels     |
|                                               | satellite: up to 45 Mbps per channel, 270ms end->end delay |

