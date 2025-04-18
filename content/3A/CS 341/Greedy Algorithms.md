These usually present themselves as **Optimization** problems - we want to maximize or minimize a certain objective function

**Features of Greedy Algorithms:**
1. No looking ahead and no backtracking
2. Usually only require a pre-processing step and a single pass through the data
3. Only one feasible solution is constructed
4. Correctness proofs can be tricky and complicated - but can prove that they always yield an optimal solution

**Feasible Solution**: For any problem instance $I$, `feasible(i)` is the set of all outputs for $I$ that satisfy the given constraints

**Choice Set**: For a partial solution $X = [x_1, ..., x_i]$ where $i < n$, we define the choice set
### Example: Making Change
**Instance**: A set $C$ of coin denominations from $C$ and a given amount $M$ 
**Find:** The minimum number of coins of denominations from $C$ that sum to $M$

### Interval Selection
**Instance:** A set $I = \{1, ... n\}$ of intervals where for $1 \leq i \leq n, i = [s_i, f_i)$ where $s_i$ is the start time and $f_i$ is the end time
**Find**: A subset of $S \subseteq I$ of pairwise disjoint intervals of maximum size $|S|$

**Some possible solutions:**
1. Select based on earliest start time
2. Select based on shortest length
3. Select based on fewest conflicts with other intervals
4. Select based on earliest finishing time

**Solution**: 
```python
# Sort intervals 1...n by finish time and relabel so f1 <= ... <= fn
S = {}
for i = 1 to n do:
	if (interval i is pairwise disjoint with all intervals in S):
		S = S union {i}
```

**Runtime:** $O(nlogn)$ to sort $+ O(n)$ loop $\implies O(nlogn)$

## Proving Greedy Algorithm Correctness
### 1. Greedy always stays ahead
Let $A = \{a_1, ... a_k \}$, sorted by finish time
Compare A to an optimum solution $B = \{b_1, ... b_k \}$ sorted by finish time. Thus $l \geq k$ and we want to prove $l = k$

**Idea**: at every step $i$ we can do at least as good by choosing $a_i$

**Solution: Use Induction!**
Base case: $i=1$
- $a_1$ had the earliest finish time of all, so $finish(a_1) \leq finish(b_1)$
- Thus, $a_1$ is disjoint from all $b_i$ for $2 \leq i \leq l$
- Thus we can replace $b_1$ with $a_1$

Inductive Step:  suppose $a_1, ... a_{i-1}, b_i, ... b_l$ is an optimal solution
- $b_i$ does not intersect $a_{i-1}$ so the greedy algorithm could have chose it, but it chose $a_i$ instead, so $finish(a_i) \leq finish(b_i)$
- $a_i$ is then also disjoint from all $b_k$ for $i + 1 \leq k \leq l$
- So we can replace $b_i$ with $a_i$
- This proves the claim. To finish proving, we argue that if $k < l$ then $a_1, ... a_{k}, b_{k+1}, ... b_l$ us optimal. But them the algorithm would have more choices after $a_k$

### 2. Exchange Proofs
General idea is to show how we can convert an optimal solution into the greedy solution
- Let $G$ be the solution produced by the greedy algorithm, and $O$ be an optimal solution
- If $G$ is the same as $O$ then greedy is also optimal. Otherwise, find a pair of items that are out of order in $O$ when compared with $G$ 
- Show that by **exchanging** the order of these two items, we create a new solution that is better (or at least no worse); that is the solution remains optimal 
	- The reasoning is typically based on how the algorithm makes its choice
- By making a number of exchanges, we will obtain the greedy solution (similar to bubblesort) and since each exchange makes the solution no worse, the greedy algorithm is also optimal


## Knapsack Problems
**Instance**: A set of items $1, ... n$ with values $v_1,...v_n$ and weights $w_1, ... w_n$ and a capacity $W$ 
**Find:** A feasible solution $X$ that maximizes $\sum_{i=1}^{n}{v_ix_i}$

**0-1 Knapsack:** We want to maximize the value of the items, and we can either **take or leave** an item
**Fractional Knapsack**: We want to maximize the value of the items, but we can take a fraction of items - if we take $2/3$ of an item, we also take $(2/3)v_i$ and $(2/3)w_i$ 

**Possible greedy strategies:**
1. Consider items in decreasing order of value
2. Consider items in increasing order of weight
3. Consider items in decreasing order of $value/weight$

**Solution**:
```python
# xi is the weight of item i taken
Sort items 1->n by value per weight and relabel so (v1/w1) => ... => (vn/wn)
freeWeight <- W
for i = 1 to n: 
	xi = min{wi, freeWeight}
	freeWeight -= xi
```

**Final Weight** is $\sum{x_i} = W$ if $\sum{w_i} \geq W$
**Runtime**: $O(nlogn)$ to sort, $O(n)$ to choose weights for each item


