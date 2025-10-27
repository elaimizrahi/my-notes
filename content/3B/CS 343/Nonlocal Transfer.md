
**Quick Review - Advanced Control Flow**
- Do not use `while` to simulate `for`
- loop exists **never** need an else clause
![[Pasted image 20251026114835.png | 400]]
- We NEVER want to use flag variables. It becomes a flag when it is used solely to affect control flow and no calculations

**Quick Review - Static Multi-level exit**
- exits control structures where it's known at **compile time**
- done using `break/continue`
- We can almost always get rid of flag variables by using multi-level exits

**Quick Review - Dynamic Allocation**
- Don't use the heap when you don't have to! 
![[Pasted image 20251026120034.png|400]]
-  we can increase the stack size from the shell with `ulimit -s unlimited`
- Some situations where dynamic allocation is necessary:
	- Storage must outlive the block where it's allocated
	- Amount of data read is unknown
	- Array elements must be initialized via the object's constructor to different values
		- Must use explicit pointers + dynamic alloc or `unique_ptr`
		- `uC++` lets us do this with the macro `uArray(T, arr, size)` 
			- allocates w/o ctor calls, is O(1) in stack, while unique_ptr is O(n) in the heap
			- `uArrayFill(T, arr, size, val)` to fill array with `val`
			- Cannot pass `uArray` as a ptr, need to use `uArrayRef`
		- When large local variables are allocated on a small stack


## Nonlocal Transfer
**Nonlocal transfer** refers to the transfer of **control flow** that _jumps across function boundaries_—that is, control doesn’t return normally through the call stack, but instead “jumps” to another part of the program that isn’t its direct caller.
![[Pasted image 20251026120824.png]]
routine h calls g calls f, but cannot return from f to h, so how to solve?

**Traditional approaches**
1. Return codes that tell us where to return to
2. Status flags that keep track of current states
3. Fixup routines `try/catch` to handle cases

Techniques are often combined:
```
if ( printf(...) < 0 ) { // check return code for error  
	perror( "printf:");  // errno describes specific error
	abort();             // terminate program
}
```

4. Return union - ALL routines must return the right codes - combines result/return code and requires return-code check on result access
	- Forces checking
	- **like Fortran, only returns one level**

**Drawbacks**
- Checking return code of status flags is optional => can be deulayed or omitted (passive vs. active)
- Return code mixes exceptional and normal values -> enlarges type or value range; normal/exceptional types/values should be independent
- Makes code difficult to read, each call results in multiple statements
- Library routines should not terminate program
- Status flags can be overwritten before being read
- Fixup routines increase cost of the call and introduce new status flags

### 2.2 Dynamic Multi-level exit
- We want to extend call/return semantics to transfer in the **reverse** direction to normal routine calls, requiring nonlocal transfer
![[Pasted image 20251026121957.png]]
- **label variable** contains pointer to a block activation on stack + transfer point within block
So it’s like saying:
> “If I `goto L`, jump to this specific place _in that specific function call_.”

If `f()` executes `goto L`, control jumps directly to `L1` or `L2` in `h()` — skipping normal returns

Label constants have **local scope** because recursion complicates where `L` points:
- If `g` is recursive, there may be multiple stack frames between `f` and `h`.
- Which one should `L` refer to?

**In C we can perform DME's with `setjmp` and `longjmp`**
- `setjmp` to initialize a label variable
- `longjmp` to goto a label variable

### 2.3 Exception Handling
- DME is usually called exception handling
	- More than error handling
- Exceptional event: known to exist but is *ancillary* to an algorithm
- Exception handling mechanisms provides some or all of the alternate kinds of control flow

### 2.4 Terminology
**Faulting Execution:** execution changing control flow due to a raised exception
**Local Exception:** when an exception is raised and handled by the same execution => source = faulting
**Nonlocal exception:** when an exception is raised by a source execution but delivered to a different faulting execution => source /= faulting
**Concurrent exception:** a nonlocal exception where the source and faulting executions are executing concurrently
**Propagation:** directs control from a raise in the source execution to a handler in the faulting execution
**Propagation mechanism**: is the rules used to locate a handler
- most common give precedence to handler higher in the call stack - specificity vs. generality, efficient linear search during propagation
**Guarded block:** is a language block with associated handlers, e.g., try-block in C++/Java.
**Unguarded block:** is a block with no handlers

### 2.5 Execution Environment
The execution environment has a significant effect on an EHM
- object-oriented concurrent environment requires more complex EHM 
- Control structures with `finally` clauses must always be executed
	- So terminating a black complicates the EHM as object destructors and **finally** clauses must be executed
- in `uC++` we use `_Finally { ... }`

### 2.6 Implementation
- DME is limited in most programming languages using exception handling
To correctly implement `throw`/`catch`, the runtime must **know the last guarded block** (i.e., the most recent `try` that can handle the exception type).

One simple (but inefficient) approach:
1. Associate a **label variable** with each exception type.
2. When entering a `try` block, **set** the label for that exception type.
3. When exiting the block, **reset** it to the previous value.
This way, a `throw` can simply `goto` the current label for that exception type.

Drawback: If you have

```
void rtn(int i) {   try { ... }     // set label on entry   
	catch(E) { ... } // reset label on exit 
}
```

and `rtn()` is called **millions of times**, but the exception is **never raised**,  
then you’re doing **millions of unnecessary label updates**.
- Instead, catch/destructor data is stored once externally for each block and handler found by linear search during a stack walk (no direct transfer).
- So termination is implemented using zero cost on guarded-block entry but expensive approach on raise (less likely)

### 2.7 Static/Dynamic Call/Return
- static/dynamic call: routine/exception name at the call/raise is looked up statically/dynamically
- static/dynamic return: after a routine/handler completes, it returns to its static (definition) or dynamic (call) context


|                | call/raise | call/raise                                   |
| -------------- | ---------- | -------------------------------------------- |
| return/handled | Static     | Dynamic                                      |
| static         | sequel     | termination exception                        |
| dynamic        | routine    | routine pointer, virtual routine, resumption |

### 2.8 Static Propagation (Sequel)
A sequel is a routine with no return value, where: 
- the sequel name is looked up lexically at the call site
- But control returns to the end of the block in which the sequel is declared
![[Pasted image 20251026125757.png | 500]]

- Advantage is that the handler is statically known (like static multi-level exit) and can be as efficient as a direct transfer 
- Disadvantage is that it only works for monolithic programs (must be statically nested at the point of use)
	- fails for library code
	- if stack is separately compiled, the sequel call in `push` no longer knows the static blocks containing calls to it

### 2.9 Dynamic Propagation
- For termination and resumption, both have dynamic raise with dynamic/static returns
- Advantage is that it works for separately compiled programs
- Disadvantage is the handler is not statically know (we use it as an advantage)
	- without dynamic handler selection, the same action and context for that action is executed for every exceptional change in control flow

#### 2.9.1 Termination
- Control transfers from the start of propagation to a handler => dynamic raise (call)
- When handler returns, performs static return => stack is unwound like sequel

##### Two main forms for _non-recoverable_ operations
- **Terminate:** end current operation and unwind stack to handler.
- **Retry:** restart guarded block after handling the exception.

**Terminate:**
```struct E {};
void f() { if (...) throw E(); }

int main() {
  try { f(); } catch (E) { /* handler 1 */ }
  try { f(); } catch (E) { /* handler 2 */ }
}
```

**Retry:**
```// Retry
try { ... } retry { ... }

// Simulation
while (true) {
  try { ... break; }
  catch (...) { /* fix and retry */ }
}
```


| Topic                   | Key Idea                                 |
| ----------------------- | ---------------------------------------- |
| **Termination**         | Stack unwinding to nearest handler       |
| **Dynamic propagation** | Exceptions bubble up call chain          |
| **Bound catch**         | Handle based on source object            |
| **Retry/Simulation**    | Restart mechanism for recoverable errors |
| **I/O exceptions**      | Raised automatically by stream errors    |

### 2.9.2 Resumption
- **Resumption** provides a limited mechanism to generate new blocks on the call stack
	- Control transfers from the start of propagation to a  handler -> dynamic raise (call)
	- when handler returns, it is dynamic return => stack is NOT unwound (like routine)
![[Pasted image 20251026134918.png]]
