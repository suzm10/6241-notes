

- high-level constructs can introduce substantial run-time overhead if we naively translate each construct independently into machine code
- local code optimization: code improvement within a basic block
- global code optimization: improvements take into account what happens across basic blocks
- most global optimizations based on data-flow analyses (algos to gather info about a program)
- data-flow analysis: for each instruction in the program, specify some property that must hold each time instruction is executed
# 9.1 The Principle Sources of Optimization

compiler only knows how to apply low-level semantic transformations using general facts like algebraic identities and performing the same operations on the same values = same result

## 9.1.1 Causes of Redundancy

compiler can eliminate redundancies


## 9.1.2 A Running Example: Quicksort


## 9.1.3 Semantics-Preserving Transformations

function/sematics-preserving transformations:
- common-subexpression elimination
	- remove recalculations (some can't be avoided because they're below the level of detaila accessible by programmer)
- copy propagation
- dead-code elimination
- constant folding

## 9.1.4 Global Common Subexpressions

- E = common sub-expression if E was previously computed + the values of the variables in E haven't changed since the previous computation

## 9.1.5 Copy Propagation

- use v for u after the copy statement u = v

## 9.1.6 Dead-Code Elimination

dead code: statements that compute values that never get used

constant folding: deducing at compile time that the value of an expression is a constant + using the constant instead

copy propagation turns copy statements into dead code

## 9.1.7 Code Motion

code motion: decreases amount of code in a loop by moving loop-invariant computations before the loop

## 9.1.8 Induction Variables and Reduction in Strength

strength reduction: replacing expensive operation (multiplication) with a cheaper one (addition)

a variable x is an induction variable if there's a positive or negative constant c such that each time x is assigned, it's value increases by c


## 9.2 Intro to Data-Flow Analysis

data-flow analysis: techniques that derive info about flow of data along program execution paths

to implement global common subexpression elimination: determine if 2 textually identical expressions evaluate to the same value along any possible execution path of the program

## 9.2.1 The Data-Flow Abstraction

execution path from point $p_1$ to point $p_n$: sequence of points $p_1, p_2, \dots, p_n$, such that for each $i=1, 2, \dots, n-1$ either:
1. $p_i$ is the point immediately preceding a statement and $p_{i+1}$ is the point immediately following that same statement
2. $p_i$ is the end of some block and $p_{i+1}$ is the beginning of some successor block
reaching definitions: definitions that may reach a program point along some path

for constant folding: find definitions that are the unique definition of their variable to reach a given program point, no matter which execution path is taken


## 9.2.2 The Data-Flow Analysis Schema

set of all possible data-flow values = domain for application

$IN[S]$ = data-flow values before each statement
$OUT[S]$ = data-flow values after each statement

data-flow problem: find a solution to a set of constraints on the $IN[S]$'s and $OUT[S]$'s for all statements S.

two sets of constraints:
- those based on the semantics of statement
- those based on the flow of control

### Transfer Functions

- relationship between data-flow values before + after the assignment statement

have two types:
- information may propagate forward along execution paths
	- $OUT[s] = f_s[IN[s]]$
- may flow backward up execution paths
	- $IN[s] = f_s[OUT[s]]$

### Control-Flow Constrains

Within a basic block consisting of statements $s_1, s_2, \dots, s_n$, control-flow is simple:
- $IN[s_{i+1}] = OUT[s_i]$

## 9.2.3 Data-Flow Schemas on Basic Blocks

- if $s_1$ is the first statement of block B, then $IN[B] = IN[s_1]$
- if $s_n$ is the last statement of block B, then $OUT[B] = OUT[s_n]$
- transfer function of basic block B, $f_B$, is derived by composing transfer functions of the statements in the block
	- $f_B = f_{s_n} \circ \dots \circ f_{s_1}$
- relationship between beginning + end of the block:
	- $OUT[B] = f_B[IN[B]]$
- if data-flow values are information about the sets of constants that may be assigned to a variable, then we have a forward-flow problem:
	- $IN[B] = \cup_{\text{P a predecessor of B}} OUT[P]$
- when data-flow is backwards, IN + OUT are reversed:
	- $IN[B] = f_B(OUT[B])$
	- $OUT[B] = \cup_{\text{S a successor of B}} IN[S]$
- find the most precise solution that satisfies the two sets of constraints: control-flow + transfer constrains

## 9.2.4 Reaching Definitions

a definition d reaches a point p if there's a path from the point immediately following d to p such that d isn't killed along the path

kill a definition of a variable x if there's any other definition of x along the path

program analysis must be conservative: if we don't know whether a statement s is assigning a value to x, we must assume that it may assign to it

aliases make it hard to tell if a statement is referring to particular variable x, so we assume we're dealing with variables that have no aliases

we assume all edges in a flow graph can be traversed which is conservative, not always true

### Transfer Equations for Reaching Definitions

consider definition $d: u = v + w$

definition generates a definition $d$ of variable $u$ + kills all other definitions in the program that define $u$ while leaving remaining incoming definitions unaffected

$f_d(x) = gen_d \cup (x - kill_d)$

$gen_d = \{d\}$
$kill_d =$ set of all other definitions of $u$ in the program

suppose block B has $n$ statements with transfer functions $f_i(x) = gen_i \cup (x - kill_i)$ for $i=1, 2, \dots, n$, then the transfer function for a block B may be written as:

$f_B(x) = gen_B \cup (x - kill_B)$

where $kill_B = kill_1 \cup \dots \cup kill_n$
and $gen_B = gen_n \cup (gen_{n-1} - kill_n) \cup (gen_{n-2} - kill_{n-1} - kill_n) \cup \dots \cup (gen_1 - kill_2 - \dots - kill_n)$


gen set contains all downward-exposed definitions: definitions inside the block that are visible immediately after the block
- definition is downwards exposed in a basic block only if it's not killed by a subsequent definition to the same variable inside the same basic block

### Control-Flow Equations

$IN[B] = \cup_{\text{P a predecessor of B}} OUT[P]$

union = meet operator for reaching definitions

use meet operator to create a summary of the contributions from different paths at the confluence of those paths

### Iterative Algorithms for Reaching Definitions

reaching definitions problem is defined by the following equations:

$OUT[ENTRY] = \emptyset$
$OUT[B] = gen_B \cup (IN[B] - KILL[B])$
$IN[B] = \cup_{\text{P a predecessor of B}} OUT[P]$

result of algorithm = least fixedpoint of the equations: solution whose assigned values to the INs and OUTs is contained in the corresponding values for any other solution to the equations

algorithm: reaching definitions
- input: flow graph for which $kill_B$ and $gen_B$ have been computed for each block B
- output: $IN[B]$ and $OUT[B]$, the set of definitions reaching the entry + exit of each block B in the flow graph
- method: start with an estimate $OUT[B] = \emptyset$ for all $B$ and iterate until IN and OUT reach convergence

iterative algo to compute reaching definitions
```
OUT[ENTRY] = empty set
for (each basic block B other than ENTRY) OUT[B] = empty set
while (changes to any OUT occur)
	for (each basic block B other than ENTRY) {
		IN[B] = U_{P a predeccesor of B} OUT[P]
		OUT[B] = gen_B U (IN[B] - KILL[B])
	}
```
## 9.2.5 Live-Variable Analysis

live-variable analysis: know whether for variable x and point p whether the value of x at p could be used along some path in the flow graph starting at p

- $def_B$ = set of variables defined in B prior to any use of that variable in B
- $use_B$ = set of variables whose values may be used in B prior to any definition of the variable

- exit condition: no variables are live on exit from the program
	$IN[EXIT] = \emptyset$
- a variable is live coming into a block if it's used before redefinition in the block or it's live coming out of the block + is not redefined in the block
	$IN[B] = use_b \cup (OUT[B] - def_B)$
- a variable is live coming out of a block iff it's live coming into one of its successors
	$OUT[B] = \cup_{\text{S a successor of B}} IN[S]$

both liveness + reaching definitions equations have:
- union as the meet operator because we only care about whether any path with desired properties exists

info flow for liveness travels backwards, opposite to the direction of control flow b/c we want to make sure that the use of a var x at point p is transmitted to all points prior to p in an execution path, so that at prior points, we know x will have its value used

iterative algo to compute live variables:
```
IN[EXIT] = empty set
for (each basic block B other than EXIT) IN[B] = empty set
while (changes to any IN occur)
	for (each basic block B other than EXIT) {
		OUT[B] = U_{S a successor of B} IN[S]
		IN[B] = use_B U (OUT[B] - def_B)
	}
```

## 9.2.6 Available Expressions

- an expression $x + y$ is available at point p if every path from entry node to p evaluates $x + y$, and after the last such evaluation prior to reaching $p$, there are no assignments to $x$ or $y$
- a block kills expression $x + y$ if it assigns or may assign $x$ or $y$ + doesn't subsequently recompute $x + y$
- a block generates $x + y$ if it definitely evaluates $x + y$ and doesn't subsequently define $x$ or $y$
![[Pasted image 20260223115454.png]]

compute set of generated expressions for each point in a block:
- at point p, set S of expressions is available
- q is the point after p with statement $x = y+z$ between them
- form the set of expressions available at q by:
	- add to S the expression $y + z$
	- delete from S any expression involving $x$

set of all killed expressions: all expressions $y + z$ such that either $y$ or $z$ is defined in the block, and $y + x$ is not generated in the block

$U$ = universal set of all expressions appearing on the right of one or more statements of the program
$IN[B]$ = set of expressions in $U$ that are available just before the beginning of $B$
$OUT[B]$ = the same for the point just after the end of $B$

$e\_gen_B$ = expressions generated by B
$e\_kill_B$ = set of expressions in U killed in B

equations:
$OUT[ENTRY] = \emptyset$
$OUT[B] = e\_gen_B \cup (IN[B] - e\_kill_B)$
$IN[B] = \cap_{\text{P a predecessor of B}} OUT[P]$

- equations are almost identical to reaching definition except the meet operator = intersection b/c an expression is only available at the beginning of a block if it's available at the end of all its predecessors

algo: iterative algo to compute available expressions

input: a flow graph with $e\_kill_B$ and $e\_gen_B$ computed for each block B. initial block = $B_1$
output: $IN[B]$ and $OUT[B]$, the set of expressions available at the entry + exit of each block $B$ in the flow graph

```
OUT[ENTRY] = emptyset
for (each basic block B other than ENTRY) OUT[B] = U
while (changes to any OUT occur)
	for (each basic block B other than ENTRY) {
		IN[B] = intersection_{P a predecessor of B} OUT[P]
		OUT[B] = e_gen_B U (IN[B] - e_kill_B)
	}
```

## 9.2.7 Summary

denote meet operator as $\wedge$

## 9.3 Foundations of Data-Flow Analysis

data-flow analysis framework $(D, V, \wedge, F)$ consists of:
1. direction of data flow $D$, which is either $FORWARDS$ or $BACKWARDS$
2. semilattice: includes domain of values $V$ and meet operator $\wedge$
3. family $F$ of transfer functions from $V$ to $V$ —must include functions suitable for boundary conditions (constant transfer functions for special nodes $ENTRY$ and $EXIT$)

## 9.3.1 Semilattices

a set $V$ and a binary meet operator $\wedge$ such that for all $x, y, \text{ and } z$ in $V$
1. $x \wedge x = x$ (meet is idempotent)
2. $x \wedge y = y \wedge x$ (meet is commutative)
3. $x \wedge (y \wedge z) = (x \wedge y) \wedge z$

semilattice has a top element, $\top$, such that 
- for all $x$ in $V$, $\top \wedge x = x$, so $\top \geq x$ 
semilattice may have bottom element, $\perp$, such that
- for all $x$ in $V$, $\perp \wedge \;  x = \perp$, so $\perp \leq x$ 

### Partial Orders

the meet operator of a semilattice defines a partial order on the values of the domain
a relation $\leq$ defines a partial order on a set $V$ if for all $x, y, \text{and } z$ in $V$, the partial order is:
1. reflexive: $x \leq x$ 
2. antisymmetric: if $x \leq y$ and $y \leq x$, then $x = y$ 
3. transitive: if $x \leq y$ and $y \leq z$, then $x \leq z$ 

the pair $(V, \leq)$ is called a poset (partially ordered set)

a relation $<$ for a poset is defined as $x < y$ iff $x \leq y$ and $x \neq y$

### The Partial Order for a Semilattice

partial order for semilattice $(V, \wedge)$: for $x$ and $y$ in $V$, we define:
- $x \leq y$ iff $x \wedge y = x$

example:
for set union $\cup$  
- top element = $\emptyset$ 
- bottom element = $U$
- partial order imposed by set union is $\supseteq$, set inclusion
	- $x \cup y = x$ implies $x \supseteq y$ 
for set intersection
- top element = $U$
- bottom element = $\emptyset$
- partial order is $\subseteq$, set containment
	- $x \cap y = x$ implies $x \subseteq y$

- greatest solution (in the sense of partial order $\leq$) is the most precise

### Greatest Lower Bounds

Suppose $(V, \wedge)$ is a semilattice

A greatest lower bound (glb) of domain elements $x$ and $y$ is an element $g$ such that:
1. $g \leq x$, 
2. $g \leq y$, and
3. if $z$ is any element such that $z \leq x$ and $z \leq y$, then $z \leq g$

meet of $x$ and $y$ is their only greatest lower bound

### Lattice Diagrams

![[Pasted image 20260223155212.png]]

we can reed meet off diagram:
since $x \wedge y$ is the glb, it's always the highest $z$ for which there are paths downward from both $x$ and $y$
- meet of $d_1$ and $d_2$ is $d_1, d_2$

### Product Lattices

set of data-flow values = power set of definitions which contains $2^n$ elements if there are $n$ definitions

suppose $(A, \wedge_A)$ and $(B, \wedge_B)$ are semilattices. The product lattice is defines as:
1. the domain of the product lattice is $A \times B$
2. the meet $\wedge$ for the product lattice is defined as: if $(a, b)$ and $(a', b')$ are domain elements of the produce lattice then
	1. $(a, b) \wedge (a', b') = (a \wedge_A a', b \wedge_B b')$
3. partial order is:
	1. $(a, b) \leq (a', b')$ iff $a \leq_A a'$  and $b \leq_B b'$

### Height of a Semilattice

ascending chain in a poset $(V, \leq)$ is a sequence where $x_1 < x_2 < \dots < x_n$ 
heigh of semilattice = largest number of $<$ relations in any ascending chain = 1 - # elements in chain

showing convergence of an iterative data-flow program is much easier if the lattice has finite height

## 9.3.2 Transfer Functions

family of transfer functions $F: V \rightarrow V$ has the following properties:
1. $F$ has an identity function $I$, such that $I(x) = x$ for all $x$ in $V$
2. $F$ is closed under composition: for any 2 functions $f$ and $g$ in $F$, the function $h$ defined by $h(x) = g(f(x)) \in F$.

### Monotone Frameworks

to make an iterative algorithm for data analysis work, we need the data-flow framework to satisfy monotonicity condition

a framework is monotone if when we apply any transfer function $f$ in $F$ to 2 members of $V$, then the first result is no greater than the second result

a data-flow framework $(D, F, V, \wedge)$ is monotone if:
- for all $x$ and $y$ in $V$ and $f$ in $F$, $x \leq y$ implies $f(x) \leq f(y)$

monotonicity can be equivalently defined as:
- for all $x$ and $y$ in $V$ and $f$ in $F$, $f(x \wedge y) \leq f(x) \wedge f(y)$

### Distributive Frameworks

distributivity condition is stronger than previous monotonicity condition
distributivity implies monotonicity

framework often obeys distributivity condition:
$$f(x \wedge y) = f(x) \wedge f(y)$$

## 9.3.3 The Iterative Algorithm for General Frameworks

algo 9.25: iterative solution to general data-flow frameworks

INPUT: a data-flow framework with the following components:
1. a data-flow graph with specially labeled ENTRY and EXIT nodes
2. a direction of the data-flow $D$ 
3. a set of values $V$
4. a meet operator $\wedge$
5. a set of functions $F$, where $f_B$ in $F$ is the transfer function for block $B$
6. a constant value $v_{ENTRY}$ or $v_{EXIT}$ in $V$, representing the boundary condition for forward + backward frameworks respectively

OUTPUT: Values in $V$ for $IN[B]$ and $OUT[B]$ for each block $B$ in the data-flow graph

METHOD: 
iterative algo for forward data-flow problem:

```
OUT[ENTRY] = v_ENTRY
for (each basic block B other than ENTRY) OUT[B] = T
while (changes to any OUT occur)
	for (each basic block B other than ENTRY) {
		IN[B] = ^_P a predecessor of B OUT[P]
		OUT[B] = f_B(IN[B])
	}
```

iterative algorithm for backward data-flow problem:

```
IN[EXIT] = v_EXIT
for (each basic block B other than EXIT) IN[B] = T
	while (changes to any IN occur)
		for (each basic block B other than EXIT) {
			OUT[B] = ^_S a successor of B IN[S]
			IN[B] = f_B(OUT[B])
		}
```

we can use our abstract framework to prove the properties:
1. if algorithm converges, the result is a solution to the data-flow equations
2. if the framework is monotone, then the solution found is the max fixedpoint (MFP) fo the data-flow equations
	1. MFP is a solution with the property that in any other solution, $IN[B]$ and $OUT[B]$ are $\leq$ the corresponding MFP solutions
3. if the semilattice of the framework is monotone and of finite height, then the algo is guaranteed to converge

## 9.3.4 Meaning of a Data-Flow Solution

ideal solution to framework can't be obtained in general, but the algo approximates the ideal conservatively

### The Ideal Solution

the ideal result for a block B is:

$$IDEAL[B] = \bigwedge _{\text{P, a possible path from ENTRY to B}} f_P(v_{ENTRY})$$
- any answer greater than ideal is incorrect
- any value smaller than or equal to ideal is conservative
	- may include certain paths that either don'e exist in the flow graph or exist but that the program can never follow


### The Meet-Over-Paths Solution

in the data-flow abstraction, we assume that every path in the flow graph can be taken, thus the meet-over-paths solution for $B$ is:
$$MOP[B] = \bigwedge_{\text{P, a path from ENTRY to B}} f_P(v_{ENTRY})$$

meet solution meets together not only the data-flow values of all executable paths but also additional values associated with the paths that can't possible be executed. therefore, 

$MOP[B] \leq IDEAL[B]$
$MOP \leq IDEAL$

### The Maximum Fixedpoint Versus the MOP Solution

$MOP \leq IDEAL$ and $MFP \leq MOP$
Therefore, $MFP \leq IDEAL$

# 9.4 Constant Propagation

constant propagation/constant folding: replaces expressions that evaluate to the same constant every time they're executed

constant-propagation framework is different because:
- it has an unbounded set of possible dataflow values, even for a fixed flow graph
- it's not distributive

constant propagation is a forward data-flow problem

## 9.4.1 Data-Flow Values for the Constant Propagation Framework

set of data-flow values is a product lattice with 1 component for each variable in the program

lattice for a single variable consists of:
1. all constants appropriate for the type of variable
2. value NAC (not-a-constant)
3. value UNDEF: no def of the var has been discovered to reach this point

- UNDEF is the top element + NAC is the bottom element
NAC $\leq$ constant values $\leq$ UNDEF

So $UNDEF \wedge v = v$  and $NAC \wedge v = NAC$

for any constant $c$, $c \wedge c = c$

for distinct constants: $c_1 \wedge c_2 = NAC$

## 9.4.3 Transfer Functions for the Constant-Propagation Framework

semi-lattice of data-flow values is the product of semilattices like

![[Pasted image 20260228210130.png]]

$m \leq m'$ iff for all variables $v$ we have $m(v) \leq m'(v)$

## 9.3.4 Transfer Functions for the Constant-Propagation Framework

set F: certain transfer functions that accept a map of variables to values in the constant lattice and return another such map

F contains constant transfer function for ENTRY node:
- given any input map, returns a map $m_0$ where $m_0(v) = \text{UNDEF}$ for all variables $v$
- before executing any program statements, there are no definitions for any variables

Let $f_s$ = transfer function of statement $s$
Let $m$ and $m'$ represent data-flow values such that $m' = f_s(m)$

Two things can happen:
1. if $s$ is not an assignment statement, then $f_s$ is the identity function
2. if $s$ is an assignment to variable $x$, then $m'(v) = m(v)$ for all variables $v \neq x$, and $m'(x)$ is defined as:
	1. If RHS of statement $s$ is a constant $c$ then $m'(x) = c$
	2. If RHS is of the form $y + z$
		1. $m'(x) = m(y) + m(z)$ if $m(y)$ and $m(z)$ are constant values
		2. NAC if either $m(y)$ or $m(z)$ is NAC
		3. UNDEF otherwise
3. if RHS is any other expression (func call or assignment through a pointer), then $m'(x)=NAC$

## 9.4.4 Monotonicity of the Constant-Propagation Framework

Consider effect of function $f_s$ on a single variable

In all but case 2b), $f_s$ either doesn't change the value of $m(x)$ or it changes the map to return a constant or $NAC$ (which are $\leq$ than input)

For case 2b, we see that the output value can't get larger as the input gets smaller

## 9.4.5 Non-distributivity of the Constant-Propagation Framework

constant propagation framework is not distributive

## 9.4.6 Interpretation of the Results


# 9.5 Partial-Redundancy Elimination

PRE = minimizing the # of expression evaluations

## 9.5.1 The Sources of Redundancy

3 forms of redundancy:
1. common subexpressions
2. loop-invariant expressions
3. partially redundant expressions

![[Pasted image 20260302162958.png]]

### Global Common Subexpressions

An expressions $b + c$ is fully redundant at point $p$ if it's an available expression at that point. That is, $b + c$ has been computed along all paths reaching $p$, and the variables $b$ and $c$ were not redefined after the last evaluation of $b + c$

### Loop-Invariant Expressions

loop-invariant code motion: moving an expression from inside the loop outside

may need to be repeated b/c once you determine a variable has a loop-invariant value, expressions using that variable may also become loop-invariants

loops:

```
while c {
	S;
}
```

are rewritten as:
```
if c {
	repeat
		S;
	until not c
}
```

and the loop-invariant expressions are placed just before the repeat, so we avoid computing loop-invariant expressions if the loop is never executed

### Partially Redundant Expressions

requires placement of new computation expressions (along paths where they originally weren't)
- like loop-invariant code motion

## 9.5.2 Can All Redundancy Be Eliminated?

can we eliminate all redundant computations along every path? no unless we can change the flow graph by creating new blocks

critical edge of flow graph: edge going from a node with > 1 successor to a node with > 1 predecessor

eliminating all redundant expressions can greatly increase size of optimized code, so we restrict our redundancy-elimination techniques to only those that may add new new blocks but don't duplicate portions of the CFG

## 9.5.3 The Lazy-Code-Motion Problem

desirable properties for programs optimized with PRE:
1. all redundant computations of expressions that can be eliminated without code duplication are eliminated
2. optimized program doesn't perform any computation that's not in the original program execution
3. expressions are computed at the latest possible time (minimizes register usage between time variable is defined + time it's used)

lazy code motion: eliminating partial redundancy with the goal of delaying computations as much as possible

### Full Redundancy

an expressions $e$ in block $B$ is redundant if along all paths reaching $B$, $e$ has been evaluated + the operands haven't been redefined subsequently

$S$ = set of blocks where $e$ is defined that renders $e$ in $B$ redundant
- set of edges leaving $S$ form cutset which if removed, disconnect $B$ from the entry block

### Partial Redundancy

if an expression $e$ in block $B$ is only partially redundant, lazy-code motion algorithm will attempt to render $e$ fully redundant in $B$ by placing additional copies of $e$ in the flow graph
- if successful, we'll also have a cutset from set of blocks $S$ containing $e$ and $B$
- like the fully redundant case, no operands of $e$ are redefined along the paths that lead from blocks in $S$ to $B$

## 9.5.4 Anticipation of Expressions

- copies of expressions are only placed at program points where the expression is anticipated

- expression $b + c$ is anticipated at point $p$ if all paths leading from $p$ compute the value of $b + c$ from the values of $b$ and $c$ that are available at that point

## 9.5.5 The Lazy-Code-Motion Algorithm

### Algorithm Overview

1. find all expressions anticipated at each program point using a backward data-flow pass
2. places the computation where the values of the expressions are first anticipated along some path
	1. after we've placed copies of an expression where the expression is first anticipated, expression is ***available*** at point $p$ if it's been anticipated along all paths reaching $p$
	2. availability solved using forward data-flow pass
	3. to place expressions at earliest possible positions, find program points where expressions are anticipated but not available
3. executing an expression as soon as it's anticipated may produce a value long before it's used, so we want to postpone it
	1. an expression is ***postponable*** at a program point if it's been anticipated but not yet used along any path reaching the program point
	2. postponable expressions are found using forward data-flow pass
	3. place expressions at program points where they can no longer be postponed
4. final backward data-flow pass to eliminate assignments to temporary variables that are used only once in the program

### Preprocessing Steps

for each block $B$, compute:
$e\_use_B$: set of expressions computed in $B$
$e\_kill_B$: set of expressions any of whose operands are defined in $B$ 


### Anticipated Expressions

### Available Expressions

an expression is available on exit from a block if it's:
1. either
	1. available on entry, or
	2. in the set of anticipated expressions upon entry (it could be made available if we choose to compute it here)
2. not killed in the block

- set of expressions placed at block $B$ = set of anticipated expressions that are not yet available b/c we don't want to recompute an available expression
		$$earliest[B] = anticipated[B].in - available[B].in$$
### Postponable Expressions

postpones the computation of expressions as much as possible while preserving original program semantics and minimizing redundancy

an expression is postponable to the exit of block $B$ if it's not used in the block and either it's postponable to the entry of $B$ or it's used in $earliest[B]$ 

an expression is not postponable to the entry of a block unless all its predecessors include the expression in their postponable sets
- meet operator is set intersection

an expression $e$ is placed at the frontier (expression transitions from being postponable to not being postponable) if one of the following holds:
1. $e$ is not in $postponable[B].out$. In other words, $e$ is in $e\_use_B$
2. $e$ can't be postponed to one of its successors. there exists a successor of $B$ such that $e$ is not in the earliest or postponable set on entry to that successor
$e$ can be placed at the front of block $B$ in either scenario b/c of the new blocks introduced in the preprocessing step

### Used Expressions

backwards pass determines if the temp variables introduced are used beyond the block that they're in

an expression is used at point $p$ if there exists a path from point $p$ that uses the expression before the value is reevaluated

a used expression at the exit of a block B is a used expression on entry only if it's not in the latest set