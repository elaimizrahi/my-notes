While monitors work well for passive objects that need mutual exclusion, communication among tasks with a monitor is indirect

We solve this issue by sending a **message** between objects.
- Tasks can communicate directly by calling each others member routines.

![[Pasted image 20251111140412.png | 400]]
*The bottom diagram is direct communication*

### 9.1 Tasks
- Like a coroutine because it has a distinguished member (main) with its own execution state
- unique because it has thread of control
- Also like a monitor because it provides mutual exclusion and synchronization
	- Only one thread is active in the objects
- Public members are implicitly mutex and other kinds of members can be made explicitly mutex
- External scheduling allows direct calls to mutex members 
- Without external scheduling, tasks must **call out** to communicate to a third party or somehow emulate external scheduling 

![[Pasted image 20251111140934.png | 500]]
*S/ME = synchronization/mutual exclusion*

- In general, basic execution properties produce difference abstractions:
	- When thread or stack is missing it comes from calling object
	- Abstractions are not ad-hoc, rather derived from basic properties
	- Each abstraction has a particular set of problems it can solve, and therefore has a place in a programming language


### 9.2 Scheduling
- A task may want to schedule access to itself by other tasks in an order different from the order that requests arrive
	- As for monitors, there are two techniques: external and internal scheduling

#### 9.2.1 External Scheduling
- As for a monitor, the accept statement can be used to control which mutex members of a task can accept calls

![[Pasted image 20251111141755.png | 500]]

**But this can also be written as:**
```c
_Accept( m1 | | m2 ) S1 ≡ _Accept( m1 ) S1; or _Accept( m2 ) S1; // S2 
if ( C1 | | C2 ) S1 ≡ if ( C1 ) S1; else if ( C2 ) S1; // S2
```

- The extended version allows different `_When` clauses after call for each accept
	- The `_When` clause is like the condition of conditional critical region
		- The condition must be true **and** a call to the specified member must exist before a member is accepted
- If all the accepts are conditional and false, the statement does nothing (like `switch` with no `case` )
- If some conditionals are true, but there are no outstanding calls, the acceptor is blocked until a call to an appropriate member is made
- If several members are accepted and outstanding calls exist to them, a call is selected based on the order of `_Accepts`
	- Hence, the order of `_Accepts` indicates relative selection priority, which avoids starvation
- It's necessary to ensure that for every true conditional, only the corresponding members are accepted -> $2^N - 1$ **if** statements needed to simulate N Accept clauses
- The acceptor is pushed on top of the A/S stack and normal implicit scheduling occurs (C < W < S)

![[Pasted image 20251203162829.png]]
- Once an accepted call completes or a caller wait()s, the statement after the accepting `_Accept` clause is executed and the accept statement is complement
- If there is a terminating `_Else` clause and no `_Accept` can be executed immediately, the terminating `_Else` clause is executed 
- Hence, the terminating else clause allows a conditional attempt to accept a call without the acceptor blocking (tryacquire) 

#### 9.2.2 Internal Scheduling
- Scheduling among tasks inside the monitor 
- As for monitors, condition, signal, and wait are used

![[Pasted image 20251203164019.png]]
- This requires a combination of internal and external scheduling
- **Rendezvous is logically pending when wait restarts Accept task, but post Accept statement still executed (No RendezvousFailure)**

Better code:
![[Pasted image 20251203164550.png]]

![[Pasted image 20251203164556.png]]

#### 9.2.3 Accepting the Destructor
- A common way to terminate a task is to have a stop member
```c
_Task BoundedBuffer{
	private:
	...
	void stop(){}
	public:
	void main(){
		for (;;){
			_Accept(stop){
				break
			}
			or _When ... 
		}
	}
}		
```
- Note - if termination and deallocation follow one another, then add an acceptor for the destructor
	- When the call to the destructor happens, the caller blocks immediately if there is a thread active in the task
	- When it's accepted, the caller is blocked and pushed onto the A/S Stack *instead of the acceptor*
	- Therefore, control restarts at the accept statement without executing the destructor member
- **The Task now behaves like amonitor because its thread is halted**
- Only when the caller to the destructor is popped off the A/S stack by the implicit scheduling is the destructor executed

### 9.3 Increasing Concurrency
- 2 Tasks involved in direct communication (client/caller and server/callee)
- Possible to increase concurrency on both the client and server side

#### 9.3.1 Server SIde
- Server manages a resource and server thread should introduce additional concurrency
![[Pasted image 20251203165249.png]]
- No concurrency on the left as the server is blocked while the client does the work
- Alternatively, the client blocks in member, server does work, and server unblocks client

#### 9.3.1.1 Internal Buffer
- The 

#### 9.3.1.1 Internal Buffer

- Previous technique provides buffering of size 1 between client and server.
- Idea: use a larger internal buffer so clients can drop off work faster.
- However, several issues arise:
	- If production and consumption rates differ (even slightly), the buffer is almost always full or empty.
	- Because a task is mutex-protected, while the server is working no calls can occur → clients cannot drop off arguments.
	- Server could periodically accept while working, but this is awkward.
	- If clients need replies, buffering only helps if out-of-order processing is useful.

- Only way to free the server’s thread to:
	- receive new requests **and**
	- return finished results  
  is to **add another thread**.

- Pattern:
	- Client = customer
	- Server = manager (accepts calls, manages work)
	- Worker(s) = employees doing the actual processing
	- Number of workers must match workload (bounded-buffer problem).

#### 9.3.1.2 Administrator

- An administrator is a server that manages multiple clients and multiple worker tasks.
- Key idea: administrator does **almost no real work**; it only coordinates.
- Administrator:
	- delegates work to workers
	- records and checks completed work
	- passes results back to clients

- Administrator is always accepting calls (it must not block itself).
- Administrator must **never call another task**, because a call may block.
- Administrator maintains a queue/list of work.
- Types of workers:
	- timer → prompts admin periodically
	- notifier → waits for external events (e.g., keyboard)
	- simple worker → does work, returns result to admin
	- complex worker → works and also interacts with a client
	- courier → worker that performs blocking calls on behalf of the admin

![[Pasted image 20251203170153.png]]
![[Pasted image 20251203170208.png]]
### 9.3.2 Client Side

- Sometimes client does not need to wait for server to finish (producer/consumer).
- Achieved via an **asynchronous call**, where client continues immediately.

- μC++ only provides **synchronous** calls.
- Asynchronous systems can be built on top of synchronous ones (and vice versa).

#### 9.3.2.1 Returning Values

- If client just drops off data, async call is simple.
- Hard part: returning a result asynchronously.
- Requires splitting call into:
	- callee.start(args)
	- ... client does work ...
	- result = callee.wait()

- Not the same as START/WAIT (those share a server thread).
- A protocol is needed to match client calls with returned results.


#### 9.3.2.2 Tickets (Protocol)

- First call sends arguments and returns a ticket.
- Second call uses the ticket to retrieve result.
- Problems:
	- client might never pick up result
	- client might reuse or forge tickets
	- protocol is error prone

#### 9.3.2.3 Call-Back Routine

- Client registers a callback routine.
- When result is ready, server calls callback with the result.
- Callback must not block; it should just store result and signal client.
- Client must poll or wait on the indicator.
- Advantage: server "pushes" result to client faster.

#### 9.3.2.4 Futures (Important)

- A future provides async behavior **without explicit protocols**.
- A single call returns an empty future immediately:
	- future = server.work(args)
	- client continues executing
	- later, future is filled when result is ready
	- accessing future blocks if not ready

- Key points:
	- Future is subtype of result type.
	- Read is blocking if empty.
	- After first retrieval, value is cached and cheap to access again.
	- Value is read-only.

- Two μC++ types:
	- Future_ESM\<T>: explicit allocation/free
	- Future_ISM\<T>: automatic lifetime management (recommended)

- Client operations:
	- f = server.work(args)
	- f() retrieves read-only result (blocks if needed)
	- f.available()
	- f.reset()
	- f.cancel()

- Server operations:
	- Create work item
	- Return future to caller
	- Worker/server computes result
	- future.delivery(result) fills it and unblocks waiting clients
	- delivery(exception) stores exception instead
### Select Statement (Futures)

- _Select waits for one or more futures to become available.
- Can combine futures using || and &&.
- Action executes once per satisfied clause.
- _Else makes statement non-blocking.
- Allows complex synchronization (e.g., “wait for 3 futures or 1 stop").

