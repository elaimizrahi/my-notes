While monitors work well for passive objects that need mutual exclusion, communication among tasks with a monitor is indirect

We solve this issue by sending a **message** between objects.
- Tasks can communicate directly by calling each others member routines.

![[Pasted image 20251111140412.png | 400]]
*The bottom diagram is direct communication*

## 9.1 Tasks
- Like a coroutine because it has a distinguished member (main) with its own execution state
- unique because it has thread of control
- Also like a monitor because it provides mutual exclusion and synchronization
	- Only one thread is active in the objects
- Public members are implicitly mutex and other kinds of members can be made explicitly mutex
- External scheduling allows direct calls to mutex members 
- Without external scheduling, tasks must **call out** to communicate to a third party or somehow emulate external scheduling 

![[Pasted image 20251111140934.png | 500]]
*S/ME = synchronization/mutual exclusion*

- In general, basic execution properties produce difference abstractions:
	- When thread or stack is missing it comes from calling object
	- Abstractions are not ad-hoc, rather derived from basic properties
	- Each abstraction has a particular set of problems it can solve, and therefore has a place in a programming language


## 9.2 Scheduling
- A task may want to schedule access to itself by other tasks in an order different from the order that requests arrive
	- As for monitors, there are two techniques: external and internal scheduling

### 9.2.1 External Scheduling
- As for a monitor, the accept statement can be used to control which mutex members of a task can accept calls

![[Pasted image 20251111141755.png | 500]]

**But this can also be written as:**
```c
_Accept( m1 | | m2 ) S1 ≡ _Accept( m1 ) S1; or _Accept( m2 ) S1; // S2 
if ( C1 | | C2 ) S1 ≡ if ( C1 ) S1; else if ( C2 ) S1; // S2
```

- The extended version allows different `_When` clauses after call for each accept
	- The `_When` clause is like the condition of conditional critical region
- If all the accepts are conditional and false, the statement does nothing (like `switch` with no `case` )
- If some conditionals are true, but there are no outstanding calls, the acceptor is blocked until a call to an appropriate member is made
- If several members are accepted and outstanding calls exist to them, a call is selected based on the order of `_Accepts`
