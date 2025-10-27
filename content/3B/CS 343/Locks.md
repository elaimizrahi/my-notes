Locks are how we package software and hardware locking into an abstract type for general use. They are used for synchronization or mutual exclusion - sometimes both.

### 6.1 Lock Taxonomy
- We have two types of locks, spinning and blocking

![[Pasted image 20251027153002.png]]

**Spinning Lock**: Busy wait until an event occurs, task oscillates between ready and running states

**Blocking Locks:** do not busy wait, but block until an even occurs - some other mechanism must unblock waiting task when the event happens

### 6.2 Spin Lock
A spin lock is implemented using busy waiting, which loops checking for an event to occur
```c
while(TestSet( Lock ) == CLOSED) // use up time-slice (no yeild)
```
Instead of looping until critical section is unlocked or an event happens, we can:
- explicitly terminate time-slice
- move back to ready state after only **one** event-check fails
```c
while(TestSet( Lock ) == CLOSED) uThisTask().yeild(); // relinquish time slice
```
**Adaptive Spin-Locks** allow this number to be addaptable, so we can check any number of times
> Most spin locks break the starvation rule, it is possible for 1+ tasks

#### 6.2.1 Implementation
In `uC++`, we can have a non-yielding spin lock `uSpinLock` and a yielding spin lock `uLock`

![[Pasted image 20251027153618.png]]
![[Pasted image 20251027153602.png]]

### 6.3 Blocking Locks
For blocking locks, acquiring task makes **one** check for open lock and blocks. The releasing task has the sole responsibility for detecting blocked acquirer and transferring lock, or just releasing the lock.

- Blocking locks reduce busy waiting by having releasing task do additional work: **cooperation**
- all blocking locks have 
	- state to facilitate lock semantics 
	- list of blocked acquirers

![[Pasted image 20251027154648.png]]

##### 6.3.1 Mutex Lock
A mutex lock is only used to provide mutual exclusion
- separates lock usage between sync and mutex
- permits optimization and checks as the lock on has one special function

Two kinds:
1. **single acquisition**: task that acquired the lock cannot acquire it again
2. **multiple acquisition**: lock owner can acquire it multiple times, called an **owner lock**
> Multiple acquisition locks can handle looping or recursion with the lock in the critical section

- A **mutex** controls ownership of a shared resource (`avail`, `owner`, `blocked` queue).
- Uses a **spin lock** internally for atomicity.
- Busy-wait avoided by yielding (`yieldNoSchedule`) instead of rescheduling.
```c
class MutexLock {
  bool avail = true;
  Task* owner = nullptr;
  queue<Task*> blocked;
  SpinLock lock;

public:
  void acquire() {
    lock.acquire();                        // grab spin lock
    while (!avail && owner != currThread()) {
      yieldNoSchedule();                   // yield, not reschedule
      lock.acquire();                      // reacquire spin lock
    }
    avail = false;
    owner = currThread();
    lock.release();
  }

  void release() {
    lock.acquire();
    owner = nullptr;
    if (!blocked.empty()) /* wake next */ ;
    else avail = true;
    lock.release();
  }
};

```
##### 6.3.1.2 uOwnerLock
`uOwnerLock` is a multiple-acquisition mutex lock that `uC++` provides
```c
class uOwnerLock {
public:
  uOwnerLock();
  uBaseTask* owner();        // nullptr if none
  unsigned int times();      // acquisition count
  void acquire();
  bool tryacquire();
  void release();
};
```

##### 6.3.1.3 Mutex-Lock Release Pattern
To ensure a mutual exclusion lock is always released, use the following patterns:

1. Executable statement - finally clause:
```c
uOwnerLock lock;
lock.acquire();
try{
	...
} _Finally{
	lock.release();
}
```

2. Allocation/deallocation (RAII - resource acquisition is initialization)
```c
class RAII{
	uOwnerLock &lock;
	public:
		RAII(uOwnerLock & lock): lock(lock) {lock.acquire()}
		~RAII() {lock.release();}
};
uOwnerLock lock; 
{
	RAII raii(lock); // lock acquired by constructor
	... // protected by lock
}// lock released by destructor
```

##### 6.3.1.4 Stream Locks
Specialized mutex lock for I/O based on `uOwnerLock` (`osacquire`, `isacquire` in µC++)
- Prevents interleaving output between threads.
```c
task1: osacquire(cout) << "abc " << "def" << endl;
task2: osacquire(cout) << "uvw " << "xyz" << endl;
```
> This could have `abc uvw xyz` as part of the output, for example

```c
{ // acquire the lock for stream cout for block duration 
osacquire acq( cout ); // named stream lock  
cout << "abc";  
osacquire( cout ) << "uvw " << "xyz " << endl; // OK? 
cout << "def";
} // implicitly release the lock when “acq” is deallocated
```

### 6.3.2 Synchronization Lock
Synchronization locks are used solely to block tasks waiting for synchronization
- This is the weakest form of blocking lock as it is only a list of blocked tasks
- **Acquiring task always blocks**
- **release is lost when no waiting task**
This is often called a **condition lock**, with wait/signal for acquire/release

- Requires mutual exclusion for safe implemenation
- Location of mutual exclusion classifies synchronization lock:
	- **external locking** use an external lock to protect task list,
	- **internal locking** use an internal lock to protect state (lock is extra state).
```c
class SyncLock{
	Task* list;
	public:
		SyncLock(): list(nullptr);
		void acquire() {
			//add self to task list
			yieldNosSchedule();
		}
		void release() {
			if (list != nullptr){
				remove task from blocked list and make ready
			}
		}
	}
```
- Needs external lock to protect access to `list` (external locking).
- Alternative: **internal locking** – use an internal spinlock.
- Avoid “lost wakeups” by ensuring external state is visible before yield.

![[Pasted image 20251027163244.png]]
- Preemption can cause race between blocking/unblocking.
- Solution: **release external lock before blocking**, reacquire later.
- Use cooperation with scheduler (`yieldNoSchedule(lock)`).
```c
void acquire() {
    m.acquire();                 // external mutex
    if (!occupied) s.acquire();  // block until event
    m.release();
}
```

##### 6.3.2.2 uCondLock
- µC++ internal synchronization lock, paired with a **uOwnerLock**.
- Implements **condition-variable semantics** (wait/signal/broadcast).
- Wait atomically releases the given mutex and reacquires it on wake-up.
```c
class uCondLock {
public:
    void wait(uOwnerLock &lock);      // release lock + block
    void signal();                    // wake one
    void broadcast();                 // wake all
    bool empty();                     // true if no waiters
};
```

Example:
```c
uOwnerLock mlk;
uCondLock clk;
bool done = false;


_Task T1 {
    void main() {
        mlk.acquire();
        while (!done)
            clk.wait(mlk);       // releases mlk + blocks
        mlk.release();
    }
};

_Task T2 {
    void main() {
        mlk.acquire();
        done = true;
        clk.signal();            // wake waiting T1
        mlk.release();
    }
};
```

- Prevents “lost signal” by guarding shared state (`done`) with mutual exclusion.
- Synchronization locks are **weaker** and require external mutual exclusion.

### 6.3.3 Barrier
A **barrier** coordinates a group of tasks performing a concurrent operation surrounded by sequential operation 
 - A barrier is for (gather) synchronization and cannot build mutual exclusion

Two kinds of barrier: 
1. Threads are equal to the group size $(T == G)$
2. Threads greater than group size $(T > G)$

> Unlike other locks, the barrier retains state about the events it manages - number of tasks blocked on the barrier
- Since manipulating this state needs mutual exclusion, most barriers internally lock their state

![[Pasted image 20251027164243.png | 400]]
- Barrier initialized to 3 -> will block all tasks until all 3 tasks have called block
- Last task to call block does not block and releases all other tasks (cooperation), making all tasks leave together (sync)

![[Pasted image 20251027164418.png]]

*Note*: Incorrectly managing the barrier cycles is called the **reinitialization problem**

##### 6.3.3.1 Fetch Increment Barrier
- spinning, T == G flag ensures waiting threads exit barrier even if fast threads change count

![[Pasted image 20251027164619.png]]

> Here we need to construct failure scenario for `await(b.count == 0)`

##### 6.3.2.2 uBarrier
`uC++` barrier is sa blocking, $T > G$, barging prevention coroutine, where the coroutine main can be resumed by the last task arriving at the barrier

![[Pasted image 20251027164832.png]]

- Member last is called by the Gth (last) task to the barrier, and then all blocked are released
- `uBarrier` has implicit mutual exclusion $\implies$ no barging $\implies$ only manages synchronization

###### Example – Summation with `uBarrier`
```c
_Coroutine Accumulator : public uBarrier {
    uBaseTask *Gth_;               // remember Gth (last) task
    int total_ = 0;
	protected:
	    void last() { total_ = 0; }    // reset after last arrives
	public:
	    Accumulator(int rows) : uBarrier(rows) {}
	    void block(int subtotal) {
	        total_ += subtotal;
	        uBarrier::block();         // synchronize with others
	    }
	    int total() { return total_; }
};

```

Here, we store the total value in the accumulator itself to avoid race conditions.

**Example - Adder Tasks**:
```c
_Task Adder {
    int row_;
    Accumulator &acc;
    void main() {
        int subtotal = 0;
        for (...) subtotal += ...;  // compute subtotal
        acc.block(subtotal);        // block at barrier, synchronize
    }
public:
    Adder(int row, Accumulator &acc) : row_(row), acc(acc) {}
};

```
Usage in main:
```c
int main() {
    const int rows = 10;
    Accumulator acc(rows);
    for (int r = 0; r < rows; r += 1)
        new Adder(r, acc);          // each Adder computes a row sum
    acc.block();                    // main task waits at barrier
    cout << acc.total() << endl;
}

```
