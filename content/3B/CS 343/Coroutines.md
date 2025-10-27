A **Coroutine** is a routine that can also be suspended at some point and resumed from that point when control returns. For more reference, see [[SE 350]] and [[Concurrency]].

Coroutines contain:
1. **Execution Location** - where the coroutine left off
2. **Execution State** - current variables, stack, etc. 
3. **Execution Status** - active, inactive, or terminated

A coroutine does not start from the beginning on each activation; it is activated at the point of last suspension.

Coroutines are the precursor to concurrent tasks, and introduce the complex concept of suspending and resume on separate stacks.

Two types of Coroutines - **Full and Semi Coroutines**

|Type|Direction of control transfer|Who resumes whom|Example analogy|
|---|---|---|---|
|**Semi-coroutine**|One-way transfer|Caller resumes coroutine|_Generator function_|
|**Full coroutine**|Two-way transfer|Coroutines resume each other|_Cooperative threads_|

## 3.1 Semi-Coroutine
A _semi-coroutine_ can yield **only back to its caller**.
- Control always flows in this pattern:  
    `main → coroutine → yield → back to main → resume coroutine → yield → ...`
- When you `yield`, you **_must_** return to whoever called you.
- When you `resume`, only the **_caller_** can wake you back up

Example for Fibonacci sequence in `uC++`:
![[Pasted image 20251027094207.png | 400]]

- **_Coroutine_** type wraps coroutine and provides all class properties
- No explicit execution state
- No parameters or return value, this is supplied by public members and communication variables
- **main()** is private/protected to hide the implementation
- All types inherit from base type `uBaseCoroutine`
- `resume` and `suspend` trigger a context switch between coroutine stacks

![[Pasted image 20251027094703.png | 400]]
