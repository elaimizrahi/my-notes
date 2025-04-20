**Polynomial time reduction:** Given decision problems $A$ and $B$, with instance sets $I(A)$ and $I(B)$, a polynomial time reduction is a map $F: I(A) \implies I(B)$ such that $F(I_A)$ is a "yes" instance of $B$ if and only if $I_A$ is a "yes" instance of $A$ and there exists a polynomial time algorithm to compute $F$

**What does this mean?**
- If I give you an instance of a vertex-cover solution, and vertex-cover is a difficult problem (not computable in polynomial time), and you can convert to instance to an instance of an Independent set problem in polynomial time (we know independent set is NP), then we know vertex-cover is also an NP problem

**P**: The set of all decision problems for which a polynomial time solution exists
**NP**: the set of all decision problems for which a polynomial-time **verifier** exists
- A verifier takes an instance **and** a certificate, and checks if the certificate is evidence that the instance is a "yes" instance
	- For example, for vertex cover, an **instance** is the graph that was given, and a **certificate** is the set of edges that might be a vertex cover 
	- A "yes" instance is true (e.g. it **is** a vertex cover), otherwise it's a no-instance
	- A "no" instance doesn't imply that the verifier is false, could be that the certificate wasn't "strong" enough 

**NP Complete**: A decision problem $A$ is NP-Complete if
1. Show that $A \epsilon NP$ (A is in NP) 
2. A is the "hardest problem in NP" (for all $B \epsilon NP$, there is a polytime reduction from $B \implies A$) (that is there is a polynomial time reduction from an NP-complete problem to A) We denote this as $A \leq_{p} B$ if A is reducible to B



**Commonly known problems**:
- [Vertex Cover (VS)](https://en.wikipedia.org/wiki/Vertex_cover#:~:text=In%20graph%20theory%2C%20a%20vertex,every%20edge%20of%20the%20graph.)
- [3-SAT](https://en.wikipedia.org/wiki/Boolean_satisfiability_problem#3-satisfiability)
- [Independent Set (IS)](https://en.wikipedia.org/wiki/Independent_set_(graph_theory)#:~:text=In%20graph%20theory%2C%20an%20independent,no%20edge%20connecting%20the%20two.)