There is not currently a well-defined terminology or notation to characterize architectural structures. But, good SWE's make common use of these principles when designing software. Lots of them are rule of thumb or patterns that are informal, but others are more documented as industry standards.


## Repository Style
- Suitable for applications in which the central issue is establishing, augmenting, and maintaining a central boyd of information (github, wikipedia, etc)

**Components:**
- central data structure representing the current system
- Collection of independent components that operate on the central data structure
- Connected with calls or direct memory access

![[Pasted image 20251201114005.png]]

**Advantages:**
- Efficient way to store large amounts of data 
- Sharing model is published as the repository schema
- Centralized management for backup, security, and concurrency control

**Disadvantages:**
- Must agree on a data model beforehand
- difficult to distribute data
- data evolution is expensive

## Pipe and Filter
- Suitable for applications that need a defined series of independed computations to be performed on data
- Component reads streams of data as input and makes streams of data as output

**Components:**
 - Filters - applies local transformation to input streams
 - Pipes - connectors that are conduits for the streams 

![[Pasted image 20251201114311.png]]

- Filters do not share state with other filters
- Filters do not know the identity of the upstream or downstream filters
- e.g. in unix we can do | for pipes and commands as filters `cat file | grep Erroll | wc -l`

**Advantages:**
- Easy to maintain and enhance systems
- Permit specialized analysis (like throughput or deadlock)
- naturally support concurrent execution

**Disadvantages:**
- not good choice for interactive systems because of transformational character
- Excessive parsing and unparsing leads to loss of performance

**Compilers started off this way, but now do more of a repository style architecture**

## Object-Oriented Style
- Useful for applications in which a central issue is identifying and protecting related bodies of information
- Data representations are associated operations are abstracted by ADT's (abstract data types)

**Components:**
- Objects act as the main components
- Connectors are function and procedure invocations

![[Pasted image 20251201114902.png]]
- Objects must preserve the integrity of the data representation
- data representation is hidden from other objects

**Advantages:**
- Possible to change implementation without affecting clients
- Can design systems as collections of autonomous interacting agents

**Disadvantages:**
 - In order fro one object to interact with another, the first must know the identity of the second objects
 - Objects cause side effect problems (A doesn't expect B's changes on C)


## Layered Style
- For applications that involve distinct classes of services that can organized hierarchially
- Each layer provides service to the layer above it and serves as a client to the layer below it

![[Pasted image 20251201115309.png]]

![[Pasted image 20251201115319.png]]

- Often exceptions are made to permit non-adjacent layers to communicate directly for efficiency reasons

**Advantages:**
 - Design is based on increasing level of abstraction
 - Changes to one of the layers affects at most two other layers
 - Different implementations with identical interfaces of the same layer are interchangeable

**Disadvantages**:
- Not all systems are easily structured like this
- Performance might need the coupling of high-level functions to their lower-level implementations

## Interpreter Style
- For applications where the most appropriate language or machine for executing the solution is not available

**Components:**
 - include once state machine for the execution engine and three memories:
	 - Current state of engine
	 - Program being interpreted
	 - Current state of program
- Connectors are procedure calls and direct memory accesses


![[Pasted image 20251201115715.png]]

**Advantages:**
 - Simulation of non-implemented hardware
 - facilitates portability of application or languages across variety of platforms

**Disadvantages:**
 - Defining, implementing, and testing are not trivial
 - Extra level of indirection slows down execution - i.e. JRE

## Process-Control Style
- For applications whose purpose is to maintain specified properties of the outputs of the process sufficiently near the given reference values

**Components:**
- Process definition includes mechanisms for manipulating some process variables
- Control algorithm for deciding how to manipulate process variables
- Connectors are the the data flow relations for:
	- Process variables - control, input, and manipulated variables
	- Set point - the desired value for a controlled variable
	- Sensors - obtain values of process variables pertinent to control

![[Pasted image 20251201120553.png]]

**Open-loop:**
![[Pasted image 20251201120626.png]]

**See [[SE 380]] for more of this stuff - not in a software context though - PID controllers**

## Client-Server Style
- For applications that involve distributed data and processing across a range of components

**Components**
 - Servers - stand-alone components that provide specific services 
 - Clients - components that call on the services provided by the servers
 - Connectors are the network

![[Pasted image 20251201120847.png]]

**Advantages**:
 - Straightforward distribution of dadta
 -  Transparency of location
 - Mix and match platforms
 - Easy to add new servers or upgrade existing ones

**Disadvantages**
 - Performance of the system depends on the performance of the network
 - Tricky to design and implement
 - Need a central register of names and services

## Peer-to-Peer (P2P)
- For applications that partition tasks/workloads between peers without centralized control

**Components:** 
- Peers - standalone components that provide specific services 
- Connectors are some kind of network (i.e. websockets)

![[Pasted image 20251201121138.png]]


**Advantages**
- No single point of failure
- Resilient against network issues
- Scalable
- More privacy
- Easy to add new peers

**Disadvantages**:
 - Cliques of peers could be disconnected 
 - Requires continuously exchanging lists of known peers 
 - Or need a central name server
 - No guaranteed response from peers


Sources:
• [Bass et al 98] Bass, L., Clements, P., Kazman R., Software Architecture in Practice. SEI  
Series in Software Engineering, Addison-Wesley, 1998  
• [CORBA96] Seigel, J. 1996. CORBA Fundamentals and Programming. John Wiley and  
Sons Publishing, New York.  
• [CORBA98] CORBA. 1998. OMG’s CORBA Web Page. In: http://www.corba.org  
• [Gamma95] Gamma, E., Helm, R., Johnson, R., Vlissides, J., 1995. Design Patterns:  
Elements of Reusable Object-Oriented Software. Addison-Wesley Inc., Reading,  
Massachusetts.  
• [IBM98] MQSeries Whitepaper. In:  
http://www.software.ibm.com/ts/mqseries/library/whitepapers/mqover  
• [JavaIDL98] Lewis, G., Barber, S., Seigel, E. 1998. Programming with Java IDL: Developing  
Web Applications with Java and CORBA. Wiley Computer Publishing, New York.  
• [Rozanski05] Rozanski, N., Woods, E. 2005. Software Systems Architecture: Working with  
Stakeholders using Viewpoints and Perspectives, 1st ed. Addison-Wesley.  
• [Shaw96] Shaw, M., Garlan, D. 1996. Software Architecture: Perspectives on an  
Emerging Discipline. Prentice Hall.

