Load balancers act as a single point of contact for a service and its many instances. Requests are proxied to a cluster of workers, and that is all. The load balancer does very little, so it can handle high loads. 

The goal of load balanceres is to make a cluster of servers look like one superior server. The load balancer provides the same interface as a single server.

**Benefits:**
- Individual servers can be replaced without affecting the overall service
	- Can deploy rolling app updates or deploy in sync
- Proxy can monitor health of servers - periodically send a health check requests - a simple GET API call

**Two Types of local load balancers**:
- **Network Address Translation** **(NAT)**
	- Works at the TCP/IP layer (layer-4 load balancer)
	- forwards packets one-by-one, but remembers which server was assigned to each client
	- Is compatible with any type of service including HTTP
- **Reverse Proxy**
	- Works at the HTTP Layer
		- Called an application-layer load balancer
	- Stores full request/response before forwarding to client
	- e.g. NginX or maybe Squid
	- Can also do SSL Termination, caching, compression


### **NAT Load Balancer**
NAT Load balancers are types of NAT devices that relays the requests to multiple equivalent serveres. They change UP addresses and ports of packets in both directions.
- Maintains IP and port mappings like a traditional NAT
- Simpler and more efficient than reverse proxy because it doesn't need TCP, HTTP, or store full request/responses

### Reverse Proxy (Nginx)
![[Pasted image 20251130153758.png]]
- TLS/SSL Certificates can be stored just on the proxy
	- Internal communication may be unencrypted
- Proxy might also cache responses, but this limits its scalability

**Expensive "hardware" load balancers implement NAT, while reverse proxies are the cheap open source option. Cloud-based LBs can do both.**

| Feature               | NAT                                   | Reverse Proxy                        |
|-----------------------|----------------------------------------|---------------------------------------|
| Routing done by       | IP address / port translation          | HTTP proxy                            |
| Scale                 | ~1–10M requests/s                      | ~100k–1M requests/s                   |
| Services supported    | Any                                    | Only HTTP                             |
| Additional features   | None                                   | SSL termination, caching              |

**Local Load balancer Limitations:**
- Single point of failure - if something is wrong with nginx you can't access any server
- Can only handle ~1M req/sec
- Resides in one data center, so data center is also a single point of failure
- Huge services need more than just local load balancers
- Can clients find a service replica without a bottleneck? 
- This is a service discovery problem


### Global Load Balancers

#### Domain Name Service (DNS)
- DNS allows multiple answers to be given for a query (multiple IP addresses per name) 
- A cached, distributed system 
- There is no limit to scaling by DNS

Along with balancing load, DNS can also connect the user to the **closest** replica of a service
- Clever DNS server examines the IP address of the requester and resolves to the server that it thinks is closes to the client (geolocation) 
- IP address answers are not just different, but customized for the particular client

### Content Delivery Network (CDN)
- Globally distributed web servers that cache responses for local clients
- A CDN is just a distributed caching HTTP reverse proxy that uses DNS to geographically load-balance
- CDN's uses HTTP caching proxies - edge servers are caching proxies that ask origin server if it doesn't have a cached response

**CDN's are add-on services that manage the edge servers (Cloudflare, Akamai, etc.)**

**IP Anycast load balancing is implemented with BGP and allows all traffic to hit the same IP address across the world**

| Feature                               | DNS                                   | IP Anycast                                      |
|---------------------------------------|----------------------------------------|--------------------------------------------------|
| Routing done by                       | Domain Name Service                    | BGP                                              |
| Maximum scale limited by              | Internet itself                        | Internet itself                                  |
| How quickly can changes be made?      | DNS TTL (minutes to hours)             | BGP convergence (~minute)                        |
| Ease of deployment                    | Requires advanced DNS software/config. | Must operate your own Autonomous System (ISP)    |

| Feature                               | Local                                      | Global                                     |
|---------------------------------------|---------------------------------------------|---------------------------------------------|
| Routing done by                       | NAT or HTTP Proxy                           | DNS or BGP                                  |
| Maximum scale limited by              | Speed of one machine                        | Internet itself                              |
| How quickly can changes be made?      | Milliseconds                                | Minutes to hours                             |
| Ease of deployment                    | Simple, using off-the-shelf software/HW     | Requires advanced DNS software/config       |
