Microservices come with their own security challenges that are unique to them. Namely, each small service has to manage access and securing communication routes to protect against data theft or unauthorized access. 

Our objective is to protect internal services from public access while minimizing attack surface and preventing direct public access to services.

There's more overhead for each service, and microservices allow attackers to see data transfer better so that they can easily identify targets. 

### API Gateways
API gateways act as an entry point to the server applications and hide some of the routes we don't want the user to see. 
- Reduces attack surface
- Minimizes potential targets for attackers

**Implementation**:
- Global authentication
	- enforces API access tokens for authentication
	- verifies pre-shared secrets for each client
- Differentiation of application Access
	- unique API access token fro each application
	- Helps in limiting API access 
- Security Strategies
	- Implements rate-limiting to prevent DOS attacks
	- Mitigates overload of microservices by external attackers

API Gateways also enforce strict access rules for managing infrastructure, integrates with Active director or similar services, and protect outbound communication.


### Communication Protection
We want to prevent middleman attacks:
- Ensure the client communicates with the intended server
- Distinguishes between malicious and legitimate servers

![[Pasted image 20251201110635.png]]


