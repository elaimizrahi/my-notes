- Simple batch system:
	- batch jobs together, only return control to monitor when finished (think old computers)
	- Problem: a single bad job could corrupt the entire system, run forever, for corrupt disk - So new features were added here
		- **Memory Protection** (protects kernel)
		- **Timer** (prevents overuse from one process)
		- **Privileged Instructions** (eg I/O instructions)
	- Brought on **Kernel Mode** and **User Mode**

		![[Pasted image 20250227211155.png]]
		- Problem: Batch system uses too much overhead, takes too much time for I/O

### Speeding up the batch
**UniProgramming**: Processor must wait for I/O instruction to complete before proceeding

![[Pasted image 20250227211415.png]]

**Multiprogramming**: When one job needs to wait for I/O, the processor can switch to the other job
- Multiprogramming added **Memory management** and **Input-Driven DMA and I/O**
- Maximize processor use

![[Pasted image 20250227211440.png]]

**Time Sharing**: Multiple users simultaneously access the system through terminals (think a server), this avoids loading everything individually
- Minimizes response time
- Commands entered at terminal
- Introduced **Time Slicing** and improved **Security**



## Major Development Advances

### 1. Process
A process is the data structure used by the OS to track programs
consists of 3 parts:
1. executable program
2. associated data for the program
3. execution content of the program (processor state, kernel/thread mode ... )

### 2. Memory Management
Five main goals: 
1. Process isolation
2. Automatic allocation
3. modular programming (able to swap program code)
4. Protection and access control
5. Long-term storage

These can be solved with **[[Virtual Memory]]** or **FileSystem**

### 3. Information Protection + Security
**Availability:** protect against interruption
**Confidentiality**: Assuring that users cannot read unauthorized data
**Data Integrity**: Protect data from unauthorized write
**Authenticity**: Proper verification of users and data

### 4. Scheduling and Resource Management
We have contrasting requirements for this, but need to ensure:
1. **Fairness**: equal access to resources
2. **Differential responsiveness:** differentiate between different types of jobs
3. **Efficiency**: Maximize throughput, minimize response time

## Modern Developments

**Multiprocessors** allow us to run multiple processes at once (one per processor)

**Multithreading** divides the process into threads (chunks) that can run concurrently, allowing one process to run in parallel on multiple processors

**Hypervisor:** A system virtual machine that runs multiple operating systems on the same platform

![[Pasted image 20250227212954.png]]

**Virtual Machine** a process virtual machine that allows all processes to run on the same OS
![[Pasted image 20250227213040.png]]

