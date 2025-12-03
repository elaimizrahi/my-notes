- P and V are low level primitives for protecting critical sections and establishing synchronization between tasks
- Shared variables provide the actual information that is communicated
- Both of these can be complicated to use 
- Split-binary semaphores and baton passing are complex
- Need high level facilities that perform some of these details automatically

## 8.1 Critical Regions
- Declare which variables are to be shared, as in `VAR v : Shared integer MutexLock v_lock`

### 8.3 Monitor 
- A monitor is an abstract data type that combines shared dxata with serialization of its modification 
```c
_Monitor name{
	shared data
	members that shee and share the data
}
```
- A mutex member is one that does not begin execution if there is another active mutex member
	- A call to a mutex member may become blocked waiting entry, and queues of waiting tasks may form
	- Public member routines of a monitor are implicitly mutex and other kinds of members can be made explicitly mutex with a qualifier `_Mutex`
- Recursive entry is allowed (owner mutex lock), i.e one mutex member can call another or itself
- Unhandled exceptions raised within a monitor should always release the implicit monitor locks so the monitor can continue to function
- Destructor must be mutex, so ending a block with a monitor or deleting a dynamically allocated monitor, blocks if there is a thread still in the monitor
- Atomic counter using monitor:

```c
_Monitor AtomicCounter {
    int counter;                    // shared data
  public:
    AtomicCounter(int init = 0) : counter(init) {}
    int inc() {                    // mutex member
        counter += 1;
        return counter;
    }
    int dec() {                    // mutex member
        counter -= 1;
        return counter;
    }
};

```

### 8.4 Scheduling
- A monitor may want to schedule tasks in an order different from the order in which they arrive (bounded buffer, readers/writes with stale/freshness)
- There are two techniques: 
	- **External**  is scheduling tasks outside the monitor and is accomplished with the accept statement
	- **Internal** is scheduling tasks inside the monitor and is accomplished using condition variables with `signal` and `wait`

#### 8.4.1 External Scheduling
- The accept statement controls which mutex members can accept calls
- By preventing certain members from accepting calls at different times, it's possible to control the scheduling of tasks
- **Each `_Accept` defines what cooperation must occur for the accepting task to proceed**

e.g bounded buffer:
![[Pasted image 20251202191845.png]]

- Queues of tasks form outside the monitor, waiting to be accepted into either insert or remove
- An acceptor blocks all calls **except a call to the specified mutex members** (uses barging prevention)
- Accepted call is executed like a conventional member call
- When the accepted task does an accept, it blocks, forming a stack of blocked acceptors
- Alternative calls that satisfy accepter's requirement are possible
```c
_Accept(insert || remove) // one of insert or remove
```
- `_NoMutex` member implies that if there's multiple threads in the monitor then it cannot be accepted
- External scheduling is simple because unblocking (signalling) is implicit

#### 8.4.2 Internal Scheduling
- Scheduling among tasks inside the monitor
- A **condition** is an external synchronization-lock, i.e. a queue of waiting tasks:
```c
uCondition x, y, z[5]
```
- `empty` returns false if there are tasks blocked on the queue and true otherwise
- `front` returns an integer value stored with the waiting task at the front of the condition queue 
- A task waits (blocks) by placing itself on a condition `x.wait() // wait(mutex, condition)`
	- Atomically block at the end of a condition queue, and releasing the monitor lock so tasks can enter monitor
- A task on a condition queue is made ready by signalling the condition `x.signal()`
	- Removes and makes ready blocked task at from of the condition queue
- Signalled tasks scheduled before calling tasks means no barging, baton passing
- **The Signaller does not block, so the signalled task must continue waiting until the signaller exits or waits**

Example: BoundedBuffer:
![[Pasted image 20251203145625.png]]

- `wait` blocks the current thread, adn restarts a signalled task or implicitly releases the monitor lock
- `signal()` unblocks the thread on the front of the condition queue after the signaller thread blocks or exits
- `signalBlock()` unblocks the thread on the front of the condition queue and blocks the signaller thread

![[Pasted image 20251203145940.png]]

**Note: We can't use external scheduling if:**
- the scheduling depends on member parameter values, e.g. compatibility code for dating
- Scheduling must block in the monitor but cannot guarantee the next call fulfills cooperation

**Solving Readers/Writer with external scheduling**:
```c
_Monitor ReadersWriter {
	int rcnt = 0, wcnt = 0;
	public:
	void endRead() {
		rcnt -=1
	}
	void endWrite(){
		wcnt = 0
	}
	void startRead() {
		if (wcnt > 0) _Accept(endWrite);
		rcnt += 1
	}
	void startWrite() {
	if (wcnt > 0) _Accept(endWrite)
	else while (rcnt > 0)_Accept(endRead)
	wcnt = 1
	}
}
```

### 8.6 Exceptions 
- An exception raised in a monitor member propagates to the callers' thread

```c
_Monitor M {
	public:
	void mem1(){
		if..._Throw E()
	}
	void mem2(){
		try{
			...if..._Accept(mem1)
		} catch (uMutexFailure::RendezvousFailure &){
			//deal w/ rendezvous failure
		}
	}
}
```
- On exiting `mem1`, caller implicitly raises non-local RendezvousFailure Exception at monitor acceptors' thread to identify failed cooperation
- RendezvousFailure is always enabled, so `_Enable` block is unnecessary
- For multiple `_Accept(clause 1 || clause 2)` a flag variable is needed to know which member failed

### 8.7 Nested Monitor Calls
- Nested monitor problem: acquire monitor M1, call to monitor M2, and wait on condition in M2 (M2 is a member of M1)
- Monitor M2's mutex lock is released by wait, but monitor M1's monitor lock is not released which leads to potential deadlock
- Releasing all lock can inadvertently release a lock -- e.g. incorrectly release M0 before M1
- Same problem occurs with locks - called the **lock composition problem**
```c
_Monitor RW {
    _Monitor RWN {
        uCondition bench;
        int rcnt = 0;
    public:
        void startRead() { rcnt += 1; }
        void endRead() {
            rcnt -= 1;
            if ( rcnt == 0 ) bench.signal();
        }
        void startEndWrite() {
            if ( rcnt > 0 ) bench.wait();   // WAIT inside nested monitor!
            // writer does sequential write
        }
    } rwn;

    _Mutex void mutexRead() { rwn.startRead(); }
public:
    void write() {
        rwn.startEndWrite();
    }
    _Nomutex void read() {
        mutexRead();
        // concurrent reading occurs here
        rwn.endRead();
    }
};
```
- If the writer waits in `rwn`, it prevents both readers and writers acquiring `rw`, which prevents starvation and forces FIFO ordering

### 8.8 Intrusive Lists
- Non-contiguous variable length data-structures (like dicts) normally require dynamic allocation as the structure increases/decreases when adding/deleting nodes
- 3 kinds of collection node: 
	- data/implicit links
	- pointer/implicit links
	- data/explicit links

![[Pasted image 20251203152541.png]]
- Like a linked list where the list pointers live inside the data itself
```c 
struct Node : public uColable {
    int value;
};
```
- Here the `uColable` is the next pointer (and possibly prev) - inside the struct
- The `uSeqable` gives us two link fields (prev and next)
- This avoid dynamic allocation while still giving us dynamic sized structures
- Most notably, we can using this for keeping track of queues (blocked/waiting) and keep it on the heap:
```c
if ( . . . ) { 
	Node n{ . . . } // allocate on thread stack 
	queue.add( n ); // chain stack node onto list 
	// block 
	queue.drop(); // node n must be at head/tail of list 
} // automatically free n
```

### 8.9 Counting Semaphore, V, P vs. Condition, Signal, and Wait
- There are several important differences between these:
	- P only blocks if `semaphore = 0`, `wait` always blocks
	- V before P affects the P, while signal before wait is lost (no state)
	- multiple Vs may start multiple tasks simultaneously, while multiple signals only start one task at a time because each task must exit serially through the monitor

**Note**: it's possible to simulate P and V using a monitor
```c
_Monitor semaphore { 
	int sem; 
	uCondition semcond; 
	public: 
	semaphore( int cnt = 1 ) : sem( cnt ) {} 
	void P() { 
		if ( sem == 0 ) semcond.wait(); 
		sem -= 1; 
	} 
	void V() { 
		sem += 1; 
		semcond.signal(); 
	} 
};
```


### 8.10 Monitor Types
- **Explicit Scheduling** occurs when: 
	- An accept statement blocks the active task on the acceptor stack and makes a task ready from the specified mutex member queue 
	- A signal moves a task from the specified condition to the signalled stack
- **Implicit Scheduling** occurs when:
	- A task waits in or exits from a mutex member, and a new task is selected first from the A/S stack, then the entry queue
- Monitors are classified by the implicit scheduling (who gets control) of the monitor when a task waits or signals or exits
- Implicit scheduling can select from the calling(C), signalled(W) and signaller(S) queues.

![[Pasted image 20251203155022.png]]
![[Pasted image 20251203155033.png]]
- We really only care about the ones where C is prioritized, as the other have uncontrolled barging or starvation

- **implicit Signal**
	- Monitors either have an explicit signal (statement) or an implicit signal (automatic signal) 
	- The implicit signal monitor has no condition variables or explicit signal statement
	- Instead, there is a waitUntil statement: `waitUntil logical-expression`
	- The implicit signal causes a task to wait until the conditional expression is true
- Additional restricted monitor-type requiring the signaller exit immediately from monitor is called an **immediate-return signal**
	- Not powerful enough to handle all cases, e.g. dating service, but optimizes the most common case of signal before return
- Remaining Monitor types:
![[Pasted image 20251203155906.png]]

- no-priority blocking requires the signaller task to recheck the waiting condition in case of a barging task - use a while loop around a signal
- no-priority non-blocking requires the signalled task to recheck the waiting condition in case of a barging task - use a while loop around a wait
- implicit (automatic) signal is good for prototyping but has poor performance
- priority nonblocking has no barging and optimizes signal before return (supply cooperation)
- Priority blocking has no barging and handles internal cooperation within the monitor (wait for cooperation)

- Coroutine Monitor (`_Cormonitor` / `_Mutex _Coroutine`)
	- Coroutine with implicit mutual exclusion on calls to specified member routines
	- can use resume, suspend, condition variables (wait, signal, signalBlock, `_Accept`)

### 8.11 Java Monitor
- Java has synchronized class members (i.e. `_Mutex` members but incorrectly named) and a synchonized statement (`synchronized` = monitor mutual exclusion (like `_Mutex`, but poorly named))
- All classes have one implicit condition variable and `wait, notify, and notifyAll()` to manipulate it
- Internal scheduling is no-priority nonblocking, which causes **barging**
	- wait statements must be in while loops to recheck conditions
- Java’s `java.util.concurrent` conditions are **incompatible** with language conditions.
- Bounded buffer limitations:
	- Only **one condition queue**, so producers/consumers block together → must use `notifyAll()`, not `notify()`.
	- Having only one condition makes some patterns impossible or inefficient.
- Problem in naïve barrier implementation:
	- `notifyAll()` wakes everyone
	- Nth thread exits monitor and **barges back in** before others wake → sees full count → starts next phase early.
- You **cannot implement condition variables using nested monitors** because `wait()` releases only _that object's_ lock, causing deadlock.












