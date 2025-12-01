## Implicit Invocation Style
- Suitable for applications that have loosely-coupled colection of components
- Even better for applications that must be reconfigured on the fly (change service provider, enabling/didsabling capabilities)

Instead of invoking a procedure directly, a component can announce one or more events. Other components in the systems can register interest in the even by associating a procedure with the event. When even is announced, the broadcasting system (connector) invokes all procedures that have been registered for the event.

![[Pasted image 20251201130829.png]]

- Announcers of the events don't know which components will be affected
- Components cannot assume the order of processing or what processing will occur from their event
- Often the connectors include the traditional procedure call in addition to the bindings of between even announcements and procedure calls


## Event Driven Architecture
- An architecture style that orchestrates behaviour around events
- Event producers and consumers, with events as significant state changes
- Event-Centric - focuses on production, detection, and consumption
- Async communication 
- Producers and consumers are decoupled

This is used for real-time data processing, distributed systems, and complex event processing

![[Pasted image 20251201131215.png]]

- Benefits
	- Scalable, flexible, responsive, resilient, and maintainable
	- Very reactive and modular
- Challenges
	- Ensuring event reliability
	- Complexity in event handling
	- Event Granularity
	- Security Concerns

## Publisher-Subscriber (Pub-Sub)
- Enables applications to announce events asynchronously
- Avoids coupling between senders and receivers

![[Pasted image 20251201131408.png]]

- Challenges
	- Need components to provide information to others as events occur when doing this in distributed systems
	- Scaling challenges - innefficiency with dedicated message queues for each consumer

**Async Messaging Subsystem**
- Key Components
	- Input messaging channel for sender (publisher)
	- Output messaging channel per consumer (subscriber)
	- Intermediary for message distribution (message broker)
- Message is a packet of data and even is a message indicating a change/action
- Advantages
	- Decoupling
	- Better scalability and sender responsiveness
	- Reliability
	- Deferred or scheduled processing

**Message Distribution:**
- Handling message subsets
	- Use of tops for dedicated channels
	- Content filtering
	- Wildcard subscribers for multiple topics
- Bi-Directional Communication
	- User request/reply pattern for acknowledgements or status communication

**Handling Poison Messages**
- Dealing with poison messages
	- prevent return to queue, store separate for analysis
	- Use dead-letter queue in message brokers
- Duplicate message handling
	- Implement duplicate detection and removal and ensure idempotency

**Message Expiration + Scheduling**
- Implement limited lifetime for messages and expiration time
- Enable embargo on messages until specific dat and time
- Restrict receiver access until specified time

- useful for broad information broadcasting, not real-time
- Applicable for system with eventual data consistency models

**Example: ROS (Robotics Operating System)**
- framework for writing robotic software 

![[Pasted image 20251201132244.png]]

- Additional Components:  
	- Services: Synchronous remote procedure call  
	- Actions: Aync communication interfaces

- ROS Master
	- Gives naming and registration, tracks pubs and subs, central communication point
- ROS Nodes
	- Single-purpose executables
- Communication
	- Topics
		- Channels for message passing between nodes
		- pub/sub messaging mechanism
	- Messages 
		- typed data structure
		- Defined w/ simple language (ROS msg)
- Services:
	- Request/response between nodes
- Actions
	- goal-oriented communication for longer running tasks
- Why pub-sub?
	- Decoupling of components
	- Flexibility
	- Ease of integration
