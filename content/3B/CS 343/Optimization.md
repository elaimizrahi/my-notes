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
