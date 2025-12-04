Architectural modelling is the reification of documentation of design decisions that comprise a system's architecture. We use architectural modelling notation as a language or means of capturing design decisions.

Modelling includes:
- Components
- Interconnectors
- Interfaces
- Configurations
- Rationale

### Load/Performance Testing
- We do testing after the fact
- Software needs to be built and runnable to be tested
- Non-function testing is often at the last stage of all tests, so even more delayed

### Software Performance Modelling (SPM)
We use software performance modelling to try and prevent architecture flaws, which are hard to fix after the fact, before even starting to implement the software. These are formal representations of the software to capture aspects of performance. Build as early as design and architecture models to express and understand the NFR's.
- Execution graphs
- Queuing networks
- Machine learning based performance models

SPM's allow us to:
- Estimate software performance
- Estimate Resource needs
- Find performance issues early
- Simulate execution of software

**SPM's include Software and System execution models**


## Software Execution Models
SEM's are constructed early in the dev  process to ensure that the chosen software architecture can achieve the required performance objectives. They provide static analysis of the mean, best, and worst-case response time. They are generally sufficient for finding serious performance problems early.

### Execution Graphs
- Represent the execution (sequence of operations) on the system. An execution graph is constructed for each scenario
- Not sufficient for a complete analysis of software performance, but the work well for understanding the software and its NFR's

![[Pasted image 20251130123323.png]]

![[Pasted image 20251130123421.png]]

##### Software Execution Model Analysis
- Make a quick check of the best-case response time
- Assess the performance impact of alternatives
- Identify critical parts of the system for performance management
- Derive parameters for the system execution model
We formulate algorithms to evaluate graphs

#### Basic Solution Algorithms
- We want to examine the graph and compute the time of a basic structure by reducing the basic structure to a 'computed node'
	- Basic structures are sequences, loops, and cases

![[Pasted image 20251130123702.png | 200]]![[Pasted image 20251130123713.png | 200]]![[Pasted image 20251130123758.png | 200]]

**Graph reduction for case nodes**:
- Computation for case nodes differs for shortest path, longest path, and average analyses. 
	- Shortest path: Time for the case node is minimum of all the case times
	- Longest path: Time for the case node is maximum of all the case times
	- Average analysis: Time is multiplying each node's time by its execution probability

**Software Resource Requirements**: Each basic node has specified software resource requirements $A_j$ for each service unit

**Processing Overhead Matrix**: A chart of the computer resource requirements for each of the software resource requests. 

![[Pasted image 20251130124116.png]]


## Computing total execution time
1. Uses the processing overhead matrix to calculate the total computer resources required per software resource for each node in the graph
2. Computer the total computer resource requirements for the graph
3. Compute the best-case elapsed time

![[Pasted image 20251130124659.png]]
![[Pasted image 20251130124711.png]]
![[Pasted image 20251130124630.png]]


### Queuing Network Models
Queuing Network Models are like software execution models, but characterize the software's performance in the **presence of dynamic factors**, such as other work loads or multiple users. It aims to solve the contention for resources
- More precise metrics that account for resource contention
- Sensitivity of performance metrics to variations in workload composition
- Scalability of the hardware and software
- Effect of new software on service-level objectives of other systems
- Finding Bottlenecks
- comparative data on performance improvement options

**Source of Contention:** 
- Multiple users of an application at onoe time
- Multiple aplications of systems executing on the same hardware resources at one time
- Application can have separate concurrent processes
- Application may be multi-threaded to handle concurrent requrests for different external processes

#### QNM Basics
- QNM represents the key computer system resources as queues and servers
	- A servers represents a component of the environment that provides some service to the software
	- A queue represents jobs waiting for service

 **Queues**:
 The basic component of queuing networks is a queue, also known as a service station or a service center. We use **Kendall Notation** to describe the Queue:
 $$
 A / S/m/B/K/SD
 $$
 - A: Interarrival time distribution
 - S: Service time of distribution
 - m: number of servers
 - B: Number of buffers (System capacity) 
 - K: Population size
 - SD: Service discipline

For A and S: 
- M = Exponential
- E = Erlang w/ parameter k
- H = Hyperexponential w/ parameter k
- D = Deterministic
- G = General

**Example:**
![[Pasted image 20251130125950.png]]

## Performance Metrics
- Performance Metrics of interest for each server are:
	- **Residence time RT:** the average time jobs spend in the server, in service and waiting
	- **Utilization U**: the average percentage of the time the server is busy 
	- **Throughput X**: the average rate at which jobs complete service
	- **Queue Length N**: the average numbers of jobs at the server

![[Pasted image 20251130135238.png]]
**Execution Profiles help us visualize the queue status**

**Calculations of Performance Metrics**:
- Utilization: $U = B/T$
- Throughput: $X = C/T$
- Mean Service Time: $S = B/C$
- Area under graph: $W = \sum_{time}{number\ of\ jobs}$
- Residence Time $RT = W/C$
- Queue Length $N = W/T$
T = Total period, C = Completed, B = Busy Time

The value of these depend on:
- Number of jobs
- Amount of service they need
- Time required to process each job
- Policy used to select the next job

**By knowing A/S/m/B/K/SD, we should be able to know the performance**

### Solving the Queuing Model
- Use similar calculations as performance metrics
- Instead of using real workload, base it on predicted **workload intensity** and **service requirements**
	- WI is the measure of the number of requests made in a given time interval
	- SR's are the amount of time that the workload requires from each of the devices that are processing
- Assume that the system is fast enough to handle the arrivals, and thus the completion rate or throughput equals the arrival rate
	- This is called **jobs-flow balance**

**Example**:
- Workload intensity = arrival rate = $\lambda = 0.4 \ jobs/s$
- Mean service time = $S = 2 \ seconds$
	- Throughput $X = \lambda$
	- Utilization $U = B/T = XS$ (Utilization Law)
	- Residence time $RT = \frac{S}{1-U}$
	- Queue length $N = X * RT$ (Little's Law)

![[Pasted image 20251130140429.png]]

##### Single queue analysis: Birth-death process
![[Pasted image 20251130140524.png]]
To find the steady-state probabilities of a birth-death process, we need to first define the rate that events occur. Birth is like entering the queue, death is leaving the queue.
- $\lambda_{n}$ is the birth rate at state $n$
- $\mu_{n}$ is the death rate at state $n$
- $\varphi_n$ is the probability of being in state n

Then the probability of being in having n jobs in the system is:
$$
p_n = \frac{\Pi_{i = 0}^{n-1}{\lambda_i}}{\Pi_{i = 1}^{n}{\mu_i}}p_0
$$
If we let $\rho = \lambda/\mu$:
$$
p_n = (\frac{\lambda}{\mu})^n p_0
$$
### Analysis of M/M/1 Queue
- Arrival rate is constant $\lambda$
- Death rate is constant $\mu$
![[Pasted image 20251130141040.png]]
**Note: The probability of having n or more jobs in the system is the complement of the probability of having fewer than n jobs**

$$
\rho = P (N >= n) = 1-P(N < n)
$$

![[Pasted image 20251130141142.png]]

### Analysis of M/M/m queue
- Birth rate = $\lambda$ for all n
- Death rate $\mu_n = n\mu$ for $n = 1 ... m-1$ and $\mu_n = m\mu$ for $n = m, m+1 ... inf$ 
- Utility $\rho = \lambda/m\mu$

Probability that an arriving job has to wait: **Erlang C Formula**

$$P(wait)=p_0​\frac{(mρ)^m}{m!(1−ρ)}$$

To solve for P0:
$$
p_0 = \left[ \sum_{n=0}^{m-1} \frac{(m\rho)^n}{n!} \;+\; \frac{(m\rho)^m}{m! (1-\rho)} \right]^{-1}
$$


### Queuing Network Models
A queuing network consists of two or more queues that are connected together are serve requests sent by clients. The routing of requests is specified by a probability matrix. 

#### **Open Models**
The requests come from a source that is external from the network and leave after completion

![[Pasted image 20251130143107.png]]
- Appropriate for systems with external arrivals and departures, such as ATM
- Workload is arrival rate, service requirements are teh number of visits for each device, average service time per visit, or the total demand for that device

- $\lambda$ = System Arrival Rate
- $V_i$ Number of visits to device
- $S_i$ mean service time at device

- Throughput $X_0 = \lambda$, $X_i = X_0 * V_i$
- Utilization $U_i = X_i * S_i$
- Resident time per visit at device i, $RT_i = \frac{S_i}{1-U_i}$
- queue length for device i $N_i = X_i * RT_i$
- System queue Length $N = \sum_i{N_i}$
- System response time $RT = \frac{N}{X_0}$

#### Closed Models
Closed QNM has no external arrivals or departures. A fixed number of jobs keep circulating among queues. This model needs:
- The number of users/jobs
- The think time - average delay between receipt of a response and the submission of the next
- Number of visits
- Service time
- Total demand per device

**Deriving System Model Parameters:**
1. Use queue-servers to represent the key computer resources defined and add the connections between queues to **complete a model topology**
2. Decide whether the system is best modelled as open or closed QNM
3. Determine the workload intensities for each scenario
4. Specify the service requirements

**Modelling Hints - Think about...**
- Multiple users and workload
- Average vs.Peak performance - Bases QNM's calculate average values
- Sensitivity - if a small change in one parameter causes a large change in the metrics, the model is sensitive
- Scalability - improves response times for your anticipated future loads
- Bottlenecks - the bottleneck device is the one with the highest utilization

