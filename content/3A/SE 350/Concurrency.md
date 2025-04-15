Communication and resource sharing in the presence of multiple concurrent threads of execution
- **Single Processor:** Multiprogramming (interleaving)
- **SMP**: Multiprocessing

We must note:
- Relative order and speed of execution threads/processes cannot be controlled
- **Race Conditions** may arise, the result of the computation is unexpectedly dependent on the sequence of operations

**Our goal is to avoid race conditions!!!**

**Example for race condition:**
```c
void echo(){
	char ch;
	ch = getchar();
	putchar(ch);
}

// Consider the possible order of execution with two threads T1 and T2:
t1: ch = getchar()
.
t2: ch = getchar()
.
t1: putchar(ch)
.
t2: putchar(ch)

// What happens if ch is a global variable?
```

**Solution**: 
- **Mutual Exclusion**
	- Only one process/thread can access a resource at a time
- This is implemented through **Critical Sections**
	- portion of code where only one process/thread can access at a time, otherwise thread is blocked and waiting to run the critical section
	- We use `entercritical()` and `exitcritical()` to denote critical sections

**Competition:** independent processes compete for access to resources
**Cooperation with shared memory**: processes/threads communicate through shared data in memory
- more common with threads since same virtual address space
**Cooperation with message passing:** processes/threads communicate by sending messages
- more common with processes, allows distributed computation

##### Requirements for concurrency
1. Enforce **Mutual Exclusion**
2. Avoid **Deadlock**
3. Avoid **Starvation**

### Mutual exclusion
Only one process at a time is allowed in the **crtical section** for a resource
- if a process halts in its noncritical section, must not interfere with others
- No deadlock or starvation

- **Software Solution** 
	- No need to memorize, covered in tutorial
- **Hardware Support**
	- Low overhead, but forces processes to busy-wait
- **OS Support** 
	- Best solution - waiting processes can be blocked
	- Can use semaphores, monitors (not needed)

**Interrupt Disabling** guarantees mutual solution in the software
```c
while(true){
 \* disable interrupts *\
 \* critical sections *\
 \* enable interrupts *\
 ...
}
```
**We  can also use atomic operations**

**Special machine instructions also ensure mutual exclusion (ie. compare + swap)** 
- Applicable to any number of processes
- simple and easy to verify
- Busy-waiting takes processor time
- deadlock + starvation possible


### Semaphores
OS-maintained data structure for signaling across processes

- A variable that has an integer value, initialized to a non-negative number
- **semWait** operation decrements the value. If the result is negative, the process is blocked
- **semSignal** operation increments. If the result is 0 or negative, a blocked process is unblocked

**Mutual exclusion with Semaphores:** 
```c
Semaphore s = 1;
void P(int i ){
	while(true){
		semWait(s);
		/* critical section */
		semSignal(s);
		/* remainder*/
	}
}
...
void main(){
 P(1)
}
```


**Avoiding starvation**
- FIFO queue avoids starvation
- **Strong Semaphores** specifies order that processes are removed from queue
- **Weak Semaphores** No guarantee on the order

![[Pasted image 20250408124633.png]]

##### Binary Semaphores
 - Semaphores only set to 1 or 0
##### Mutexes
- Variation of Semaphores specialized for mutual exclusion only
- Binary Semaphore initialized to 1
- semWait/semSignal renamed to lock/unlock
- only the process that locked can unlock

### Producer/Consumer problem
One or more producers generating + placing data in a buffer with a single consumer taking items out. Only one producer or consumer may act at a time

- **infinite** queue** is a queue where producers append and consumers take items out of it
- **finite queue** implements circular buffer
	- We add a modulus to the append/pop index so that the producers/consumers loop around the queue


### Message Passing
User processes can call **send(destination, message)** or **receive(source, message)** to synchronize 

1. **Blocking send, blocking receive** - both sender + receiver are blocked until message sent (rendezvous)
2. **Nonblocking send, blocking receive** - sender continues on, receiver blocked until message arrives
3. **Nonblocking send, nonblocking receive** - neither sender + receiver have to wait

**Direct addressing** - send primitive uses identifier of receiver
- **explicit addressing** receive primitive uses identifier of sender process
- **implicit addressing** receive accepts any message, returns id of sender

**Indirect addressing** - create a shared queue (port/mailbox), messages are sent/received from port
- can have one->one, many->one for **port** one->many, many->many for **mailbox**

**Messages have headers** containing type, destination id, source id, etc. with message content stored underneath


### Examples

#### Unix 
- **Pipes** are most common mechanism between processes
- **Circular Buffer** producer/consumer mechanism
- OS ensure mutual exclusion
- Uses messages, shared memory, semaphores, and signals

#### Linux
- Atomic operations are used
- Spinlocks
	- protect kernel content
	- basic reader/writer types
- Uses semaphores and barriers to prevent processor from reordering loads/stores

#### Reader/Writers problem
Any number of readers can read, but only one writer may use the resource at a time. If a writer is writing to the resource, no reader may read it

**Solutions**
1. Readers have priority
	-  Lightswitch pattern (lock/unlock on each semaphore)
2. No starvation
	- uses a turnstile solution to wait followed by signal on the same semaphore
	- Allows one process(writer) to block multiple other processes of the same type (reader) 
3. Writers have priority
	- Uses turnstile implementation
	- Checks if we have noReaders and noWriters (which are also semaphores)

