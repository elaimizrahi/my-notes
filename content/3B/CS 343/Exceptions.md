Chapter 4 discusses more exceptions
### 4.1 Derived Exception Types

**Goal:**  
Use inheritance to organize exceptions and enable polymorphic handling.

**Hierarchy example:**

```
Exception  
├── IO  │   
├── File│   
	└── Network  
	└── Arithmetic      
├── DivideByZero      
	└── Overflow
```

#### Key points:
- Enables **catching general exceptions** (e.g., `catch (Arithmetic)`) while allowing specific derived exceptions for details.
- Promotes **modularity**: high-level code can handle abstract errors; low-level code defines specifics.
- Catch order matters — **handlers are checked linearly**, first match wins.
- Derived types let a handler match both base and derived exceptions.
    

```
try {   ... } 
catch (Arithmetic &a) { ... }   
catch (Overflow &o) { ... }  // never reached if Arithmetic matched first
```

#### Exception slicing / truncation:
- In standard C++, a caught exception is **sliced** (truncated) to handler type.
- In **uC++**, dynamic casting avoids slicing (keeps actual type info).
`try { throw D(); } catch (B &e) { dynamic_cast<D&>(e); }  // works safely in uC++`

---

### 4.2 Catch-Any

**Catch-any** (`catch (...)` or `_CatchResume(...)`) handles any uncaught exception.
#### Uses:
- Logging, cleanup, recovery, or rethrow.
- Provides a general “safety net” when the specific type is unknown.

```try {   ... } 
_CatchResume(...) { logError(); _Resume; }   // resume for recovery   catch(...) { cleanup(); _Throw; }            // termination for recovery
```

**Java `finally`:**
`try { ... } catch (E e) { ... } finally { cleanup(); }`
- Always executes, even on exception exit.
- Handles non-exceptional cleanup (RAII substitute).
    
### 4.3 Exception Parameters

**Purpose:**  
Pass extra data from the raise site to the handler, allowing context-aware handling.
```
struct E {   
	int i;   
	E(int v) : i(v) {} 
};  
void f() { throw E(3); }  
try { f(); } 
catch (E &e) { cout << e.i; }  // access passed parameter
```

#### Notes:
- Parameters may be passed **by value, reference, or pointer**.
- Useful for customizing the handler’s behavior based on raise-site data.

### 4.4 Exception List

**Definition:**  
The list of exception types a routine can throw — part of its function signature.
`void f() throw(E1, E2);`

#### Purpose:
- Enables **static detection** of unhandled exceptions at compile time.
- Ensures functions document what exceptions they may raise.
#### Types:
1. **Checked exceptions** — explicitly listed, enforced (Java-style).
2. **Unchecked exceptions** — not listed (C++ `noexcept` replaces `throw()`).
    
`void g() noexcept(false) { throw E(); }   // may throw void h() noexcept(true)  { }              // guaranteed not to throw`

#### Implications:
- A derived function’s exception list must be **compatible** with its base class’s list.
- Checked exceptions can restrict generic code — reason why C++ removed `throw()` specs in favor of `noexcept`.
    

### 4.5 Destructor
**Destructors** are implicitly `noexcept` → **cannot raise exceptions**.
If a destructor raises an exception:
- It causes **stack unwinding within unwinding**, leading to termination.
- Exception from destructor during an active exception → program abort.
    
`struct C {   ~C() noexcept(false) { throw E(); } // dangerous! };`

**Example:**  
If object `y`’s destructor throws inside a `try` block that’s already unwinding for another exception:
- The new exception interrupts stack unwinding.
- The program terminates unless caught immediately.
    
### 4.6 Multiple Exceptions

**Concept:**  
Multiple exceptions can be **nested**, but not **propagate simultaneously**.

```
void f(int i) {   
	if (i == 0) throw E0();   
	try { f(i - 1); }   
	catch (E1) { if (cnt > 0) throw E2(); 
} // rethrow nests exception }
```

#### Rules:
- Only one active propagation at a time.
- A second exception cannot start while another is unwinding.
- Destructors may trigger new exceptions during cleanup, but only one propagates.
- Use `std::uncaught_exceptions()` to detect if unwinding is active.