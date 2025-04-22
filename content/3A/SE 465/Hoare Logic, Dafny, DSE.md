---

# **Full Summary with Examples (L09-L18)**

---
# Hoare Logic Rules

## Basic Rules

### 1. **Assignment Rule (ASGN)**

> After assigning `x := e`, to ensure postcondition `Q`,  
> you must have `Q` true **after substituting** `e` for `x`.

$$
\{ Q[e/x] \} \; x := e \; \{ Q \}
$$

- **Q\[e/x\]** means: replace all occurrences of `x` in `Q` by `e`.
- Used **before** the assignment to check correctness **after** the assignment.

---

### 2. **Consequence Rule (CONS)**

> If you have a stronger precondition or a weaker postcondition,  
> you can adjust your triple.

$$
\text{If } P' \Rightarrow P \quad \text{and} \quad Q \Rightarrow Q', \quad \text{then}
$$

$$
\{ P' \} \; C \; \{ Q' \}
$$

where \( \{ P \} \; C \; \{ Q \} \) is the original Hoare triple.

- **Strengthen preconditions**: assume more.
- **Weaken postconditions**: guarantee less.

---

### 3. **Skip Rule (SKIP)**

> Doing nothing keeps everything the same.

$$
\{ P \} \; \text{skip} \; \{ P \}
$$

---

### 4. **Composition Rule (SEQ)**

> If `C1` establishes a condition needed for `C2`, you can compose them.

If

$$
\{ P \} \; C_1 \; \{ Q \} \quad \text{and} \quad \{ Q \} \; C_2 \; \{ R \}
$$

then

$$
\{ P \} \; C_1;C_2 \; \{ R \}
$$

- The postcondition of `C1` must match the precondition of `C2`.

---

## Rules for Conditionals and Loops

### 5. **If-Then-Else Rule (IF)**

> To verify an `if` statement, verify both branches separately under the proper conditions.

$$
\text{If} \quad \{ P \land B \} \; C_1 \; \{ Q \} \quad \text{and} \quad \{ P \land \neg B \} \; C_2 \; \{ Q \}
$$

then

$$
\{ P \} \; \text{if } B \text{ then } C_1 \text{ else } C_2 \; \{ Q \}
$$

- Both branches must ensure the same postcondition \( Q \).

---

### 6. **While Rule (WHILE)**

> If you have a loop invariant `I`, and it’s preserved through each iteration, then the loop is correct.

If

- \( \{ I \land B \} \; C \; \{ I \} \)
- \( I \) holds before the loop starts

then

$$
\{ I \} \; \text{while } (B) \; \text{do } C \; \{ I \land \neg B \}
$$

- After loop ends (when \( B \) is false), \( I \land \neg B \) should imply the desired postcondition.

---

## Special Notes

- Invariants for loops must be strong enough to establish the postcondition when the loop terminates.
- In proofs, it’s common to chain together **ASGN + CONS** quickly to move between states.
- Reasoning about loops is the hardest part; choosing a good invariant is key.

---
# Dafny Cheat Sheet (Syntax + Exam Strategy)

## Basic Syntax

### Program Structure

```dafny
method Name(params) returns (results)
  requires Precondition;
  ensures Postcondition;
{
  body
}
```

- `requires`: what must be true before the method runs.
- `ensures`: what will be true after the method finishes.

---

### Assertions and Assumptions

```dafny
assert P;
assume P;
```

- `assert`: must prove `P` holds at that point.
- `assume`: tell Dafny to *pretend* `P` is true.

---

### Loops

```dafny
while (condition)
  invariant Invariant;
  decreases Metric;
{
  body
}
```

- `invariant`: must hold **before and after** each loop iteration.
- `decreases`: must **decrease** every iteration to prove termination.

---

### Arrays

```dafny
var a: array<int> := new int[5];
a[i] := value;
```

- Use `.Length` to access array size.
- Index starts at 0.

---

### Ghost variables

```dafny
ghost var x: int;
```

- Ghost variables are for verification only, not runtime.

---

## Common Exam Problems (In-Depth)

### 1. Method Verification

**Task:**  
Prove that a method is correct based on its precondition (`requires`) and postcondition (`ensures`).

**What you do:**
- Write correct `requires` (what assumptions you need).
- Write `ensures` that match exactly what the method is supposed to guarantee.
- In the method body, add `assert` if Dafny struggles to verify something.
- If needed, `assume` only when you know something is true but Dafny can't figure it out easily.

**Typical issues:**
- Forgetting to assert something obvious (e.g., array is non-empty).
- Writing an `ensures` that isn't strong enough or is too strong to be provable.

**Example:**

```dafny
method Increment(x: int) returns (y: int)
  ensures y == x + 1
{
  y := x + 1;
}
```

**Notes:**
- Very short methods can often be verified just by matching the math.
- Always re-read your `ensures` as a **claim**: "After running this method, is this statement true?"

---

### 2. Loop Invariants

**Task:**  
Come up with a correct invariant to prove a while loop is safe and correct.

**What you do:**
- State facts that should always be true when entering and exiting the loop.
- Include:
  - **Progress**: e.g., `0 <= i <= n`
  - **What you are building**: e.g., `sum = 1 + 2 + ... + i`
  - **Any important side properties**: e.g., `array remains sorted`

**Typical issues:**
- Forgetting an invariant about bounds (e.g., `i <= n`).
- Writing an invariant that isn't true initially.

**Example:**

```dafny
method Factorial(n: nat) returns (f: nat)
  ensures f == if n == 0 then 1 else n * (n-1)!
{
  f := 1;
  var i := 1;
  while (i <= n)
    invariant 1 <= i <= n+1
    invariant f == (i-1)!
    decreases n - i + 1
  {
    f := f * i;
    i := i + 1;
  }
}
```

**Notes:**
- Always check: Is the invariant true **before** the loop starts? **After** each loop iteration?
- If an invariant talks about values like `i`, make sure to control the range tightly.

---

### 3. Termination Proofs

**Task:**  
Show that a loop eventually stops (proving it won’t run forever).

**What you do:**
- Add a `decreases` clause.
- The `decreases` expression must go down (strictly decrease) every iteration.
- It must stay non-negative and finite.

**Typical choices:**
- `n - i` if `i` is moving up toward `n`
- `array.Length - i`
- `someCounter` decreasing

**Example:**

```dafny
method Countdown(n: nat)
{
  var i := n;
  while (i > 0)
    invariant 0 <= i <= n
    decreases i
  {
    i := i - 1;
  }
}
```

**Typical issues:**
- Writing a decreases term that can get stuck (e.g., not guaranteed to go down).
- Forgetting decreases in nested loops.

**Notes:**
- If a loop modifies two variables, you might need a tuple: `decreases (x, y)`

---

### 4. Ghost Variables for Proofs

**Task:**  
Use ghost variables to help state or prove properties without affecting real code.

**What you do:**
- Declare a ghost variable at the start.
- Update it manually inside your loop/method to track hidden information.
- Use ghost variables in `invariant`, `assert`, etc.

**Example:**

```dafny
method SumGhost(n: nat) returns (s: nat)
  ensures s == n*(n+1)/2
{
  ghost var realSum := 0;
  var i := 0;
  s := 0;
  while (i <= n)
    invariant 0 <= i <= n+1
    invariant realSum == i*(i-1)/2
    invariant s == realSum
    decreases n - i
  {
    s := s + i;
    ghost realSum := realSum + i;
    i := i + 1;
  }
}
```

**Notes:**
- Ghost variables must be consistent with real variables if your invariant depends on both.
- Ghost code disappears at runtime, so it doesn't slow down the program.

---

### 5. Array Reasoning and Quantifiers

**Task:**  
Prove statements about *every element* of an array (or about existence of one).

**What you do:**
- Use `forall` to express "for every index" properties.
- Use `exists` to express "there is some index" properties.
- Always be careful with bounds (`0 <= i < array.Length`).

**Typical statements:**
- "Array is sorted":
  ```dafny
  forall i, j :: 0 <= i < j < a.Length ==> a[i] <= a[j]
  ```
- "All elements are positive":
  ```dafny
  forall i :: 0 <= i < a.Length ==> a[i] > 0
  ```

**Example: Max element verification**

```dafny
method FindMax(a: array<int>) returns (m: int)
  requires a.Length > 0
  ensures forall i :: 0 <= i < a.Length ==> a[i] <= m
{
  m := a[0];
  var i := 0;
  while (i < a.Length)
    invariant 0 <= i <= a.Length
    invariant forall j :: 0 <= j < i ==> a[j] <= m
    decreases a.Length - i
  {
    if a[i] > m {
      m := a[i];
    }
    i := i + 1;
  }
}
```

**Notes:**
- Be precise: `forall i :: (bounds) ==> (property)`.
- If needed, split bounds checking and property checking into two different assertions.

---

# Bubble Sort (Correct and Verified) in Dafny

```dafny
predicate sorted(a: array<int>, from: int, to: int)
  requires 0 <= from <= to <= a.Length
  reads a
{
  forall i, j :: from <= i <= j < to ==> a[i] <= a[j]
}

predicate pivot(a: array<int>, to: int, pvt: int)
  requires 0 <= pvt <= to <= a.Length
  reads a
{
  forall u, v :: 0 <= u < pvt < v < to ==> a[u] <= a[v]
}

method bubbleSort(a: array<int>)
  requires a.Length > 0
  ensures sorted(a, 0, a.Length)
  ensures multiset(a[..]) == multiset(old(a[..]))
  modifies a
{
  var i: nat := 1;

  while i < a.Length
    invariant 1 <= i <= a.Length
    invariant sorted(a, 0, i)
    invariant multiset(a[..]) == multiset(old(a[..]))
  {
    var j: nat := i;
    while j > 0
      invariant 0 <= j <= i
      invariant sorted(a, 0, j)
      invariant pivot(a, i + 1, j)
      invariant sorted(a, j, i + 1)
      invariant multiset(a[..]) == multiset(old(a[..]))
    {
      if a[j - 1] > a[j]
      {
        var temp := a[j - 1];
        a[j - 1] := a[j];
        a[j] := temp;
      }
      j := j - 1;
    }
    i := i + 1;
  }
}
```

---

# Key things to notice


✅ **Invariants**:

- Outer loop:
  - `sorted(a, 0, i)` → first `i` elements are sorted
  - multiset preserved → no elements lost or duplicated

- Inner loop:
  - `[0, j)` sorted
  - `[j, i+1)` sorted
  - `pivot(a, i+1, j)` ensures everything before `j` ≤ everything after `j`
  - multiset preserved

✅ **Postconditions**:

- Final array sorted
- Elements preserved (permutation via multiset equality)
# Dynamic Symbolic Execution (DSE)

Dynamic Symbolic Execution (DSE) combines **symbolic execution** and **concrete execution** to systematically explore program paths and find bugs.

Instead of running a program on fixed inputs, DSE explores *all possible paths* by building **path conditions** and solving them.

---

## How it works

1. **Start with concrete inputs**: run the program normally.
2. **At each branch** (e.g., `if (x > 0)`):
   - Record the **path constraint** taken (e.g., `x > 0`).
3. **After execution finishes**:
   - Flip one branch condition to **explore a new path** (e.g., `x <= 0`).
   - Use a **SMT solver** to generate new inputs satisfying the flipped path constraint.
4. **Repeat**:
   - Re-run program with new input.
   - Collect new paths.
   - Systematically explore all feasible paths.

---

## Key Concepts

- **Concrete values**: actual numbers like `x = 5`.
- **Symbolic values**: variables like `x` representing any possible value.
- **Path condition (PC)**: conjunction of all branch decisions taken so far.
- **Constraint solver**: tool (e.g., Z3) that finds inputs satisfying path conditions.
- **Bug detection**: asserts, crashes, divide-by-zero, etc. detected automatically.

---

## Advantages

- **Systematic path exploration**: finds corner cases easily.
- **Automatic test generation**: tests created from solving constraints.
- **No need to manually write test cases**.

---

## Challenges

- **Path explosion**: too many paths as program size grows.
- **Complex constraints**: solver may not handle all real-world constraints efficiently.
- **External calls**: if program interacts with environment, symbolic execution can get stuck.

---

## EXE (EXecutable Executable Analysis)

> A DSE tool designed for **bug finding** in C programs.

**Features:**
- Works at **binary level**: does symbolic execution at LLVM IR level.
- Finds **memory errors**: out-of-bounds, use-after-free, double free, etc.
- Uses **constraint solvers** to automatically generate inputs causing crashes.

**How EXE works:**
- Tracks symbolic memory.
- When memory safety properties are violated under symbolic conditions, flags errors.
- Solves for real inputs that trigger bugs.

**Limitations:**
- Scalability issues (path explosion).
- Struggles with complex system interactions (e.g., network or file IO).

---

## DART (Directed Automated Random Testing)

> Early DSE tool focusing on **path coverage**.

**Key idea:**
- Mixes **symbolic execution** and **random testing**.
- Executes program concretely first, records constraints at branches.
- Mutates inputs to drive execution down different paths.

**Main steps:**
1. Randomly pick an input.
2. Run the program concretely.
3. Log all branch decisions (symbolic path).
4. Systematically flip branch conditions and generate new inputs.
5. Explore until time limit/path limit.

**Strengths:**
- **Simple to implement**.
- **High coverage** in practice for many small programs.
- Can find **deep bugs** unreachable by pure random testing.

**Weaknesses:**
- Still suffers from **path explosion**.
- Constraint solving becomes bottleneck.

---

## DSE vs Pure Symbolic Execution

| Feature | DSE (Dynamic Symbolic Execution) | Pure Symbolic Execution |
|:--------|:---------------------------------|:-------------------------|
| Inputs | Starts with concrete, hybridizes | Fully symbolic from start |
| Exploration | Step-by-step through paths | Symbolic tree exploration |
| Strength | Scales better initially | Finds general bugs without fixed inputs |
| Weakness | Path explosion still hard | Scalability even worse |

---

# Quick Example Walkthrough (Simple Program)

```c
int foo(int x) {
  if (x > 0) {
    if (x < 10) {
      error();
    }
  }
  return 0;
}
```

- Initial input: `x = 5`
  - Path: `x > 0 && x < 10`
  - Hits `error()`.
- DSE records path constraint: `(x > 0) && (x < 10)`.
- Can explore other paths by solving:
  - Flip `x < 10` → `x >= 10`
  - Flip `x > 0` → `x <= 0`

New inputs:
- `x = 15` explores `(x > 0 && x >= 10)`.
- `x = -1` explores `(x <= 0)`.

DSE will eventually cover **all 3 paths**.

---

# Final DSE Summary

| Term | Meaning |
|:-----|:--------|
| Path condition (PC) | Collected branch conditions during execution |
| SMT solver | Tool that finds inputs satisfying conditions |
| Path explosion | Too many paths to explore exhaustively |
| Concrete + symbolic | Core DSE strategy (dynamic + symbolic) |
| EXE | Binary-level DSE tool for finding memory bugs |
| DART | Early DSE tool using random testing and symbolic paths |

---
# Kani vs Dafny — Quick Comparison

| Feature                | **Kani**                                      | **Dafny**                                     |
|------------------------|-----------------------------------------------|-----------------------------------------------|
| **Language**           | Rust-based (runs on Rust programs)            | Own language (imperative + functional mix)    |
| **Main Use Case**      | Bounded model checking of Rust code           | Formal verification from specs to code        |
| **Verification Style** | Model checking (symbolic execution + SAT)     | Deductive verification using SMT solvers      |
| **Tool Type**          | Bounded verifier (loop unrolling, etc.)       | Proof-based verifier                          |
| **Loops**              | Automatically unrolled (needs `--unwind`)     | Must manually specify invariants + decreases  |
| **Specs**              | Uses `#[requires]`, `#[ensures]` annotations  | `requires`, `ensures`, `invariant`, etc.      |
| **Memory Safety**      | Checks Rust safety + UB (like MIRI)           | Dafny is memory-safe by design (no unsafe)    |
| **Assertions**         | Uses `assert!()` macro                        | Uses `assert` keyword                         |
| **Target Audience**    | Systems-level / embedded Rust devs            | Formal methods, education, theorem proving    |
| **SMT Solver**         | Uses CBMC internally                          | Uses Z3 SMT solver                            |
| **Output**             | Counterexamples (if property fails)           | Proof obligations + verification success      |
| **Strengths**          | Works directly on Rust, checks real bugs      | Very strong specification & invariant support |
| **Weaknesses**         | Limited by bounds (false negatives possible)  | Can require more annotation effort            |

---

## When to Use

- ✅ **Use Kani** when:
  - You want to verify **Rust code** directly.
  - You care about **memory safety**, **assertions**, or **runtime errors**.
  - You're okay with **bounded verification** (up to fixed loop depth).

- ✅ **Use Dafny** when:
  - You want full formal **proofs** with loop invariants.
  - You're learning verification concepts like **Hoare logic**, **contracts**, etc.
  - You need to prove **mathematical properties** about algorithms.

---

## TL;DR

- **Kani** = Rust-focused, fast, bounded model checker.
- **Dafny** = Spec-first, mathematically rigorous verification language.

---
