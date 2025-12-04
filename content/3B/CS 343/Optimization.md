- A computer with infinite memory and speed requires no optimizations to use less memory or run faster - but with finite resource4, optimization is necessary for performance
- General forms of optimizations are:
	- **reordering**: Data and code are reordered to increase performance in certain contexts
	- **eliding** removal of unnecessary data, data accesses, and computation
	- **replication**: processors, memory, data, code and duplicated because of limitations in processing and communication speed
- Optimized program must be isomorphic to the original
- Kinds of optimizations are restricted by the kind of environment

### 10.1 Sequential Optimizations
- Most programs are sequential - even concurrent programs are
	- large sections of sequential code per thread connected by small sections of concurrent code where threads interact
- **Sequential** execution presents simple semantics for optimization - operations occur in program order
- Dependencies result in partial ordering among a set of statements (precedence graph)

![[Pasted image 20251203171527.png]]
![[Pasted image 20251203171535.png]]
![[Pasted image 20251203171549.png]]

- Very complex reordering, reducing , and overlapping of operations are allowed
- Overlapping implies micro-parallelism, **but limited capability in sequential execution**

### 10.2 Memory Hierarchy
- Complex memory hierarchy: 

![[Pasted image 20251203171737.png]]
- Optimizing data flow along this hierarchy defines a computer's speed
- Hardware aggressively optimizes data flow for sequential execution
- Having basic understanding of cache is essential to understanding performance of both sequential and concurrent programs

#### 10.2.1 Cache Review
- Problem: CPI is 100x faster than memory and 100,000x times faster than disk
- Solution, copy data from general memory into very fast local registers
- But we only have a few registers for billions of bytes of memory
- But we quickly run out of registers and must rotate the data

**Solution:** Use hardware caches to stage data without pushing to memory and allow sharing of data among the programs

![[Pasted image 20251203172034.png]]

- Caching transparently hides the latency of accessing main memory
- Cache loads in 64/128/256 bytes called cache line, with addresses multiple of line size

- In theory, cache can eliminate registers, but registers provide small addressable area (register window) with short addresses - allowing for shorter instructions

#### 10.2.2 Cache Coherence
- Multi-level caches used, each larger but with diminishing speed and cost
- e.g. having an L1, L2, and L3 cache

- Data reads logically percolate variables from memory up the memory hierarchy, making cache copies to registers
- Data writes from registers to variables logically percolate down the memory hierarchy through cache copies to memory
- Unlike registers, all cache values are shared across the computer, which means variables can replicated in a large number of locations

- Cache coherence is a hardware protocol ensuring this duplicated data is fully updated -> we occasionally propagate up the memory hierarchy 
- Cache consistency addresses **when** the processor sees the update, and then does bidirectional synchronization
- ![[Pasted image 20251203173027.png]]
	- Writes eventually appear in largely the same order as writter
	- Critical section works as writes to shared variable appear before write to lock release
	- Otherwise, spin until write appears
- If threads continually read/write to the same memory locations, they invalidate duplicate cache lines, resulting in excessive cache updates - this is called **cache thrashing**
	- The updated value bounces from one cache to the next
	- because cache line contains multiple variables, cache thrashing can occur inadvertently, called **false sharing** 
- This false sharing would happen when two threads share the same cache line and send different variables across them


### 10.3 Concurrent Optimizations
- In sequential execution, **strong memory ordering**: Reading always returns last value written
- In concurrent execution, **weak memory ordering:** reading can return previously written value or value written in future
	- Notion of **current** value becomes blurred for shared variables unless everyone can see values assigned simultaneously
- SME control order and speed of execution, otherwise non-determinism causes random results or failure
- Sequential section accessing private variables can be optimized normally **but not across concurrent boundaries**
- If concurrent sections are accessing shared variables they can be corrupted by sequential optimizations so restrict optimizations to ensure correctness
- For correctness and performance, identify concurrent code and only restrict **its** optimization

#### 10.3.1 Disjoint Reordering
- $R_x \rightarrow R_y$ allows $R_y \rightarrow R_x$ 
	- Reordering Disjoint reads has no issues
- $W_x \rightarrow R_y$ allows $R_y \rightarrow W_x$ 
	- In dekker entry protocol both threads read dontwantin, both set wantin, both see dontwantin, and proceed
- $R_x \rightarrow W_y$ allows $W_y \rightarrow R_x$ 
	- In synchronization flags, allows interchanging lines 1 and 3 for Consumer
- $W_x \rightarrow W_y$ allows $W_y \rightarrow W_x$ 
	- In synchronization flags, allows interchanging lines 1 and 2 in Producer and lines 3 and 4 in consumer
	- In Peterson's entry protocol, allows interchanging lines 1 and 2
- Compiler uses all of these reorderings to break mutual exclusion
	- Moves lock entry/exit after/before critical section because entry/exit variables not used in it
	- e.g. **Double-check locking** for concurrent singleton pattern

#### 10.3.2 Eliding
- For high-level language, compiler decides when/which variables are loaded into registers and for how long
- Elide reads (loads) by copying (replicating) value into a register
```c
		Task1.                     Task2
		 ...                       register = flag 
		flag = false (write)       while(register)
```
- Hence, variable logically disappears for duration in register
	- So task spins forever in busy loop if R before W
- Also, elide meaningless sequential code
	- sleep(1) is unnecessary in a sequential program

#### 10.3.3 Replication
- Why is there a benefit to reorder R/W?
- Modern processors increase performance by executing multiple instruction in parallel on **replicated hardware**
	- internal pool of instructions take from program order
	- begin simultaneous execution of instruction pool as inputs
	- collect results from finished instructions
	- instructions with independent inputs execute out-of-order
- From sequential perspective, disjoint reordering is unimportant
- For concurrency, it's very important

### 10.4 Memory Model
- Manufacturers define set of optimizations performed implicitly by processor 
- Set of optimizations indirectly define a memory model:

![[Pasted image 20251203190621.png]]

- AT has events occur instantaneously ⇒ slow or impossible (distributed). 
- SC accepts all events cannot occur instantaneously ⇒ may read old values 
- SC still strong enough for software mutual-exclusion (Dekker 5.18.6 / Peterson 5.18.7). 
- SC often considered minimum model for concurrency. 
- No hardware supports just AT/SC. 
- TSO (x86/SPARC), PSO, WO (ARM, Alpha), RC (PowerPC) + atomic R/W synchronizations

### 10.5 Preventing Optimization Problems
- All optimization problems result from races on shared variables
- If shared data is protected by locks (implicit or explicit)
	- Locks define the sequential concurrent boundaries
	- Boundaries must preclude optimizations that affect concurrency
- Called race free as synchronization and mutual exclusion preclude races
- But, race free does have races. 
- Races are internal to locks, which lock programmer must deal with
- Two approaches:
	- ad hoc: programmer manually augments all data races with pragmas to restrict optimizations 
	- Formal: Language has memory model and mechanisms to abstractly define races in program: portable but oftern suboptimal
- Data access/compiler: `volatile qualifier`
	- Force variable loads and stores to/from registers (at sequence points)
	- created for longjmp or force access for memory-mapped devices
	- for architectures with few registers, practically all variables are implicitly volatile
	- Java Volatile and C++ atomic are stronger
- Program order/compiler (static): disable inlining, `asm("...")`
- memory order / runtime (dynamic): sfence, lfence, mfence (x86)
- Locks built to ensure sequential consistency for protected shared variables
	- **no user races and strong locks means that we have a sequentially consistent memory model**




