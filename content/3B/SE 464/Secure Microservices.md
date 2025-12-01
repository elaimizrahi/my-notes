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

- Use of HTTPS/TLS (Transport Layer Security)
	- Encrypts and authenticates communication channels
	- Ensures confidentiality and authenticity of data in transit
- TLS Protocol Mechanics
	- Involves a handshake between communicating parties 
	- Server presents a certificate containing a public key

**TLS Certificate Process**:
 - Certificate:
	 - Has public key enclosed in a private envelope
	 - Signature by issuing authority using a private key
- Importance of private key security 
	- leakage of private keys compromises certificate authenticity
- Client's certificate verification strategies
	- 1. Accept all certificates
	- 2. Accept self-signed certificates
	- 3. Accept certificates signed by trusted authorities
	- 4. Accept only a specific certificate (certificate pinning)

**Standard TLS:** Only the client verifies the server certificate
**Mutual TLS (mTLS)**: Both client and server present and verify certificates. Uses similar verification strategies as standard TLS

#### Internal security Within Clusters
- Want to avoid different services talking directly to one another
- Need to limit the damage that can be done in scenario when intruder breaks in

**Namespaces**: In Kubernetes, we use namespaces to group clusters into smaller, manageable units. Each namespace acts like a distinct segment.
 - Network policies are also used to regulate traffic between namespaces - they define rules for inbound and outbound traffic

**Outbound Communication Policies:**
- Outbound traffic control prevents misuse of services for attacks like SSRF
- Protects against installation of malicious software in pods
- If neglected, unauthorized software can be installed and can be exploited

**Monitoring Clusters:**
- Behaviour monitoring frameworks like Falco define expected behaviour via configuration files
- Security alerts are done by frameworks which notify operators on deviation from normal behaviour - they enhance detection of suspicious activities within the cluster

- Resource quotas in Kubernetes allow us to set resource limits and prevents DOS attacks by limiting resource consumption and are essential to maintain availability

#### Internal Security Within Individual Microservice
- Performed with JSON Web tokens (JWTs)
	- Consists of a header, body, and signature
	- Header specifies signing algorithm
	- Body contains user claims and token validity
	- Signature ensure token integrity and authenticity

![[Pasted image 20251201111925.png]]

**Lifecycle Management:**
- Token validity and renewal
	- Balancing token lifespace for security and usability
	- Use of refresh tokens for longer validiity
- token issuance process
	- User credentials verified by auth service
	- Successful validation leads to JWT issuance
- Token Management Strategy
	- issuance of both access and refresh tokens
	- Revoking and renewing token upon expiration
- Minimizing risks of token theft
	- Automatic invalidation of all user tokens if one is compromised

**Options for Token Management in Microservices:**
- Option A:
	- Use external provider for validation
- Option B: 
	- Centralized token generation, individual services validate tokens
	- Asymmetric crypography with secret key only known to the auth service
![[Pasted image 20251201112312.png]]

**We also want to minimize open ports, as each open port increases attack surface**
- Access control lists can also be implemented to select which visitors are accepted
- Also want to sanitize errors and exceptions to ensure that it doesn't reveal anything about the system (ie. SQL database)
- Can also do Docker Container Hardening
	- Restricting ports, avoiding running as root user, make file system of containers read-only
- can also do input data validation - prevents things like SQL Injection

**Middleware**:
- Middleware acts as a way to cross-cut security concerns
- Acts as an extension of the backend service
- Order of middlewares: Exception -> Authentication -> Authorization -> Access Control

**Risks**:
- SQL Injection
- Remote Code Execution (RCE)
- Cross-site scripting (XSS)

**Validation Techniques:**  
- Validate integer ranges  
- Validate string usage contextually  
- Ensure data transfer object deserialization is secure  
**Context-Specific String Validation:**  
- Database queries: Prevent SQL injection via OR mappers or prepared statements  
- Webpages: Escape strings to mitigate XSS  
- URLs: Validate against allow list, sanitize for SSRF prevention  
- Email services: Verify email format and recipient validity


