A **Coroutine** is a routine that can also be suspended at some point and resumed from that point when control returns. For more reference, see [[SE 350]] and [[3A/SE 350/Concurrency]].

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

**Correct Usage and Construction:**
- Eliminate computation or flag variables retaining information about execution state
- When possible, the simplest coroutine construction is to write a direct (stand-alone) program
- Convert to coroutine by: 
	- putting processing code into coroutine main
	- converting reads if program is consuming or writes if program is producing to suspend (convert writes `cout` to suspend)
	- use interface members and communication variables to transfer data in/out of coroutine
- Note that the above approach is impossible for advanced coroutine problems

#### 3.2 `uC++` EHM
- exceptions must be generated from a specific kind of type
- supports two kinds of raising: throw and resuming
- supports two kinds of handlers, termination and resumption
- supports propagation of nonlocal and concurrent exceptions
- all exception types are grouped into a hierarchy

#### 3.3 Exception Type
- C++ allows any type to be used as an exception type
- `uC++` requires exception types to those defined by `_Exception`
	- `_Exception exception_type_name {...}`
- has all properties of a class

#### 3.4 Inherited Members
- Each exception type inherits the following members from `uBaseException`
```c
class uBaseException { // like std::exception  
    uBaseException( const char * const msg = "" );  
    const char * const message() const; // C++ std::exception::what 
    const uBaseCoroutine & source() const;  // coroutine that raise E
    const char * sourceName() const; // name of coroutine
    virtual void defaultTerminate();  // if E thrown but not handled - forward UnhandledException
    virtual void defaultResume(); // if E resumed but not handled - throw the exception
    };
```

- `uBaseException( const char * const msg = "" )` – msg is printed if the exception is not caught.


#### 3.5 Raising
Two methods: 
1. `_Throw [exception-type]`
	- if no exception-type, it is a re-throw
2. `_Resume [exception-type] [_At uBaseCoroutine-id]`
	- if no exception-type, it is a re-resume
	- `_At` clause allows the specified exception or current one to be raised at another coroutine/task

![[Pasted image 20251027100727.png]]

In the above case, `C++` would print **B** while` uC++` would print **D**

#### 3. Handler
Two kinds of handlers:

**Termination**: 
- Termination is done the same as C++, with the `catch` clause in a `try` block

**Resumption:**
- `uC++` extends the try block to include resumption with **`_CatchResume(E) {...}`**
	- must appear before catch clauses 

*Note:* Resumption handlers cannot perform a break, continue, goto, or return, but it can throw another exception

**Termination/Resumption**
- When a `_Resume()` is thrown, only `_CatchResume()` blocks will be checked
- When a `_Throw()` is thrown, only `catch()` blocks will be checked
- If no blocks are found for the respective handler, it will use the `_DefaultResume()` or `_DefaultThrow` respectively

*Note*: **\_Resume()/\_Throw** implicitly store the **this** associated with the member raising an exception

#### 3.7 Nonlocal Exceptions
A **nonlocal exception** is an exception raised by one coroutine (the **source**) but **handled in another coroutine** (the **faulting coroutine**)
- Each coroutine has **its own stack**, so normal (“local”) exceptions stay within that stack.
- Nonlocal exceptions are raised using \_Resume ... \_At
- We allow nonlocal exceptions by wrapping the suspend section in `_Enable()`
- **Nonlocal exceptions** jump _across coroutine stacks_.  
    → Think of it as throwing an exception _from one thread_ into another coroutine that’s currently suspended.

![[Pasted image 20251027102550.png | 400]]

A nonlocal exception is raised using:
`_Resume E() _At c;`

This means:
> “Raise exception `E` inside coroutine `c`.”

1. **Coroutine must be suspended first.**  
    You can only inject an exception into a coroutine that’s currently paused.  
    Otherwise, it has no point to resume into.
2. **The source doesn’t handle it.**  
    The source coroutine (that called `_Resume E() _At c`) immediately continues after issuing the resume.
3. **The faulting coroutine (`c`)**:
    - Performs a local `_Resume` internally when it’s reactivated. 
    - Handles the exception inside its `_CatchResume` block.
4. **If it doesn’t handle the exception:**
    - It raises an internal `_UnhandledException` in itself,
    - which is propagated back to whoever last resumed `c`.

![[Pasted image 20251027103922.png]]

#### 3.8 Memory Management
![[Pasted image 20251027104200.png]]

Coroutine stacks expand to the next stack
- They are normally allocated in the heap with stack size 256K and **dows not grow**
- can be adjusted in the coroutine constructor definition -> `C(): uBaseCoroutine(8192)` for 8K stack
- can check for stack overflow with member `verify()`

#### 3.9 Semi-Coroutine Example
##### 3.9.1 Same Fringe

**Goal:**  
Check if two binary trees have the same sequence of **leaf nodes** (the “fringe”).

**Idea:**  
Traverse both trees in order, yielding each leaf value as it’s found — compare them one-by-one.

**Why coroutines help:**  
A normal recursive traversal would need to store one tree’s leaves (extra data structure).  
A coroutine can _suspend_ at each leaf and resume later, removing that need.

**Algorithm summary:**
- Each tree iterator is a coroutine that:
    1. Recursively walks the tree.
    2. `suspend()`s whenever it reaches a leaf.
    3. Resumes when the main function calls `next()`.
        

```c
// Coroutine iterator for leaf traversal 
_Coroutine Iterator {  
	 Node *cursor;   
	 void walk(Node *n) {     
		 if (!n) return;     
			 if (!n->left && !n->right) { cursor = n; suspend(); }
			 walk(n->left); 
			 walk(n->right);   
	}   
	void main() { walk(cursor); cursor = nullptr; } 
	public:   
		T* next() { resume(); return cursor; } 
	};
}
```


**Comparison (actual method):**

```c
bool sameFringe(BTree& t1, BTree& t2) {   
	auto i1 = Iterator(t1), i2 = Iterator(t2);   
	for (;;) {     
		T *a = i1.next(), *b = i2.next();     
		if (!a || !b) return a == b;      // both done?     
		if (*a != *b) return false;       // mismatch   
	} 
}
```

Elegant: traversals advance in lockstep without extra stacks.


## 3.10 Full Coroutine
- A **semi-coroutine** always resumes back to its **starter** (one-way control).
- A **full coroutine** supports **bidirectional resumption** — control can pass between _any_ coroutines in a **resume cycle**.
```
semi-coroutine: main → A → suspend → main
full coroutine:  A ↔ B ↔ C ↔ A (cycle)
```

- `suspend()` → deactivates current coroutine and reactivates the **last resumer**.
- `resume()` → deactivates the current coroutine and activates the **target coroutine** (the one being resumed).

|Concept|Meaning|
|---|---|
|`this`|Currently executing coroutine|
|`uThisCoroutine`|Reference to current coroutine object|
|`last resumer`|The coroutine that last resumed this one|
|`starter`|The coroutine that first resumed this one|
> Note: `this` and `uThisCoroutine` can change at different times.

A **full coroutine** has three distinct phases:
1. **Start the cycle** — create coroutines and link them by `resume()` calls.
2. **Execute the cycle** — coroutines repeatedly resume each other.
3. **Stop the cycle** — eventually return control to `main`.

Starting requires each coroutine to **know at least one other coroutine**, creating circular references:
```c
Fc x, y;
x.partner(y);
y.partner(x);
x.resume();   // start cycle
```

![[Pasted image 20251027111056.png | 400]]

#### 3.11 Coroutine Languages
- Coroutines have two forms:
	1. stackless - use the caller's stack and a fixed-sized local state
		- cannot call other routines then suspend - only can suspend in the main
	2. stackful: separate stack and a fixed-size (class) local state
Simula, CLU, C#, Ruby, Python, JS, Lua, F#, all support yield constructs

*Note:* C++20 has an API for coroutines and outline code to build stackless, stackful, or even fibres




