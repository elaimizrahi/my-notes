When there is a permanent blocking of a set of processes that are competing for the same resources or communicating with each other. Each deadlocked process is waiting for something that can only happen from another deadlocked process.

For concurrency, we use a **Joint Progress Diagram** to check for fatal regions
![[Pasted image 20250408131036.png]]

#### Conditions for Deadlock
1. Mutual exclusion - only one process per resource at a time
2. Hold-and-Wait - process can hold resources while waiting for others
3. No resource preemption - no resource can be forcibly removed from a process holding it
4. **Circular Wait** - closed chain of processes holding resources needed by others

**1-3** are about system policies, while condition **4** is about a sequence of operations

This can also be confirmed easier by **Resource Allocation Graphs** 
![[Pasted image 20250408131334.png]]
**Arrow in:** resource Ra requested by process P1 (P1 -> Ra)
**Arrow out:** resource Ra held by process P1 (P1 <- Ra)

#### Deadlock Prevention (check dining philosophers problem)
- Monitor solution - remove hold and wait 
	- All resources required atomically
- "room" solution - prevent circular wait 
	- cannot build circular graph
- "righty" solution - prevents circular wait 
	- cannot build circular graph


### Deadlock Avoidance
1. Do not start process if its demands might lead to deadlock
2. Do not give resource request to a process if it might lead to deadlock

##### Banker's Algorithm
- Always want to have a safe state when allocating resources
	- if allocating leads to an unsafe state, deny the resource

![[Pasted image 20250408131900.png]]![[Pasted image 20250408131907.png]]

**Issues:**
- Max resource requirement must be known beforehand
- must be a fixed number of resources to allocate
- must be independent processes
- could lead to blocking for a long time
- Must check every possibility until a valid/no solution is found

### Deadlock Detection
1. Mark processes with no allocations as run (they are ok since they don't allocate)
2. find unmarked processes where requested <= available resources and mark it
3. Update available vector by adding the resourced that were allocated for the marked process
4. Repeat 2-3 until no processes found, if any are unmarked we have a deadlock


