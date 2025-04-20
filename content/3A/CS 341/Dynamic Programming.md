Dynamic Programming encompasses a huge set of problems in computer science - and are difficult to get good at.

We want to split a problem up into subproblems, solving these and setting up recurrences that will let us compute a final value (and maybe recover it too!)

**Steps**:
1. Identify the subproblem
2. Establish the recurrence 
3. Set values for the base cases
4. Specify the order of computation
5. Recovery of the solution (if needed)

**Note:** in CS 341 we must use bottom-up dp solutions, specify runtime, and usually correctness follows from the above

**Common problems**:
- Interval Scheduling
- 0/1 knapsack
- longest increasing subsequence
- edit distance
- optimal BST's

Credit to [Kelly Guan](https://www.linkedin.com/in/kelly-guan/) for the notes below - it covers the templates for common dynamic programming problems and how we solve them

```python
# 1. 0/1 Knapsack

# Problem: choose items with weights and values to maximize value without exceeding capacity
# Variants: Subset Sum, Partial Equal Subset Sum, Target Sum, etc
# Recursive Step: dp[i][w] = max value using first i items and capacity w

def knapsack_01(I, W):
    n = len(I)
    dp = [[0 for _ in range(W + 1)] for _ in range(n + 1)]
    
    for i in range(1, n + 1):
        value, weight = I[i - 1]
        for j in range(1, W + 1):
            if j >= weight:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - weight] + value)
            else:
                dp[i][j] = dp[i - 1][j]
    
    return dp[n][W]


# 2. Unbounded Knapsack

# Problem: like 0/1 knapsack but can reuse items
# Variants: Coin Change, Integer Break, Rod Cutting
# Recursive State: dp[w] = max value achievable with weight w

def knapsack_unbound(I, W):
    n = len(I)
    dp = [[0 for _ in range(W + 1)] for _ in range(n + 1)]
    
    for i in range(1, n + 1):
        value, weight = I[i - 1]
        for j in range(1, W + 1):
            dp[i][j] = max(dp[i - 1][j], dp[i][j - weight] + value)
    
    return dp[n][W]


# 3. Linear DP / Fibonacci Sequence

# Problem: Simple recurrence relations like dp[i] = dp[i - 1] + dp[i - 2]
# Variants: Climbing Stairs, Tiling Problems, House Robber, Valid Word Checker
# Recursive State: 1D array

def house_robber(nums):
    """
    Each house has a certain amount of money stashed, but adjacent houses have security systems.
    Robbing two directly adjacent houses will alert the police.
    Return the max amount of money you can rob without alerting the police.

    dp[i] = max amount of money stolen by robbing up to the ith house.
    Base cases:
    dp[0] = 0 (no houses → no money)
    dp[1] = max(nums[0], nums[1])
    Recursive step:
    dp[i] = max(dp[i - 2] + nums[i], dp[i - 1])
    """
    n = len(nums)
    dp = [0 for _ in range(n + 1)]
    dp[0] = 0
    dp[1] = max(nums[0], nums[1])
    
    for i in range(2, n + 1):
        dp[i] = max(dp[i - 2] + nums[i], dp[i - 1])
    
    return dp[-1]


# 4. Longest Common Subsequence

# Problem: Find the length of the longest subsequence common to two sequences
# Variants: Edit Distance, Longest Palindromic Subsequence, Shortest Common Supersequence
# Recursive State: dp[i][j] = LCS of first i chars of A and first j chars of B

def LCS(A, B):
    n = len(A)
    m = len(B)
    dp = [[0 for _ in range(m + 1)] for _ in range(n + 1)]
    
    for i in range(1, n + 1):
        for j in range(1, m + 1):
            if A[i - 1] == B[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    
    # backtrack
    i, j = n, m
    res = []
    while i > 0 and j > 0:
        if A[i - 1] == B[j - 1]:
            res.append(A[i - 1])
            i -= 1
            j -= 1
        elif dp[i - 1][j] > dp[i][j - 1]:
            i -= 1
        else:
            j -= 1
    res.reverse()
    return dp[n][m], res


# 5. Palindrome DP

# Problem: Work on palindromic substrings
# Variants: Palindrome Partitioning, Longest Palindromic Substring
# Recursive State: dp[i][j] = whether or best result for substring s[i:j+1]

def longest_palindrome_substring(s):
    n = len(s)
    dp = [[False for _ in range(n)] for _ in range(n)]
    
    start, max_len = 0, 1
    
    for i in range(n):
        dp[i][i] = True  # Single character is a palindrome
    
    for i in range(n - 1):
        if s[i] == s[i + 1]:
            dp[i][i + 1] = True
            start = i
            max_len = 2
    
    for length in range(3, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j] and dp[i + 1][j - 1]:
                dp[i][j] = True
                if length > max_len:
                    start = i
                    max_len = length
    
    return s[start: start + max_len]


# 6. DP on Subsequences

# Problem: Consider subsets or subsequences of elements
# Variants: Count Increasing Subsequences, Subset Sum, LIS
# Recursive State: Often uses combinations or bitmasking

def longest_inc_subsequence(nums):
    n = len(nums)
    dp = [1 for _ in range(n)]
    
    for i in range(n):
        for j in range(i):
            if nums[i] > nums[j]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)

```