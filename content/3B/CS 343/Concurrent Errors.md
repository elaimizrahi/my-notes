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
- Unlike 