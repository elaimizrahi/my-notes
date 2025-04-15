
## What is Divide and Conquer?

Divide-and-conquer is a general algorithmic paradigm:

- **Divide**: Split the problem into subproblems  
- **Conquer**: Recursively solve each subproblem  
- **Combine**: Merge the subproblem results into a final solution  

Use it when:

- The problem decomposes cleanly (no overlapping subproblems)  
- Combining subproblem results is efficient  
- Subproblems are balanced in size  

---

## Counting Inversions

**Goal**: Given an array $A[1..n]$, count the number of inversions.

**Definition**: $(i, j)$ is an inversion if $i < j$ and $A[i] > A[j]$.

**Example**:  
For $A = [1, 5, 2, 6, 3, 8, 7, 4]$, the inversions are:  
$(2, 3), (2, 5), (2, 8), (4, 5), (4, 8), (6, 7), (6, 8), (7, 8)$

Naive solution:  
Two nested loops gives $\Theta(n^2)$ time.

---

## Divide-and-Conquer Strategy

Let $n$ be a power of 2.

- $c_\ell$ = inversions in $A[1..n/2]$  
- $c_r$ = inversions in $A[n/2+1..n]$  
- $c_t$ = transverse inversions, where $i \leq n/2 < j$ and $A[i] > A[j]$

Total:

$$
T(n) = c_\ell + c_r + c_t
$$

We compute $c_\ell$ and $c_r$ recursively.  
How to compute $c_t$?

---

## Option 1: Binary Search in Right Half

- For each $i \leq n/2$, binary search $A[i]$ in the right half.
- Each binary search: $O(\log n)$ → Total: $O(n \log n)$
- Sorting: $O(n \log n)$

Recurrence:

$$
T(n) \leq 2T(n/2) + O(n \log n)
$$

Result:  
$T(n) \in O(n \log^2 n)$

---

## Option 2: Enhanced MergeSort

During merge step, count transverse inversions:

- If $S[i] < S[j]$ → no inversion  
- If $S[i] > S[j]$ → all remaining $S[i..]$ are greater  

Still linear merge → overall time:  
$T(n) = \Theta(n \log n)$

---

## Polynomial Multiplication

Given:

$F(x) = f_0 + f_1x + \cdots + f_{n-1}x^{n-1}$  
$G(x) = g_0 + g_1x + \cdots + g_{n-1}x^{n-1}$

Want:

$H(x) = F(x)G(x) = \sum_{i=0}^{2n-2} h_i x^i$

Naive solution: nested loops → $\Theta(n^2)$ time.

---

## Divide-and-Conquer Polynomial Multiplication

Split:

$F = F_0 + F_1 x^{n/2}$  
$G = G_0 + G_1 x^{n/2}$

Then:

$$
FG = F_0G_0 + (F_0G_1 + F_1G_0)x^{n/2} + F_1G_1 x^n
$$

Recurrence:

$$
T(n) = 4T(n/2) + \Theta(n) \Rightarrow T(n) = \Theta(n^2)
$$

No improvement over naive.

---

## Karatsuba’s Algorithm

Key identity:

$$
(F_0 + F_1 x^{n/2})(G_0 + G_1 x^{n/2}) = F_0G_0 + ((F_0 + F_1)(G_0 + G_1) - F_0G_0 - F_1G_1)x^{n/2} + F_1G_1 x^n
$$

Only 3 recursive calls:

- $F_0G_0$
- $F_1G_1$
- $(F_0 + F_1)(G_0 + G_1)$

Recurrence:

$$
T(n) = 3T(n/2) + \Theta(n) \Rightarrow T(n) = \Theta(n^{\log_2 3}) \approx \Theta(n^{1.58})
$$

---

## Toom-Cook and FFT

### Toom-Cook

- Generalizes Karatsuba's algorithm
- $T(n) = \Theta(n^{\log_k(2k-1)})$  
- Gets closer to linear, but slowly

### FFT

- Over complex numbers  
- Recurrence: $T(n) = 2T(n/2) + \Theta(n)$  
- Multiplication in $\Theta(n \log n)$

---

## 🧮 Matrix Multiplication

Classic method: $T(n) = \Theta(n^3)$

Divide-and-conquer with 8 recursive calls:

$$
T(n) = 8T(n/2) + \Theta(n^2) \Rightarrow T(n) = \Theta(n^3)
$$

---

## 🔥 Strassen’s Algorithm

Compute 7 products instead of 8:

$$
T(n) = 7T(n/2) + \Theta(n^2) \Rightarrow \Theta(n^{\log_2 7}) \approx \Theta(n^{2.81})
$$

---

## 📈 What This Means

### General Rule:

If an algorithm uses $k$ multiplications on $\ell \times \ell$ matrices:

$$
T(n) = \Theta(n^{\log_\ell k})
$$

### Best Known:

- Current best exponent: $O(n^{2.37188})$  
- These are "galactic" algorithms — not practical yet
