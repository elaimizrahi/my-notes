A service is scalable if it can easily handle growth in the number of concurrent users/requests. Metrics are measures of work throughput:
- Requests/queries per second
- Concurrent users
- Monthly active users

This can be difficult though:
- Limited speed per machine
- Coordinating multiple machines can be difficult
- Sharing data among multiple machines is difficult
- User distributed worldwide
- High probability one machine will fail as it scales


**Vertical Scaling**
- Easiest approach is to just buy a faster machine to run your service 

![[Pasted image 20251124124427.png]]

**Factors affecting performance:**
- Number of CPU cores, their speed, lack of competing processes
- Amount of memory
- Type of disk (SSD or Magnetic) 
- Network 
- Presence of GPUs, TPUs, or special accelerators

**Cloud Computing makes scaling easier for us**: 
- **Vertical Scaling**
	- Can change the instance type of a virtual machine 
	- Vertical scaling just requires re-booting the machine
- **Horizontal Scaling**
	- Just need to purchase more VM instances
	- New instance can be used in just a few minutes

## Vertical Scaling — Pros & Cons

| Pros                                                                                                  | Cons                                                                             |
| ----------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| Easy to write your programs.                                                                          | Cannot handle really huge loads.                                                 |
| Most languages support multithreading.                                                                | Cannot be scaled quickly in a fine-grained manner (must replace entire machine). |
| Most “off-the-shelf” software is designed for one machine (MySQL, Oracle DB, Apache, Nginx, Node.js). | Single point of failure.                                                         |
| Modern servers can run many cores (~96).                                                              | Poor price/performance ratio for top-end machines.                               |
| Can connect hundreds of disks before hitting I/O limits.                                              | Motherboards with many sockets are expensive.                                    |
| Avoids slow communication with external machines.                                                     | Fastest CPUs are expensive.                                                      |
**Vertical scaling is *not* scalable!**


**We need horizontal scaling for global apps**:
- Public cloud providers can give you lots of machines, but using them well is tough
- Most of the focus for class will be horizontal scaling


## HTTP and Web Servers
Web servers take **requests** as input and output **responses**. These structure how web servers interact with client computers.

We can convert a program to a service by:
- Listening to requests over the network
- Running many copies concurrently using threads/processes
- Using queues to store unhandled requests and unsent responses

We set up a request and response queue to handle the data that needs to be processed. It can serve these requests to any copy of the app currently running

![[Pasted image 20251124130411.png]]
*e.g. While one thread is waiting for the IO to complete, another can use the CPU*

#### HTTP
- Client-server data exchange protocol
- Invented for web browsers to fetch pages from webservers
- **Request**:
	-  human-readable header with URL, method, plus some optional ones
	- Optional Body storing raw data
- **Response**
	- Human-readable header with response code, plus some optional onse
	- Optional Body

![[Pasted image 20251124131059.png]]

### Recap
- A software service is a program that runs continuously, giving responses to requests.  
- Scalability is the ability of a service to grow to handle many concurrent users (ideally an arbitrarily large number).  
- Two approaches to scaling that are useful in different scenarios: 
	- Vertical scaling is upgrading your machine(s).  
		- The simplest and most efficient way of scaling... but there is a ceiling. 
	- Horizontal scaling is adding more machines. 
	- Coordinating a cluster of machines is complicated, but it's necessary for global scale and massive throughput.
- Showed that web server frameworks let you translate a simple program into a multi-threaded service with concurrency.  
- Introduced HTTP as the most common type of service.  
	- Client requests a document (specified in path/url)  
	- Server sends document in the response.  
	- High-level overview of Wikipedia’s architecture