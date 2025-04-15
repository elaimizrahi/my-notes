### Syntax
**Atomic Formulas**
- **A, B, C, true, false** or any other variables 

Atomic formulas can be combined with formulas:
- $\land$ - and
- $\lor$ - or
- $\implies$ - implication
- $\iff$ - if and only if (iff)
- $\lnot$  - not

**Validity**: A formula is valid if and only if $\lnot F$ is unsatisfiable
If $F$ is valid:
- Every suitable assignment for $F$ is a model for $F$
- Every suitable assignment for $F$ is not a model for $\lnot F$
- $\lnot F$ does not have a model and is unsatisfiable

#### Negation Normal Form
Every occurrence of negation in F is applied to an atomic sub-formula of F
- $\lnot a \lor \lnot b \lor c$ is in NNF
-  $\lnot (a \land b \land \lnot c)$ is not in NNF 

#### Disjunctive Normal Form
a disjunction of conjunctions of literals
$$
F \;=\;\bigvee_{i=1}^{n}\,\Bigl(\,\bigwedge_{j=1}^{m_i} L_{i,j}\Bigr).
$$
**Note:** determining whether a DNF formula F is satisfiable is easy (easy == linear in the size of the formula)
#### Conjunctive Normal Form
A conjunction of disjunctions of literals

$$
F \;=\;\bigwedge_{i=1}^{n}\,\Bigl(\,\bigvee_{j=1}^{m_i} L_{i,j}\Bigr).
$$
**Note:** determining whether a CNF formula F is satisfiable is hard (hard == NP-complete)

**For every formula $F$ there is an equivalent formula $F_1$ in CNF and $F_2$ in DNF**


## First Order Logic

First order logic is like propositional logic, but we add the quantifiers $\exists$ (there exists) and $\forall$ (for all), uninterpreted functions, and predicates

##### Uninterpreted functions
if $f$ is an uninterpreted function, then it has an [**arity**](https://en.wikipedia.org/wiki/Arity) denoted by $g_{/2}, f_{/0}$ 
- This means we have just defined the signature - which might not always be satisfiable

In First Order logic, we then can use quantifiers and uninterpreted functions to prove logic 

**Example:** 
Someone who lived in Dreadbury Mansion killed Aunt Agatha. Agatha, the Butler and Charles were the only people who lived in Dreadbury Mansion. A killer always hates his victim, and is never richer than his victim. Charles hates no one that aunt Agatha hates. Agatha hates everyone except the Butler. The Butler hates everyone not richer than Aunt Agatha. The Butler also hates everyone Agatha hates. No one hates everyone. Agatha is not the Butler.

![[Pasted image 20250309213407.png]]


### Models
A Model **M** is defined as: 
![[Pasted image 20250309213448.png]]

### Equality
- **Reflexivity:** For all $x$, $R(x, x)$
- **Transitivity:** For all $x, y, z$, if $R(x, y)$ and $R(y, z)$, then $R(x, z)$
- **Symmetry:** For all $x, y$, if $R(x, y)$, then $R(y, x)$.


References:
https://link.springer.com/book/10.1007/978-0-8176-4763-6
https://link.springer.com/book/10.1007/978-3-540-74113-8
