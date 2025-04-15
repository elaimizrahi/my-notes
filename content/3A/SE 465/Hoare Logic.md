
**Hoare Triples** are how we write assertions about WHILE programs 

$\{A\}\ c\ \{B\}$

if $A$ holds in state $q$ and if $q \rightarrow q'$, then $B$ holds in $q'$

Hoare triples are *partial correctness assertions*, which means they do not imply the termination of c
- This is because $q'$ may not exist 

$[A]\ c\ [B]$ - this is how we denote *total correctness*, but it's much less used

**Total Correctness:** $q \models a$ is how we write that assertion $A$ holds in state $q$ (well defined when $q$ is defined on all variables occurring free in A)

**Example**  
$\{\, y \le x \}\; z := x;\; z := z + 1\; \{\, y < z \}$

**Inference**
we write $\vdash \{A\}\ c\ \{B\}$ when we can derive the triple from the inference rules
- there is one inference rule for each command in WHILE

### Pre/Post Conditions

**Weakening the precondition**:
- we generally want to give the weakest precondition - allows a larger amount of inputs
- **Example:** $\{x < 10\}\ x:=5\ \{x=5\} \rightarrow \{true\}\ x:=5\ \{x=5\}$

**Strengthening the postcondition:**
- we want to create the strongest postcondition so that we know what is true after the triple has completed
- **Example**: $\{true\}\ x:=5\ \{true\} \rightarrow \{true\}\ x:=5\ \{x=5\}$

