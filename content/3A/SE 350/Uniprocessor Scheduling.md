Assign processes to be executed by the processors
Decide which ones will wait/progress

### Long-Term Scheduling
Control program admission. Decides which ones are even considered to be scheduled 
- Can the scheduler have another process?
- Which process should it take on (FIFO, priority, etc)

### Medium-Term Scheduling
Part of the swapping function, decides which one to select
- Depends on available virtual memory and how much multiprogramming is wanted

### Short-Term Scheduling
Dispatcher
- Executes the most frequently
- Invoked from:
	- Clock interrupts
	- I/O interrupts
	- System calls
	- yield
	- signals
 - **Criteria**
	 - **User Oriented (UO)**
		 - Throughput (how many process per unit of time)
		 - Response time (time between submission and completion of process)
	- **System Oriented (SO)**
		- Throughput
		- Processor utilization (percent of time processor is busy - we want higher)
		- Device utilization (same but for I/O)
		- Fairness (split share of resources equally)

### Decision Making
**Non-Preemptive**
- Process will run until it can't anymore
**Preemptive**
- Processes may be interrupted and paused by the OS
- Allows for better service
- Scheduler must run more often

**Arrival time** is when a process is submitted
**Finish time** when it's completed 
**Service time** is the time the processor must spend to execute (unknown before)
**Turnaround time** is $finish\ time - arrival\ time$ 
**Normalized turnaround time** is $(turn\ around\ time)/(service\ time)$

### Scheduling Options

#### First come first served
 - simplest, least overhead
 - non-preemptive
 - a short process might have to wait a while before it runs
 - favours CPU-bound processes 
 - still blocks and queues based on I/O

#### Preemptive Round Robin
- Time slicing
- Same as FIFO but preemptive
	- a process runs until it hits the time slice, where it gets thrown in the back of the queue
- Reduced throughput, more overhead

**Note:** ![[Pasted image 20250408120602.png]]

#### Shortest Process Next
- Non-preemptive 
- Selects the one with shortest **absolute** service time
- increases throughput, reduces turnaround time
- unfair!


**Note:** Service time estimated by $S_{n+1} = \alpha T_n + (1 - \alpha) S_n$
	where $S_n$ is the predicted execution time at step n and $T_n$ is the measured execution time at step n

#### Shortest Remaining Time (EDF)
- Preemptive version of **SPN**
- must estimate processing time like SPN
- used in labs

#### Highest Response Ratio 
- Nonpreemptive
- choose next process with the greatest ratio:
		$Ratio = (time\ spent\ waiting\ + expected\ service\ time)/(expected\ service\ time)$

#### Priorities
- Processes are given priorities (0 -> highest)
- Scheduler chooses based on available ones and their priorities
- Tough to assign priorities


#### Feedback Scheduling
- All processes have priority 0
	- Each process executes for a time quantum
	- Every time a process is preempted, its priority is lowered by one
	- RR within same priorities


Cool note, not needed for exam:
![[Pasted image 20250408121458.png]]