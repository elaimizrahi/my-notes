Services are black boxes, exposing network APIs. 
- Decouples development from different parts of the system
- Network APIs define the format and meaning of the requests and responses

REST is the most popular format for network APIs
- Based on HTTP

e.g. Wikipedia is one monolithic architecture, with Squid caching proxies and MariaDB databases

**Monolithic design**
- Advantages
	- Easy to build, deploy, test, coordinate, and share data
	- Best choice for simple apps, some large ones like facebook use it too
- Disadvantages
	- Creates bottlenecks in software development process
	- Leads to messy, fragile code
	- While app must be redeployed for small new functionality
	- Must choose one programming language, build system, etc.

**Microservices**
- Many smaller services work together to achieve the same goal that could be implemented in a monolithic design
- Each microservice is a black box to the rest of the system. It's an independent service
- Implementation details are hidden from the rest of the system

- Disadvantages
	- More difficult to trace through request handling coe for debugging
	- ACID transactions are more difficult to support
	- Developers get silo-ed or isolated in sub-projects

These service APIs can help us also support multiple desktop and mobile apps with cloud-based data and services. As long as they have a common format of requests and responses (REST, Thrift, Protobuf, GraphQL, etc.)

![[Pasted image 20251130145643.png]]


