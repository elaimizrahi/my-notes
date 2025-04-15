
## Closest Pairs

**Goal:**  
Given $n$ points $(x_i, y_i)$ in the plane, the task is to find a pair $(i, j)$ that minimizes the distance. The distance between two points is defined as  
$$
d_{i,j} = \sqrt{(x_i - x_j)^2 + (y_i - y_j)^2}
$$  
This is equivalent to minimizing the squared distance  
$$
d_{i,j}^2 = (x_i - x_j)^2 + (y_i - y_j)^2.
$$  
**Assumption:** All $x_i$’s are pairwise distinct.

---

## Divide and Conquer Approach

**Idea:**  
Separate the points into two halves based on the median $x$-value:
- Let $L$ be all $\frac{n}{2}$ points with $x \le x_{\text{median}}$.
- Let $R$ be all $\frac{n}{2}$ points with $x > x_{\text{median}}$.

The closest pair is either:
- Both points in $L$, or  
- Both points in $R$, or  
- A transverse pair (one point in $L$ and one in $R$).

---

## Finding the Shortest Transverse Distance

Define  
$$
\delta = \min(\delta_L, \delta_R),
$$  
where $\delta_L$ and $\delta_R$ are the smallest distances found in the left and right halves respectively.

For transverse pairs, we only need to consider pairs $(P, Q)$ such that:
- The distance from $P$ to the right half is at most $\delta$, and
- The distance from $Q$ to the left half is at most $\delta$.

For any point $P = (x_P, y_P)$, it suffices to consider only those points $Q$ with  
$$
y_P \le y < y_P + \delta.
$$  
Thus, it is enough to check the distances $d(P, Q)$ for $Q$ within this rectangle.

---

## Bounding the Number of Points in the Rectangle

**Claim:**  
There are at most 8 points from the initial set (including $P$) that can lie inside the rectangle.

**Proof Idea:**  
Cover the rectangle with 8 squares, each of side length $\delta/2$.  
- The squares on the left contain points solely from $L$.
- The squares on the right contain points solely from $R$.

As a consequence, each square can contain at most one point, thereby bounding the total by 8.

---

## Data Structures and Runtime Analysis

1. **Initialization:**  
   - Sort the points with respect to $x$.  
   - **Cost:** $O(n \log n)$ (performed before the recursive calls).

2. **Merging:**  
   - Merge sorted lists based on the $y$-coordinate to ensure the points remain ordered by $y$.

3. **Recursive Process:**  
   - Finding the $x$-median is achieved in $O(1)$ time.
   - For subsequent recursive calls:
     - Splitting the sorted lists takes $O(n)$ time.
     - Removing points at distance $\ge \delta$ from the splitting line takes $O(n)$ time.
     - Inspecting the remaining points (in increasing order of $y$) and computing the distance to the next 8 points takes $O(n)$ time.

**Overall Runtime:**  
$$
T(n) = 2T\left(\frac{n}{2}\right) + \Theta(n)
$$  
Thus, $T(n) \in \Theta(n \log n)$.

---

## Beyond the Master Theorem: Median of Medians

**Median Finding:**  
Given an array $A[0 \ldots n-1]$, the median is the entry that would appear at index $\lfloor n/2 \rfloor$ if $A$ were sorted.

**Selection Problem:**  
For an array $A[0 \ldots n-1]$ and an integer $k \in \{0, \dots, n-1\}$, the goal is to find the entry that would be at index $k$ after sorting.

**Known Results:**  
- Sorting $A$ requires $O(n \log n)$ time.
- A simple randomized algorithm can find the $k^{th}$ element in expected $O(n)$ time.

---

## The Selection Algorithm: Quick-Select

**Algorithm:** `quick-select(A, k)`

Given:
- An array $A$ of size $n$.
- A target index $k$ where $0 \le k < n$.

**Steps:**
1. Choose a pivot $p$ from $A$.
2. Partition $A$ around $p$ so that the pivot is placed at its correct index $i$.
3. If $i = k$, return $A[i]$.
4. If $i > k$, recursively apply quick-select on the subarray $A[0, \dots, i-1]$ with target index $k$.
5. If $i < k$, recursively apply quick-select on the subarray $A[i+1, \dots, n-1]$ with target index $k - i - 1$.

**Question:**  
How can we choose a pivot such that both $i$ and $n-i-1$ are not too large?

---

## Median of Medians Algorithm

**Sketch of the Algorithm:**
- **Divide:** Partition the array $A$ into $\frac{n}{5}$ groups $G_1, G_2, \dots, G_{\frac{n}{5}}$, each of size 5.
- **Conquer:** Find the median of each group, denoted $m_1, m_2, \dots, m_{\frac{n}{5}}$, in $O(n)$ time.
- **Select Pivot:** Use the median of these medians as the pivot $p$, which takes $T(n/5)$ time.

**Claim:**  
With this choice of $p$, both indices $i$ (number of elements less than or equal to $p$) and $n-i-1$ are at most $\frac{7n}{10}$.

---

## Proof for the Median of Medians

- At least half of the medians (i.e., at least $\frac{n}{10}$ of the groups) are greater than $p$.
- For each such median, there are at least 3 elements in its group that are greater than or equal to it.
- Consequently, there are at least $\frac{3n}{10}$ elements greater than $p$, so at most $\frac{7n}{10}$ elements are less than $p$.
- A similar argument applies for the elements greater than $p$.

**Runtime Recurrence:**  
$$
T(n) \le T\left(\frac{n}{5}\right) + T\left(\frac{7n}{10}\right) + O(n)
$$  
This recurrence yields $T(n) \in O(n)$.
