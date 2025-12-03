### 7.1 Race Condition
- A race condition occurs when there is missing synchronization or mutual exclusion
- Two or more tasks race along assuming synchronization/mutual exclusion has occurred
- Can be difficult to locate 
### 7.2 No Progress 
#### 7.2.1 Live-lock 
- Indefinite postponement: "You go first" problem on simultaneous arrival (symptom consuming CPU)
- Caused by poor scheduling in entry protocol
- Always mechanism to break tie on simultaneous arrival to deal with live-lock

#### 7.2.2 Starvation
- A selection algorithm ignores one or more tasks so they are never executed 
- Long-term (infinite) starvation is rare, but short-term can occur and is a problem
- Like live-lock, the starving task might be ready at any time, switching among active, ready, and possibly blocked

#### 7.2.3 Deadlock 
- Deadlock is the state when one or more processes are waiting for an event that will not occur
- Unlike live-lock/starvation, deadlocked task is usually blocked so it's not consuming CPU

##### 7.2.3.1 Synchronization Deadlock
- Failure in cooperation, so a blocked task is never unblocked
```c
int main{
	uSemaphore s(0); //closed
	s.P(); // wait for lock to open
}
```
#### 7.2.3.2 Mutual Exclusion Deadlock 
- Failure to acquire a resource protected by mutual exclusion
![[Pasted image 20251202164043.png | 400]]
- Deadlock, unless one of the cars is willing to backup 
- There are 5 conditions that must occur for a set of processes to deadlock
	- A concrete shared resource requiring mutual exclusion
	- A process holds a resource while waiting for access to a resource held by another process
	- Once a process has gained access to a resource, the runtime system cannot get it back
	- There exists a circular wait of processes on resources
	- There conditions must occur simultaneously

```c
// Simple deadlock example using semaphores
uSemaphore L1(1), L2(1);
_Task task1 {
    void main() {
        L1.P();      // acquire L1
        // use R1
        L2.P();      // acquire L2
        // use R1 & R2
    }
};

_Task task2 {
    void main() {
        L2.P();      // acquire L2
        // use R2
        L1.P();      // acquire L1
        // use R2 & R1
    }
};
```

### 7.3 Deadlock Prevention
- We want to eliminate one or more of the conditions required for a deadlock from an algorithm so that a deadlock can never occur
#### 7.3.1 Synchronization Prevention
- Eliminate all synchronization from a program
- No communication
- impossible in most cases

#### 7.3.2 Mutual Exclusion Prevention
- Deadlock can be prevented by eliminating one of the 5 conditions:
1. Eliminate Mutual Exclusion (Impossible because most resources must be used exclusively)
2. Eliminate Hold and wait - do not give any resource unless all resources can be given. Not good for resource utilization and might starve
3. Allow preemption - Let the OS forcibly take resources away from tasks. But, resource use is dynamic and we can't enforce static policies
4. Eliminate circular wait (Best Option)
	- Key idea: impose a global ordering on resources `i.e. R1 < R2 < R3`
	- A talk may only request resources in increasing order
	- Guaranteed that if there's no cycle there's no circular wait and no deadlock
5. Prevent all 4 conditions from happening together. 

### 7.4 Deadlock Avoidance
- Monitor all lock blocking and resource allocation to detect any potential formation of deadlock
- Achieve better resource utilization, but additional overhead to avoid deadlock

#### 7.4.1 Banker's Algorithm
- Demonstrate a safe sequence of resource allocations that imply no deadlock
- However, requires a process to state its max resource needs

![[Pasted image 20251202165820.png]]

- Is there a safe order of execution that avoids deadlock should each process require its maximum resource allocation?
- A safe order exist and hence the algorithm allows the resource request
- If there is a choice of processes to choose for execution, it does not matter which path is taken

#### 7.4.1 Allocation Graphs
- One method to check for potential deadlock is to graph processes and resource usage at each moment a resource is allocated  

![[Pasted image 20251202170237.png]]

Use Graph reduction to locate deadlocks:
![[Pasted image 20251202170312.png]]

### 7.5 Detection and Recovery
- Instead of avoiding deadlock let it happen and recover
- Discovering deadlock is difficult
- Recovery involves preemption of one or more processes in a cycle

### 7.6 Which Method To Choose
- Maybe just ignore the problem
- Only the ordered resource policy turns out to have much practical value