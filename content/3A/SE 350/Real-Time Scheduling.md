**Task:** real-time computation unit
- execution thread with additional timing parameters
**Hard real-time:** missing deadlines is not an option
- must use worst-case service time
- only schedulable if it meets all deadlines under all possible scenarios
**Periodic Task:** task is activated periodically every T time units

We denote tasks $\tau_i$ service time $C_i$ Relative deadline $D_i$ and period $T_i$
- for each job $\tau_{ij}$ of $\tau_i$:
	- Arrival time: $a_{ij} = a_{ij-1} + T_i$
	- Sporadic: $a_{ij} >= a_{ij-1} + T_i$
	- Absolute deadline: $d_{ij} = a_{ij-1} + D_i$

**Task utilization for a periodic/sporadic task:**  $U_i = C_i/T_i$
- percent of processor time required by the task

**System utilization:** $U = U_1 + U_2 +\ ... + U_N$

#### Schedulability analysis
We set a bound and must check that $U <= bound$ 
- Default (base): Task clearly not schedulable if $U > 1$
- For most algorithms, we define a bound $U_b$ such that the task is schedulable if $U <= U_b$
This tells us if we are able to schedule a task set

#### Rate-Monotonic scheduling (RM)
- fixed priority scheduling algorithm 
- assigns priorities on the bases of the tasks periods (shorter period = higher priority)

**Liu & Layland schedulability analysis:** schedulable if $U \leq U_b(N) = N \left( 2^{1/N} - 1 \right) \xrightarrow{N \to \infty} \ln 2 = 0.693$
- But if $U_b(N) < U <= 1$, then nothing can be said according to the analysis

#### Earliest Deadline First (EDF)
- Dynamic priority scheduling algorithm
- - priority is inversely proportional to its current absolute deadline
- earlier deadline - higher priority
A task set is scehdulable if $U <= U_b = 1$
- Since tasks with $U > 1$ cannot be schedule by any algorithm, EDF is optimal (if it can be scheduled, EDF will succeed)

#### Comparisons
- in practice RM will be chosen over EDF
	- For most task sets, RM has a better $U_b$ than log 2
	- if task periods are harmonic, then $U_b = 1$ 
	- RM is easier to implement when there is a limited number of priority levels
	- RM is more transparent - easier to understand what's wrong 
		- if a task executes for too long higher priority tasks not affected

If $D_i != T_i$:
- EDF still better
- instead of RM use DM (deadline monotonic)

If $D_i < T_i$:
- $U_b$ still works by changing periods to deadlines


