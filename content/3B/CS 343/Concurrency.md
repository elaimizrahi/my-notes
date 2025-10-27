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

#### 5.8.4 Thread Object
We allocate thread objects using `_Task`
```c
_Task T {
	void main(){...} // thread starts here
}
//COBEGIN 
{
	T t; // create object on stack, start thread
}//COEND

//START
T *t = new T; //creates thread object on heap + starts
//WAIT
delete t;
```
We can use this to simulate COBEGIN/COEND and START/WAIT

![[Pasted image 20251027135644.png | 500]]
![[Pasted image 20251027135657.png | 500]]

- $i$ cannot be assigned until tf is deleted, otherwise the value could change in s2/s3

### 5.9 Termination Synchronization
A thread terminates when:
- it finishes normally
- it is killed by parent/sibling (not in `uC++`)
- finishes with an error
- because the parent terminates (not in `uC++`)

This my be used to perform a final communicates between threads, and is possible for independent threads

### 5.10 Divide-and-conquer
Divide and conquer is characterized by splitting work across data, then doing the work independently on the data
- The work on each data group is identical to the work performed on the data as a whole
- Termination synchronization is required to know when the work is done (gather)

This can be done effectively with `COFOR`

**COFOR** lets us create a dynamic number of coroutines
```c
#include <uCobegin.h>
int main(){
	...
	COFOR(r, 0, rows,
		subtotals[r] = ...
		this is the coroutine main
	)
}
will wait for all threads to complete down here
```
This can also be done using Actors:
```c
_Actor Adder {  
    int * row, cols, & subtotal; // communication
     Allocation receive( Message & ) {
    ...
    } public:
    
    Adder( int row[], int cols, int & subtotal ) : row( row ), cols( cols ), subtotal( subtotal ) {} };
    
int main(){
	...//same
	uActor::start();
	for(int r = 0; r < rows; r += 1){
		*new Adder(matrix[r] cols, subtotals[r]) | uActor::startMsg;
	}
	uActor::stop();
}
```

This can also be done with tasks, where it's the same setup as the actor, but you need a second for loop in the main function to delete the Adder Tasks

### 5.11 Exceptions
- Exceptions can be handled locally in a task, nonlocally among coroutines, or concurrently among tasks
	- all concurrent exceptions are nonlocal, but nonlocal exceptions can also be sequential
- Unhandled exceptions go to coroutine's last resumer, or a tasks' joiner
- These are possible because each coroutine has its own stack
- Nonlocal exceptions between tasks and coroutines are the same as between coroutines

![[Pasted image 20251027141029.png]]

After an event is delivered, the faulting execution is not interrupted, it polls:
- when an `_Enable` statement begins/ends
- after call to suspend/resume
- after call to yeild
- after call to wait
- after a call to `_Accept` unblocks for `RendezvousFailure`

### 5.12 Synchronization and Communication During Execution
Synchronization occurs when one thread waits until another thread has reached a certain execution point (state and code)
- This is needed when transmitting data between threads
- One thread has to be ready to transmit the information and the other has to be ready to receive, at the same time
- Otherwise one might transmit when no one is receiving or vice-versa

![[Pasted image 20251027141419.png]]

- This loop waiting for an event among threads is called a busy wait.

### 5.13 Communication
- Once threads are synced, there are many ways to transfer information between them
	- If they share memory, can transfer by value or address
	- If not (distributed), then transferring information by value is straightforward but address is difficult

### 5.14 Critical Section
- Threads may access non-concurrent objects like a file or linked-list
- There is an issue if multiple threads are acting on the same object at once though
- So we make the operation **Atomic** to indicate it must be done synchronously

**Critical Sections** are groups of atomic operations on data
**Mutual Exclusion** is the preventing of simultaneous execution of a critical section by multiple threads

- **We need to minimize the amount of mutual exclusion (i.e., make critical sections as small as possible, Amdahl’s law) to maximize concurrency**

### 5.15 Static Variables
Note: static variables in a class are shared among all objects generated by that class
- These variables need mutual exclusion to be done right
- This can be ok in a few special cases, like in a task constructor (if tasks are generated serially)

Correct usage:
```c
T::T(int tid){...}

T* t[10]
for(int i = 0; i < 10; i++){
	t[i] = new T(i);
}

OR

uArray(T, t, 10);
for(int i = 0; i < 10; i++){
	t[i](i)
}
```
> In general, we want to avoid using shared static variables in concurrent programs

### 5.16 Mutual Exclusion
Is it possible to write code guaranteeing a statement (or group of them) are always serially executed by 2 tasks?

- Rules :
    1. Only one thread can be in a critical section at a time with respect to a particular object (safety).
    2. Threads may run at arbitrary speed and in arbitrary order, while the underlying system guarantees a thread makes progress (i.e., threads get some CPU time).
    3. If a thread is not in the entry or exit code controlling access to the critical section, it may not prevent other threads from entering the critical section.
    4. In selecting a thread for entry to a critical section, a selection cannot be postponed indefinitely (liveness). Not satisfying this rule is called **indefinite postponement** or **livelock**.
    5. After a thread starts entry to the critical section, it must eventually enter. Not satisfying this rule is called starvation.

Notes:
- • If threads are not serviced in first-come first-serve (FCFS) order of arrival, there is a notion of unfairness
-  Indefinite postponement and starvation are related by busy waiting.
- Unfairness implies waiting threads are overtaken by arriving threads, called barging.
### 5.17 Self-Testing Critical Section
```c
void CriticalSection(){
	static uBaseTask*  curr;
	curr = &uThisTask();
	for(int i = 0; i < 100; i ++){
		...//work
		if (curr != &uThisTask()){ //check
			abort("interference");
		}
	}
	
}
```

### 5.18 Software Solutions
##### 5.18.1-5 Invalid examples
1. Regular Lock
2. Alternation
3. Declare Intent
4. Retract Intent
5. Prioritized Retract Intent

#### 5.18.6 Dekker's Algorithm
First **complete software solution** to **mutual exclusion** that satisfies all 5 rules (safety, progress, bounded waiting, etc.).

**Key Idea**:
Combine **intent flags** (`WantIn`, `DontWantIn`) with a shared **Last** variable to fairly alternate entry between two threads.

**Algorithm Overview**:
1. Each thread sets its **intent** (`WantIn`) before entering.
2. If both threads want in → the one that was **last** (per `Last`) **retracts** its intent and waits.
3. After exiting the critical section, a thread sets `Last` to itself.

This ensures **fair turn-taking** and avoids starvation.

![[Pasted image 20251027145221.png]]

To avoid Hesselink failure, add conjunction `&& you == WantIn` on line 6 and the conditional on line 8

- Dekker's algorithm has **unbounded overtaking** (not starvation) because race loser retracts intent

#### 5.18.7 Peterson’s Algorithm (Modified Declare Intent)

Simpler and more efficient successor to **Dekker’s Algorithm**, solving **mutual exclusion** for 2 threads with **fewer reads/writes** and **bounded overtaking**.

**Core Idea:**
Each thread announces **intent** (`WantIn`) and updates a shared variable `Last` to indicate **which thread gets priority** next.

#### Algorithm (simplified)
```c
me = WantIn;       // declare intent
::Last = &me;      // give priority to other thread (RACE!)
while (you == WantIn && ::Last == &me) {}  // wait if other wants in
CriticalSection();
me = DontWantIn;   // retract intent

```

- ❌ **RW-unsafe** on real hardware — requires **atomic read/write**.
	- Works only if reads/writes don’t “flicker” (non-atomic behaviour can break it).
	- In theory (atomic memory model), it’s fully correct.
- has **Bounded Overtaking** since the race loser does not retract intent

#### 5.18.8 N-Thread Prioritized Entry
Extend Peterson/Dekker logic to **N threads**, where each thread gets a **priority number** (0 = highest).

Algorithm:
1. Each thread announces its **intent** (`WantIn`).
2. Two-step entry protocol:
    - **Step 1:** Wait for all _higher-priority_ threads to finish.
    - **Step 2:** Wait for all _lower-priority_ threads that are currently inside.
3. Enter **critical section**, then clear intent.
```c
intents[priority] = WantIn;     // declare intent
// wait for higher priorities
for (int i = 0; i < priority; i++)
    while (intents[i] == WantIn) {}

// wait for lower priorities
for (int i = priority+1; i < N; i++)
    while (intents[i] == WantIn) {}

```
- **Breaks Rule 5 (Bounded Waiting):**  
    High-priority threads can **starve** low-priority ones indefinitely.
- ❌ Not fully **fair** → lower threads may never enter.
- 🧠 RW-safe versions exist but need more bits:
    - 3-bit version = RW-unsafe
    - 4-bit version = RW-safe
- No known solution for all 5 rules using only N bits.

##### 5.18.9 N-Thread Bakery (Tickets)

- Generalization of Lamport’s Bakery algorithm for N threads.
- Each thread “takes a number” to ensure fair, ordered entry into the critical section.
- Lowest ticket number has highest priority; thread ID breaks ties.
- Resembles a physical ticketing system: everyone waits their turn.
- Threads reset their ticket when done, releasing the spot.
    
```c
_Task Bakery {
    int *ticket, N, priority;

    void main() {
        for (int i = 1; i <= 1000; i++) {
            // Step 1: Select a ticket
            ticket[priority] = 0;             // mark as selecting
            int max = 0;
            for (int j = 0; j < N; j++) {     // find largest ticket
                int v = ticket[j];
                if (v != INT_MAX && max < v) max = v;
            }
            ticket[priority] = max + 1;       // assign next ticket

            // Step 2: Wait for turn
            for (int j = 0; j < N; j++) {
                while (ticket[j] < ticket[priority] ||
                       (ticket[j] == ticket[priority] && j < priority)) {}
            }

            CriticalSection();

            // Exit protocol
            ticket[priority] = INT_MAX;       // release ticket
        }
    }
};
```

##### 5.18.10 Tournament Algorithm

- Organizes threads in a **binary tree**.
- Each **node** = small 2-thread mutual exclusion (Dekker/Peterson).
- Threads “compete” up the tree to the **root** (winner enters CS).
- Exit protocol **retracts intents in reverse order**.
- Each node ensures progress → no starvation.
- Combines **fairness** and **performance**.
- **Unbounded overtaking** possible since nodes unsynchronized

```c
_Task TournamentMax {
    struct Token { int intents[2], turn; } *t;
    int depth, id;

    void main() {
        int lid = id;
        // entry protocol
        for (int lv = 0; lv < depth; lv++) {
            binary_prologue(lid & 1, &t[lv][lid >> 1]);
            lid >>= 1;
        }
        CriticalSection();
        // exit protocol (reverse order)
        for (int lv = depth - 1; lv >= 0; lv--) {
            lid <<= 1;
            binary_epilogue(lid & 1, &t[lv][lid >> 1]);
        }
    }
};
```

- Scales well for many threads (log N levels)  
- No starvation (each node ensures progress)  
-  Parallel competition per tree level
- Unbounded overtaking → possible unfairness

##### 5.18.11 Arbiter Algorithm
- Dedicated **arbiter thread** controls access to CS.
- Clients signal intent → arbiter sequentially grants access.
- No thread runs the entry logic; arbiter handles synchronization.
- Prevents indefinite postponement, ensures **no starvation**.
- Uses busy waiting → high CPU cost.


```c
bool intents[N], serving[N];  // all false

_Task Client {
    int me;
    void main() {
        for (int i = 0; i < 100; i++) {
            intents[me] = true;               // entry protocol
            while (!serving[me]) {}           // wait for arbiter
            CriticalSection();
            intents[me] = false;              // exit protocol
        }
    }
};

_Task Arbiter {
    void main() {
        int i = N;
        for (;;) {
            do { i = (i + 1) % N; }           // circular search
            while (!intents[i]);              // skip inactive
            intents[i] = false;               // grant access
            serving[i] = true;
            while (serving[i]) {}             // wait for exit
        }
    }
};
```


### 5.19 Hardware Solutions
- Hardware solutions introduce level below the software level 
- Special instructions to perform an atomic read and write operation
- sufficient for multitasking on a single CPU

##### 5.19.1 Test/Set Instruction
We can use the `TestSet(Lock)` instruction to create locks
- if test/set returns open then loop stops and lock is set to closed
- if test/set returns closed then loop executes until other thread sets lock to open

This works for N threads attempting to enter a critical section and only needs one shared datum
- No guarantee of eventual progress though


##### 5.19.2 Swap Instruction
`Swap` interchanges a Dummy variable with a Lock, so that the thread holds the lock and gives the dummy until it's done
```c
int dummy = CLOSED
do{
	Swap(Lock, dummy)
} while (dummy == CLOSED);
lock = OPEN
```

##### 5.19.3 Fetch and Increment Instruction
Performs an increment between the read and write with `FetchInc`
```c
main(){
	while(FetchInc(Lock) != 0);
	critical section
	Lock = 0;
}
```

Once done with its critical section it resets lock to 0 for next thread to pass through
Overflow is only a problem if all values used simultaneously and FIFO service



