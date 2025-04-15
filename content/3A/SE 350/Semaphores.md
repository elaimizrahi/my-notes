
Notes taken from [The Little Book of Semaphores](https://greenteapress.com/wp/semaphores/)

Semaphores are like integers - with three differences:
1. Can initialize to any int, but can only increment or decrement by 1 afterward
2. When a thread decrements, if the value is < 0 then the thread blocks itself until another increments the semaphore
3. When a thread increments, if there are other threads waiting, one of the waiting threads is unblocked

### Syntax 
- Initialized with `s = Semaphore(1)`
- Increment with `s.increment()`
	- also can be `s.signal()` (which is what we use in SE350)
- Decrement with `s.decrement()`
	- also can be `s.wait()` (which is what we use in SE350)

### Reasoning
 - impose constraints that help programmers avoid errors
 - Solutions are often much cleaner
 - Can be implemented efficiently

# Patterns
This is the most important part of SE350 and is how we implement concurrency 
- Can also be done with monitors, but they were not covered due to time
- **For all solutions, they are wrapped in a `while(true)` loop**
#### Signalling
The simplest use for semaphores is signalling, where one thread sends a signal to another to indicate something has happened

ie. 
**Thread A**
```c
// some code (a1)
sem.signal()
```
**Thread B**
```c
sem.signal()
// some code (b1)
```

The semaphore in this program guarantees that a1 has completed before thread B begins b1

### Rendezvous
**Problem**: Generalize the signal pattern so that it works both ways. A has to wait for B and vice versa.
**Thread A**
```c
statement a1
statement a2
```
**Thread B**
```c
statement b1
statement b2
```

We want a1 to happen before b2 and b1 to happen before a2!

**Solution**
- Create two semaphores named `aArrived` and `bArrived`
	- as suggested, they indicate when each thread has arrived at the rendezvous

**Thread A**
```c
statement a1
aArrived.signal()
bArrived.wait()
statement a2
```
**Thread B**
```c
statement b1
bArrived.signal()
aArrived.wait()
statement b2
```

This ensures that Thread A waits for B to arrive before starting a2, and vice versa for B


### Mutex (Mutual Exclusion)

We can use semaphores to also enforce mutual exclusion - that is **only** one thread accesses the shared variable at a time
- We can think of them as a token that passes from one thread to another, allowing one thread at a time to proceed
	- think like a talking stick where only one person can talk at a time and they need to have the stick
**Problem**:
Use semaphores to the following example to enforce mutual exclusion for the variable `count`
**Thread A**
```c
count = count + 1
```
**Thread B**
```c
count = count + 1
```

**Solution**:
- create one semaphore `countAvailable` initialized to 1 (this is a **Mutex**)
	- every time a thread wants to use a semaphore, call `countAvailable.wait()`

**Thread A** 
```c
countAvailable.wait()
// Critical Section
count = count + 1
countAvailable.signal()
```
**Thread B** 
```c
countAvailable.wait()
// Critical Section
count = count + 1
countAvailable.signal()
```


#### Multiplex 
**Problem**: A multiplex problem is like a Mutex one, but in this case we want to allow some $n$ number of threads to access their critical sections at the same time
- This pattern is a multiplex, which can be seen in real life by only allowing a certain amount of people into a place at one time - with someone at the door keeping track

**Solution**: 
- create a semaphore `multiplex` initialized to the number $n$ that is the limit
	- follow the same pattern as Mutex!
**All n Threads** 
```c
multiplex.wait()
// Critical Section
multiplex.signal()
```


### Barrier 
**Problem:** Consider the rendezvous problem from before - the problem is that this solution only works with two threads. How do we make it work for any number?

Every thread should run the following code:
```c
rendezvous
critical point
```

**The requirement is that no thread executes `critical point` until after all have executed `rendezvous`**
- we can assume there are n threads and the value is stored in a variable n that is accessible from all threads


**Solution**:
- create a variable `count` that keeps track of the number of threads that have arrived
- Create a `mutex` semaphore that provides exclusive access to `count`
- Create a final semaphore `barrier` which is **locked** (0) until **all** threads arrive, at which point it should be **unlocked** (1+)

**Setup**
```c
n = number of threads
count = 0
mutex = Semaphore(1)
barrier = Semaphore(0)
```

**All threads**:
```c
mutex.wait()
count += 1
mutex.signal()
if(count == n){
	barrier.signal()
}
barrier.wait()

barrier.signal() 
// If we don't have this line, will have deadlock since we decrement without re-incrementing, which will cause the final one to not get notified that it can run

// Note also that if we move mutex.signal() to the bottom, the first thread would get blocked at barrier.wait() and never release the mutex, causing a deadlock

// Critical section
```

This pattern of a wait and signal in rapid succession is called a **turnstile**, because it allows one thread to pass at a time, and it can be locked to stop all threads
- Initialized to locked (0), and the nth thread unlocks it to let all n threads go through
### Reusable Barrier 
Often a set of cooperating threads will perform a series of steps in a loop and sync at the barrier after each step. For this,, we need a barrier that locks itself after all the threads have passed through

**Problem:** Rewrite the barrier solution so that after all threads have passed through, the turnstile is locked again

**Solution**: 

**Setup**
```c
n = number of threads
count = 0
mutex = Semaphore(1)
turnstile = Semaphore(0)
turnstile2 = Semaphore(1) // Added because these threads run in a while loop, so we need to make sure it's ok when going back to the rendezvous (a thread can go all the way from turnstile.signal() to turnstile.wait())
```

**All threads**:
```c
rendezvous

mutex.wait()
count += 1
if(count == n){
	turnstile.signal()
	turnstile2.wait()
	
}
mutex.signal()
// Observe that the check on count is inside the mutex so that a thread cannot be interrupted after changing the counter adn before checking it


turnstile.wait()
turnstile.signal() 

mutex.wait()
count -= 1
if(count == 0){
	turnstile.wait()
	turnstile2.signal()
}
mutex.signal()
// Same issue here is fixed

turnstile2.wait()
turnstile2.signal()

// Critical section
```


### Queue
Semaphores can also be used to represent a queue. In this case, it is initialized to 0, and you can't signal unless there is a thread waiting so the value of the semaphore is never positive
**Problem:** Use semaphores to enforce this

**Solution**:

**Setup**:
```c
leaderQueue = Semaphore(0)
followerQueue = Semaphore(0) 
```
`leaderQueue` is where the leaders wait and `followerQueue` is where the followers wait

**Leaders Thread**:
```c
followerQueue.signal()
leaderQueue.wait()
dance()
```

**Followers Thread**:
```c
leaderQueue.signal()
followerQueue.wait()
dance()
```


# Synchronization Problems

## Producer/Consumer
 We often have a division of labor between threads in multithreaded programs. A common pattern of this is **producer/consumer** where producers create items and add them to a data structure while consumers remove and process the items
- A good example is event-driven programs, where a thread will produce the event (handle a keypress and add to queue) while another will consume and process the event (event handler in UI seen in [[CS 349]])

**Problem:** use semaphores to solve this problem, thinking about the following constraints:
- While an item is being added or removed from the buffer, it is in an inconsistent state so threads must have exclusive access to the buffer
- If a consumer arrives while the buffer is empty, it blocks until a producer adds a new item

Assume producers perform the following over and over:
```c++
event = waitForEvent()
buffer.add(event)
```
and consumers:
```c++
event = buffer.get()
event.process()
```


**Solution:**
- create a mutex so only one can access the buffer at a time
- keep track of the number of items with a semaphore to know if the buffer is empty
- create a **local variable** event, which means each thread has its own version

**Setup:**
```c++
mutex = Semaphore(1)
items = Semaphore(0)
local event
```

**Producer Thread**
```c++
event = waitForEvent()
mutex.wait()
buffer.add(event)
items.signal()
mutex.signal()
```

**ConsumerThread**
```c++
items.wait()
mutex.wait()
event = buffer.get()
mutex.signal()
event.process()
```


## Readers-Writers

The reader-writer problem pertains to any situation where a data structure, database, or file system is read and modified by concurrent threads
- While the data structure is being written or modified, we must bar others from reading 

**Problem:** a writer cannot enter the critical section while any other thread (reader or writer) is there, and while the writer is there, no other thread may enter. 
- Any number of readers can be in the critical section simultaneously
- Writers must have exclusive access to the critical section

This exclusion pattern is sometimes called **categorical mutual exclusion**
**Note:** we usually call the data structure a "room"

**Solution**:
- We want to keep track of the readers, and know if the room is empty
- mutex protects the `readers` counter

**Setup:**
```c++
int readers = 0
mutex = Semaphore(1)
roomEmpty = Semaphore(1)
```

**Reader Thread**:
```c++
mutex.wait()
	readers += 1
	if(readers == 1){
		roomEmpty.wait() // first reader in locks
	}
mutex.signal()

room.read()

mutex.wait()
	readers -= 1
	if(readers == 0){
		roomEmpty.signal() // last reader out unlocks
	}
mutex.signal()
```

**Writer Thread**:
```c++
roomEmpty.wait()
	room.write()
roomEmpty.signal()
```

**This is such a common pattern that we have named it a Lightswitch, which we can use as needed**
```c++
class Lightswitch:
	def __init__(self): 
		self.counter = 0 
		self.mutex = Semaphore(1)

	def lock(self, semaphore): 
		self.mutex.wait()
			self.counter += 1
			if self.counter == 1:
				semaphore.wait()
		self.mutex.signal()

	def unlock(self, semaphore):
		self.mutex.wait()
			self.counter -= 1
			if self.counter == 0:
				semaphore.signal()
		self.mutex.signal()
```

So now:
**Reader Thread**:
```c
readLightSwitch = Lightswitch()
roomEmpty = Semaphore(1)

// solution
readLightSwitch.lock(roomEmpty)
// critical section
readLightswitch.unlock(roomEmpty)
```


#### Starvation
We may have the problem where a writer arrives, but many readers keep coming and going and the room is never empty. Here, the writer would "starve" and not ever get to perform its action

**Solution**:
- We want to use a turnstile to ensure that all the readers + writers have a chance to take their turn, then they will continue the loop

**Setup**:
```c
readSwitch = Lightswitch()
roomEmpty = Semaphore(1)
turnstile = Semaphore(1)
```
**Reader Thread**:
```c++
turnstile.wait()
turnstile.signal()
readSwitch.lock(roomEmpty)
	room.read()
readSwitch.unlock(roomEmpty)
```

**Writer Thread**:
```c++
turnstile.wait()
	roomEmpty.wait()
		room.write()
turnstile.signal()
roomEmpty.signal()
```

- If a writer arrives while there are readers, it will block at line 2, which means the turnstile will be locked
- When the last reader leaves, it signals roomEmpty, unblocking the waiting writer, which then enters its critical section


## Dining Philosophers
The standard representation of this problem features a table with five plates, five forks, and a single bowl of spaghetti. Five philosophers (interacting threads) come to the table and:
```c
while True:
	think()
	get_forks()
	eat()
	put_forks()
```

- Each philosopher must hold **Two** forks to eat, so a philosopher might have to wait for another to put theirs down before eating

**Problem:** write a version of `get_forks` and `put_forks` that satisfies the following:
- Only one philosopher can hold a fork at a time
- No deadlocks can occur
- No philosopher can starve waiting for a fork
- More than one philosopher should be able to eat at once
												![[Pasted image 20250410130039.png]]

**Solution**:
- We represent forks with the functions `left` and `right`
- Use a list of semaphores to represent which forks are available
```python
def left(i): return i 
def right(i): return (i + 1) % 5
forks = [Semaphore(i) for i in range(5)]
```

**Observe:** if we have only four philosophers, a deadlock is impossible as there will always be an extra fork if everyone picks up a fork
- so, control the number of philosophers with a **Multiplex** initialized to 4 named `footman`

Now:
```c
def get_forks(i):
	footman.wait()
	fork[right(i)].wait()
	fork[left(i)].wait()

def put_forks(i):
	fork[right(i)].signal()
	fork[left(i)].signal()
	footman.signal()
```


## Cigarette Smoker's Problem
We have four threads: an agent and three smokers. The smokers loop forever, first waiting for ingredients, then making and smoking the cigarettes. The ingredients are tobacco, paper, and matches. The agent has an infinite supply of ingredients, and each smoker has an infinite supply of one of the ingredients (one has matches, other paper, other tobacco)
- If the agent puts out tobacco and paper, the smoker with the matches should pick up both, make a cigarette, and signal to the agent

This problem is really interesting when you can't change the agent code


**Solution:**
- use three helper threads called "pushers" that respond to the signals from the agent, keep track of ingredients, and signal the appropriate smoker
**Setup:**
```c
isTobacco = isPaper = isMatch = False // is it on the table?
tobaccoSem = Semaphore(0) // signal to smoker
paperSem = Semaphore(0)
matchSem = Semaphore(0)
agentSem = Semaphore(0)

```

**Pusher A**:
```python
tobacco.wait()
mutex.wait()
	if isPaper:
		isPaper = False
		matchSem.signal()
	elif isMatch: 
		isMatch = False
		paperSem.signal()
	else:
		isTobacco = True
mutex.signal()
```

- we have the same with pusher B and C, just swapped for each item

**Smoker (with tobacco)**
```c
tobaccoSem.wait()
makeCigarette()
agentSem.signal()
smoke()
```
- we have the same with smoker B and C, just swapped for each item

This is a rough solution, but overall the idea is to ensure that the pushers update the item signals, and the smokers wait for these to update then signal to the agent (which has its own code to wait it's own signal, update to the new item and send out that signal)


