### 11.1 Atomic (Lock-free) Data Structure
- Lock free data structures have operations, which are critical sections, but performed without ownership
	- e.g. add/remove node without any blocking duration
- Lock-free is still locking -> spin for conceptual lock -> busy-waiting (starvation)
- If guarantees eventual progress, it's **wait-free**

#### 11.1.1 Compare and Set Instruction
- The compare-and-set instruction performs an atomic compare and conditional assignment

![[Pasted image 20251203193553.png]]
- if compare/assign returns true then loop stops and lock is set to closed
- if compare/assign returns false then loop executes until the other thread sets lock to open
- Alternative implementation assigns comparison value with the value when not equal 
```c
bool CASV(int& val, int& comp, int nval){
	//begin atomic
	if (val == comp){
		val = nval 
		reutrn true
	}
	comp = val
	reutrn false
}
```
- Assignment when unequal useful to restart operations with new changed value

#### 11.1.2 Lock-free stack
- e.g. build a stack with lock-free push and pop operations

![[Pasted image 20251203194013.png]]
![[Pasted image 20251203194422.png]]

#### 11.1.3 ABA Problem
- Pathological failure for series of pops and pushes, called ABA problem
- Given stack with 3 nodes:
	- top $\rightarrow A \rightarrow B \rightarrow C$
- Popping task, $T_i$ sets t to A and dereferenced t->next to get next node B for argument to CAS
- $T_i$ is now time-sliced before the CAS, and while blocked, nodes A and B are popped, and A is pushed again: 
	- top $\rightarrow A \rightarrow C$ // B is gone
- When $T_i$ restarts, CAS successfully removes A as same header before time-slice
- **but not incorrectly sets top to its next node B**
	- top -> B -> ???
- stack is now corrupted!!!

#### 11.1.4 Hardware Fix
- Probabilistic solution for stack exists using double-wide casvd instruction, which compares and assigns 64/128 bit values for 32/64 bit architectures
- Compare pointer _and a counter_.
- Counter increments on push → CAS fails if node was removed & reinserted.

### 11.2 Safe Memory Reclamation
- Problem:
	- Lock-free structures dereference nodes that may be **freed by another thread**, causing segmentation faults.

- Solution: **hazard pointers**:
	- Each thread records which nodes it might dereference.
	- Removed nodes go on a private retire list.
	- Only freed when _no thread_ has them in hazard pointers.

Without SMR, lock-free ≠ safe.

### **11.3 Exotic Atomic Instructions**

##### **LL/SC (Load-Linked / Store-Conditional)**

Better than CAS:
- `LL`: read & reserve a memory location.
- `SC`: write only if reservation still valid.
    
Avoids ABA because SC fails on _any_ interference, not specific-value mismatch.
##### Hardware Transactional Memory (HTM)
- Like mini-database transactions.
- Speculatively executes a block and commits if no conflicts.
- Very fast but limited in size and hardware availability.

### **11.4 GPGPU Basics (SIMD / SIMT)**

##### **SIMT (GPU architecture)**
- Thousands of lightweight threads in **warps** (lockstep execution).
- Branch divergence (threads take different branches) slows execution.
##### **Programming model**

- CPU launches kernels on GPU blocks → blocks contain warps → warps contain threads.
Performance principles:
- Avoid branching.
- Align memory for coalesced access.
- Move heavy computation to device; minimize host-device transfers (PCIe bottleneck).

### 11.5 Concurrency Languages

---

### **11.5.1 Ada**
- Protected types = monitors with **automatic signal**.
- Syntax resembles monitors with guarded entries:

```
protected type Buffer is
    entry insert(elem : in ElemType) when count < Size;
    entry remove(elem : out ElemType) when count > 0;
private
    count : Integer := 0;
end Buffer;
```

### **11.5.3 Java**
- Java threads:
	`class MyTask extends Thread {     public void run(){ ... } }`
- Thread must be started explicitly: `t.start()`.
- **Synchronized** methods/blocks act as monitors:
    - Only **one condition variable per object** → `wait`, `notify`, `notifyAll`.
    - No priority → **barging** → must wrap waits in **while**, not if.
- Cannot implement monitor-like external scheduling (no `accept`).
- No direct communication between tasks; must emulate via wait/notify.
- Virtual threads (Loom) behave similarly but lightweight.

### **11.5.4 Go**
- Lightweight goroutines:
	`go foo(3, f)`
- No direct reference to goroutine → no task object.
- Communication is via typed **channels** (CSP model):
	`ch := make(chan int, 10) ch <- 5 x := <- ch`
- `select` multiplexes multiple channels (like `_Select` on futures).
- Emphasizes message passing, not shared-memory concurrency.

### **11.5.5 C++11 Concurrency**
- `std::thread` is an OO wrapper over pthreads (kernel threads):
	`thread t1(hello, "Peter"); t1.join();`
- Supports lambdas, functors, functions.
- Must `join()` or `detach()` before thread object dies.
- **Locks**: `std::mutex`, `std::unique_lock`, `std::condition_variable`.
	`unique_lock<mutex> lock(m); cv.wait(lock, []{ return cond; });`
- Wait must be in **while** loops to prevent barging.
- Futures:
	`future<int> f = async(foo); int x = f.get();`
- Provides atomic operations via `std::atomic`.

### **11.5.6 Pthreads**
- C-level threads with manual management.
- Type-unsafe communication:
	`pthread_create(&tid, NULL, start_routine, arg); pthread_join(tid, &retval);`
- Explicit mutexes and condition variables:
	`pthread_cond_wait(&cv, &mutex);`
- Must always use **while(condition not met)** before waiting.
- No external scheduling, only internal scheduling.