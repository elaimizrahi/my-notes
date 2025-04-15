## MergeSort and Recurrence Setup

- **MergeSort Design**
  1. Split array `A` into `A_L` (⌈n/2⌉) and `A_R` (⌊n/2⌋).
  2. Recursively apply MergeSort to `A_L` and `A_R`.
  3. Merge sorted halves.

- **Recurrence Relation**
  - Exact:
    ```
    T(n) = T(⌈n/2⌉) + T(⌊n/2⌋) + Θ(n), if n > 1
           Θ(1), if n = 1
    ```
  - Simplified (Exact):
    ```
    T(n) = T(⌈n/2⌉) + T(⌊n/2⌋) + cn
           d, if n = 1
    ```
  - **Sloppy Recurrence** (ignores floor/ceil):
    ```
    T(n) = 2T(n/2) + cn, if n > 1
           d, if n = 1
    ```

## 🌳 Recursion Tree Method

- Assume `n = 2^j`
- Steps:
  1. Start with root node `T(n)`
  2. Each node branches into two `T(n/2)`
  3. Replace nodes with their work: `cn`, `cn/2`, ...
  4. Stop at leaves: `T(1) = d`
  5. Sum across all levels for total work

- Total cost: $T(n) = cn + 2(cn/2) + 4(cn/4) + ... + n(c) = cn log n + dn ⟹ Θ(n log n)$

## 📐 Solving the Exact Recurrence

- When `n = 2^j`, recursion tree gives exact result
- If expressed with Θ-notation, gives asymptotic complexity for all `n`
- For formal proof, use induction (substitution method)

## 🧮 The Master Method

For recurrence of the form $T(n) = a T(n/b) + Θ(n^y)$
Let `x = log_b a`. Then:

- If `y < x`:  $T(n) ∈ Θ(n^x)$  "heavy leaves"
- If `y = x`:  $T(n) ∈ Θ(n^x log n)$ "balanced"
- If `y > x`:  $T(n) ∈ Θ(n^y)$ "heavy top"


## 🔁 Recursion Tree Analysis of Master Method

Let `n = b^j`, then:
- At level `i`:  
- Work per node: `c(n/b^i)^y`
- Number of nodes: `a^i`
- Total work: `c a^i (n/b^i)^y`

- Total cost: $T(n) = d * a^j + c * n^y * \sum_{i=0}^{j-1} (a/b^y)^i$

- Let `r = a / b^y = b^{x - y}`, then the sum becomes geometric:
- If `r < 1`: converges ⇒ Θ(n^y)
- If `r = 1`: ⇒ Θ(n^y log n)
- If `r > 1`: ⇒ Θ(n^x)

## 📏 Substitution (Induction) Method

1. Guess the solution form (e.g., `T(n) = O(n log n)`)
2. Prove with induction
3. Use constants if needed

**Example**: $T(n) = 2T(n/2) + n ⇒$ Guess: $T(n) ∈ O(n log n)$