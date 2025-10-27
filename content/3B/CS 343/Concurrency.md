### Key Concepts
- A **thread** is an independent sequence of execution within a program.
- A **process** is a program component with its own thread and memory.
- A **task** also has its own thread but often shares memory (light-weight process).
- **Parallel execution**: true simultaneous execution on multiple CPUs.
- **Concurrent execution**: multiple threads appear to run in parallel via interleaving.  
    → concurrency = threads of control, not processors.

### 5.1 Why Write Concurrent Programs
- Breaks a problem into independently executing threads (like modular routines).
- Some problems are **naturally concurrent** — best described by multiple interacting activities.
- Improves **efficiency** by exploiting both algorithmic and hardware-level parallelism.
    
### 5.2 Why Concurrency Is Difficult
**Challenges:**
- **Understanding:** coordinating many concurrent activities is complex.
- **Specification:** deciding how to partition tasks and when/how they interact.
- **Debugging:** non-determinism (different execution orders each run → Heisenbugs).

**Analogy:**  
Moving furniture — can’t do it alone, but coordinating multiple helpers efficiently is hard:
- How many helpers?
- Where are the bottlenecks (e.g., door, large items)?
- What communication is needed (e.g., fragile items, teamwork)?
    
**Takeaway:**
> Concurrency offers performance and structure benefits, but introduces coordination, communication, and reasoning complexity.

### 5.3 Concurrent Hardware
####  Core idea
- Even on **one CPU**, concurrency is possible via **context switching** — rapid swapping of threads on the same processor.
    - **Uniprocessor:** multitasking for tasks.
    - **Multiprocessor:** multiple CPUs for true parallelism.
#### Task switching
- Parallelism is simulated by switching threads **non-deterministically** between instructions.
- Unlike coroutines (which switch predictably), **task switching** can occur _anywhere_, even between two machine instructions.  
    → introduces unpredictability and concurrency bugs.

**Consequences:**
- Programs must tolerate arbitrary switching order.
- Switching can happen:
    - **Explicitly** – controlled by program logic.
    - **Implicitly** – due to **interrupts** (I/O, timers, page faults).
        
> ⚠️ Task execution is not continuous — time-sliced by interrupts.
###  5.4 Execution States
#### Preemptive vs Non-Preemptive
- **Preemptive scheduling:** interrupts determine when context switches occur → non-deterministic.
- **Non-preemptive scheduling:** switching only at defined points (e.g., end of routine).
- Preemptive context switches occur between **instructions** → much harder to reason about.
    
#### Multi-CPU and Distributed Systems

|Type|Memory|Communication|Pointer Sharing|
|---|---|---|---|
|**Multiprocessor**|Shared|Fast|✅ Works|
|**Distributed System**|Separate|Network-based|❌ Does not work|
- Real systems often mix both: CPUs for main, graphics, bus, network, etc.
    
**Summary:**  
Concurrency relies on context switching — simulated on one CPU, true on multiple.  
Preemptive scheduling and interrupts make execution timing unpredictable, requiring careful synchronization and coordination.

### 5.5 Threading Model
A threading model defines the relationship between threads and CPUs 
- the OS manages CPUs providing logical access via **kernel threads** scheduled across the CPUs

- Relationship is denoted by user:kernel:CPU, where:  
    ◦ 1:1:C (kernel threading) – 1 user thread maps to 1 kernel thread  
    ◦ N:N:C (generalize kernel threading) – N × 1:1 kernel threads (Java/Pthreads/C++) ◦ M:1:C (user threading) – M user threads map to 1 kernel thread (no parallelism) ◦ M:N:C (user threading) – M user threads map to N kernel threads (Go, μC++)
- Often the CPU number (C) is omitted.

![[Pasted image 20251027114951.png]]

> We can also add nano threads on top of user threads, and cirtual machines below the OS


### 5.6 Concurrent Systems
Concurrent Systems can be divided into 3 types:
1. those that attempt to discover implicit concurrency in an otherwise sequential program
	- fundamental limit to how much concurrency can be found
2. those that provide concurrency through implicit constructs, which a programmer uses to build a concurrent program
	- concurrency accessed indirectly w/ special mechanisms
3. those that provide concurrency through explicit constructs
	- concurrency accessed directly and threads explicitly managed
**Types 1 and 2 come from type 3, and to solve all concurrency problems, threads need to be explicit**


### 5.7 Speedup
Program **speedup** is $S_C = \frac{T_1}{T_C}$ where $C$ is number of CPUs and $T_1$ is sequential execution

![[Pasted image 20251027115534.png]]

Speedup is affected by:
1. amount of concurrency
2. critical path of efficiency
3. scheduler efficiency

**Amdahl's Law:** concurrent section is P $\implies$ sequential section $1 - P$ , maximum speedup using $C$ CPUs is:

$$
S_C = \frac{1}{(1-P) + \frac{P}{C}}
$$
where $T_1 = 1$, $T_C$ = sequential + concurrent

**Concurrent programming consists of minimizing the sequential section (1 − P)**
- Concurrent stages performs scatter/gather pattern; scatter data/computation, gather results.
- While sequential sections bound speedup, concurrent sections bound speedup by the **critical path of computation.**

- **independent execution:** all threads created together and do not interact.
- **dependent execution:** threads created at different times and interact.


### 5.8 Thread Creation
Concurrency requires 3 mechanisms in a programming language
1. creation 
2. synchronization
3. communication

#### 5.8.1 COBEGIN/COEND
- Compound statement with statements run by multiple threads

![[Pasted image 20251027120656.png]]
> Each **BEGIN/END** is a new thread, so the code in those sections can run separately, and will be gathered in **COEND**

We can use thread graphs to represent thread creations:

![[Pasted image 20251027120817.png | 500]]

> We can also make dynamic number of threads using recursion
```c
void loop(int N){
	if (N != 0){
		COBEGIN
			BEGIN p1(...) END
			BEGIN loop(N-1) END
		COEND
	}
}
cin >> N;
loop(N);
```

#### 5.8.1 START/WAIT
Start thread in the routine and wait(join) at thread termination, allowing arbitrary thread graph:

![[Pasted image 20251027121042.png]]
> START(p, 5) starts method $p$ with input argument $5$
- This allows the same routine to be started multiple times with different arguments

#### 5.8.3 Actor
An **Actor** is a unit of work **without** a thread, like BEGIN/END

![[Pasted image 20251027121531.png]]

- an executor thread matches. an actor with a message and runs the actor's behaviour like COBEGIN/COEND
- Usually no shared information among actors and no blocking is allowed

```c
#include <uActor.h>
    
struct StrMsg : public uActor::Message { // derived message  
	string val; // string message  
	StrMsg( string val ) : Message( uActor::Delete ), // delete after use
};

_Actor Hello { // : public uActor  
	Allocation receive( Message & msg ) { // receive base type
		iftype( StrMsg, msg ) {
			...msg->val
		} eliftype ( StopMsg, msg ) return Delete; // delete actor
		endiftype // MUST HAVE
		return Nodelete; // reuse actor
	}
};

int main() { 
	uActor::start(); //start actor system
	*new Hello() | *new StrMsg( "hello" ) | uActor::stopMsg; 
	*new Hello() | *new StrMsg( "bonjour" ) | uActor::stopMsg;
	uActor::stop(); // wait for all actors to terminate
}


```