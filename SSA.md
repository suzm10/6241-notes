
# 1. Intro

use dominance frontiers to compute SSA form + control-dependence graph

optimizations that can benefit from using SSA:
-  code motion
- elimination of partial redundancies
- constant propagation

if a variable has $D$ definitions + $U$ uses, there can be $D \times U$ use-def chains.
when similar info is encoded in SSA form, there can be at most $E$ def-use chains

def-use info: list of places in program that each variable is uses b/c every use of V is a use of the value provided by the unique assignment to V

at join nodes, add special form of assignment, $\phi$-function, operands are assignments of V that reach the join point

![[Pasted image 20260315162713.png]]

## 1.1 Constant Propagation

when $\phi$ functions are placed carefully, nonlinear behavior is possible but unlikely
- size of SSA program is linear in the size of the original program
- time to do the work is linear in the SSA size

initially, algo assumes each edge is unexecutable + each variable is a constant with an unknown value (T)
- non-constant value marked $\perp$ during execution

![[Pasted image 20260315165859.png]]

Suppose V has 1 assignment in original program, so any use of V will either be use of the entry $V_0$ or a use of the value $V_1$. 
Let $X$ be a basic block that assigns to $V$, so X will determine value of $V$ when control flow along any edge $X \to Y$. 
- any node strictly dominated by $X$ will always see $V_1$ 
- control may be able to reach node $Z$ not strictly dominated by $X$. Suppose $Z$ is the first such node along a path, then $Z$ is in the dominance frontier of $X$ and need a $\phi$-function

we can place $\phi$-function for $V$ by finding the dominance frontier of every node that assigns to $V$

# 2. Control Flow Graphs

nonnull paths $p: X_0 \to X_J$ and $q: Y_0 \to Y_k$ are said to converge at a node $Z$ if:
$$X_0 \neq Y_0$$
$$X_J = Z = Y_K$$
$$(X_j = Y_k) \implies (j = J\text{ or }k = K)$$
paths $p$ and $q$ start at different nodes and are almost node-disjoint, but they come together at the end
- use "or" because one of the paths may happen to be a cycle, so we don't want to exclude the case $X_j = Y_K$ and vice versa

# 3. Static Single Assignment Form

assignment statement $A$ has the form:
- $LHS(A) \leftarrow RHS(A)$
- LHS is a tuple of distinct target variables and RHS is a tuple of expressions of the same length

2-steps to translate a program into SSA form:
1. some trivial $\phi$-functions $V \leftarrow \phi(V,V,\dots)$ are inserted at some of the join nodes in the program's CFG
2. new variables $V_i$ (for $i = 0, 1, 2, \dots$) are generated and mentions of $V$ are replaced with $V_i$

- each execution of the $\phi$ function in X uses only one of the operands, but which one depends on the flow of control just before entering X

a program is in SSA form if each variable is a target of exactly 1 assignment statement

translation to SSA form replaces original program by new program with the same CFG

for every original variable $V$, following conditions are required for the new program:
1. if 2 nonnull paths, $X \to Z$ and $Y \to Z$ converge at node $Z$, and nodes $X$ and $Y$ contain assignments to $V$ (in the original program), then a trivial $\phi$-function $V \leftarrow \phi(V, \dots, V)$ has been inserted at $Z$ (in the new program)
2. each mention of $V$ in the original program or in an inserted $\phi$-function has been replaced by a mention of a new variable $V_i$, leaving the new program in SSA form
3. along any control flow path, consider any use of a variable $V$ (in the original program) and corresponding use of $V_i$ (in the new program). Then $V$ and $V_i$ have the same value

minimal SSA form: has minimum # of $\phi$ functions subject to condition 1)

pruned SSA form: doesn't place a $\phi$-function at convergence point $Z$ if there are no more uses for $Z$ in or after $V$

## 3.1 Other Program Constructs

how to map some constructs to an explicit intermediate text suitable for use in a compiler

### 3.1.1 Arrays

![[Pasted image 20260315225949.png]]

![[Pasted image 20260315231219.png]]

problem for anyone who uses Update method is to communicate results of dependence analysis to optimizations that are usually formulated in terms of scalar variables
- HiddenUpdate solves this by not mentioning assigned array operand, instead using the target of the assignment

### 3.1.2 Structures

assignment to structure field = update of structure
use of structure field = access of structure

if analysis or language semantics reveals a structure whose $n$ fields are always accessed disjointly, then the structure can be decomposed into $n$ distinct variables
- expression of structure's decomposition in SSA form makes the program's actual use of the structure more apparent to subsequent optimization
### 3.1.3 Implicit References to Variables

construction of SSA form requires knowing variables modified and used by a statement

statement may use or modify variables not mentioned by statement itself (implicit references)

implicit references:
- global variables modified or used by a procedure call
- aliased variables
- dereferenced pointer variables

to obtain SSA form, we must account for implicit + explicit variables, either by conservative assumption or by analysis

to support optimizations of code that don't involve the heap but is interspersed with heap-dependent code:
- conservatively model heap storage by representing entire heap as a single variable that's both modified + used by any statement that changes the heap

for any statement S, 3 types of references affect translation into SSA form:
1. MustMod(S): set of variables that must be modified by execution of S
2. MayMod(S): set of variables that may be modified by execution of S
3. MayUse(S): set of variables whose values prior to execution of S may be used by S

represent implicit references for any statement S by transformation to an assignment statement A, where all variables in MayMod(S) appear in LHS(A) and all variables in $MayUse(S) \cup (MayMod(S) - MustMod(S))$  appear in RHS(A)

call might change any global variable or any parameter passed by reference, but it's reasonable to assume it won't change variables local to the caller

## 3.2 Overview of the SSA Algorithm

Translation to minimal SSA form takes 3 steps:
1. dominance frontier mapping is constructed from CFG
2. using dominance frontiers, locations of the $\phi$-functions for each variable in the original program are determined
3. variables are renamed by replacing each mention of an original variable $V$ by an appropriate mention of a new variable $V_i$

# 4. Dominance

## 4.1 Dominator Trees

Let $X$ and $Y$ be nodes in the CFG
if $X$ appears in every path from Entry to $Y$, then $X$ dominates $Y$, then $X$ dominates $Y$

domination is both reflexive + transitive

if $X$ dominates $Y$ and $X \neq Y$, then $X$ strictly dominates $Y$, $X \gg Y$

if $X$ doesn't strictly dominate $Y$, we write $X \not\gg Y$

immediate dominator of $Y$: closest strict dominator of Y on any path from Entry to Y

in a dominator tree, the children of a node $X$ are all immediately dominated by $X$

root of dominator tree is Entry and any node Y other than Entry has idom(Y) as its parent in the tree

dominator tree can be constructed from CFG in O(E) time

- predecessor, successor, + path always refer to CFG
- parent, child, ancestor, and descendant always refer to the dominator tree

## 4.2 Dominance Frontiers

$DF(X)$ of a CFG node $X$ is the set of all CFG nodes $Y$ such that $X$ dominates a predecessor of $Y$ but doesn't strictly dominate $Y$

![[Pasted image 20260316154755.png]]

to compute $DF(X)$ in time linear to $\sum_X |DF(X)|$, we define 2 intermediate sets for each node:

$$DF(X) = DF_{local}(X) \cup \bigcup_{Z \in Children(X)} DF_{up} Z$$
$DF_{local}X=$ successors of $X$ that contribute to $DF(X)$

$$DF_{local}X = \{Y \in Succ(X) | X \not\gg Y\}$$
$DF_{up}(Z)=$ contribution that $Z$ passes up to $idom(Z)$ is defined by:

$$DF_{up}(Z)=\{Y \in DF(Z) \;|\; idom(Z) \not\gg Y \}$$
##### Lemma 1. The dominance frontier equation is correct

##### Lemma 2. For any node X, 

$$DF_{local}(X) = \{Y \in Succ(X) \;|\; idom(Y) \neq X\}$$
Proof: $(X \gg Y) \implies (idom(Y) = X)$

##### Lemma 3. For any node X and any child Z of X in the dominator tree
$$DF_{up}(Z) = \{Y \in DF(Z) \;|\; idom(Y) \neq X\}$$
Proof: $(X \gg Y) \implies (idom(Y) = X)$

Calculating $DF(X)$ for each CFG node $X$:

```
for each X in a bottom-up traversal of the dominator tree do
	DF(X) <- 0
	for each Y in Succ(X) do
		if idom(Y) != X then DF(X) <- DF(X) U {Y}
	end
	for each Z in Children(X) do
		for each Y in DF(Z) do
			if idom(Y) != X then DF(X) = DF(X) U {Y}
		end
	end
end
```

## 4.3 Relating Dominance Frontiers to Joins

Given a set S of CFG nodes, the set the set J(S) of join nodes is defined to be the set of all nodes $Z$ such that there are 2 non-null CFG paths that start at 2 distinct nodes in S and converge at $Z$ 

iterated join $J^+(S)$ is the limit of the increasing sequence of sets of nodes:
$$J_1 = J(S)$$
$$J_{i+1} = J(S\; U J_i)$$
if $\ell$ happen to be the set of assignment nodes for a variable $V$, then $J^+(S)$ is the set of $\phi$-function nodes for V

join operations map sets of nodes to sets of nodes
extend dominance frontier mapping from nodes to sets of nodes:
$$DF(S) = \cup_{X \in S} DF(X)$$
if the set S is the set of assignment nodes for a variable V, then we'll show that
$$J^+(S) = DF^+(S)$$
the following lemmas relate dominance frontiers to joins:

##### Lemma 4. For any non-null path $p: X \to Z$ in CFG, there's a node $X' \in \{X\} \cup DF^+(\{X\})$ on p that dominates $Z$ 

##### Lemma 5: Let $X \neq Y$ be 2 nodes in CFG, and suppose that non-null paths $p: X \to Z$ and $q: Y \to Z$ in CFG converge at Z. Then $Z \in DF^+(\{X\}) \cup DF^+(\{Y\})$ 

##### Lemma 6: For any set S of CFG nodes, $J(S) \subseteq DF^+(S)$

##### Lemma 7: For any set $S$ of CFG nodes such that Entry $\in S$, $DF(S) \subseteq J(S)$

##### Theorem 2: The set of nodes that need $\phi$-functions for any variable $V$ is the iterated dominance frontier $DF^+(S)$ where $S$ is the set of nodes with assignments to $V$

## 5. Construction of Minimal SSA Form

### 5.1. Using Dominance Frontiers to Find Where $\phi$-functions are Needed

algo to insert trivial $\phi$-functions performs outer loop once for each var V in the program

data structures used:
- W = worklist of CFG nodes being processed
	- in each iteration, W = set of nodes that contain assignments to V
	- each node $X$ in worklist ensures that each node $Y$ in $DF(X)$ gets a $\phi$-function
- Work(X) = array of flags for each node $X$ indiciating whether or not it's in the worklist
- HasAlready(X) = array of flags for each node $X$ indicating whether a $\phi$-function for $V$ has already been inserted at $X$

Placement of $\phi$-functions:
```
IterCount = 0
for each node X do
	HasAlready(X) = 0
	Work(X) = 0
end
W = \emptyset
for each var V do
	IterCount = IterCount + 1
	for each X \in A(V)
		Work(X) = IterCount
		W = W U {X}
	end
	while W != \emptyset do
		take X from W
		for each Y in DF(X) do
			if HasAlready(X) < IterCount
				then do
					place <V = \phi(V,...,V)> at Y
					HasAlready(Y) = IterCount
					if Work(Y) < IterCount
						then do
							Work(Y) = IterCount
							W = W U {Y}
						end
			end
		end
	end
end
```

## 5.2. Renaming


