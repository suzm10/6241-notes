
cd Project1
cmake -S . -B build -DCMAKE_INSTALL_PREFIX=./install -GNinja
cmake --build build --target install

cd install
./run_pass.sh global_1d_array
clang -o "global_1d_array.exe" "global_1d_array-transformed.bc" "../part1/boundscheck.cpp"
./global_1d_array.exe 1 2

int a[10]
a[i] = 5

```
upper bound check:
checkUpper(index = i, bound = 10):
	i has to be < 10
	if i >= 10:
		out of bounds

checkLower(index = i, bound = 0)
	i has to be >= 0
	if i < 0:
		out of bounds
```
		
```
f(v) < g(v)
5 <= v = f(v)
5 <= v + 1 = g(v)
```

```
v + 1 <= 10
v <= 10
```
# 3.1 Local Elimination

checks can be eliminated from a basic block through local analysis in these situations:
1. identical checks:
	1. if C and C' are two identical checks, C precedes C', and the variables used in the check aren't redefined after C, then C' can be eliminated
2. subsumed checks with identical bounds
	1. if the two expressions are functions of the same variable, and the compiler determines that one check is always greater than the other, then one check can be eliminated b/c it's subsumed by the other check
		1. $$MIN \leq f(v) \text{ and } MIN \leq g(x) \equiv MIN \leq f(v) \text{ if } f(v) < g(v)$$
		2. $$f(v) \leq MAX \text{ and } g(v) \leq MAX \equiv g(v) \leq MAX \text{ if } f(v) < g(v)$$
3. subsumed checks with identical subexpressions
	1. 2 bound checks on the same subscript expression with different lower or upper bounds
		1. $$MIN_1 \leq f \text{ and } MIN_2 \leq f \equiv maximum(MIN_1, MIN_2) \leq f,$$
		2. $$f \leq MAX_1 \text{ and } f\leq MAX_2 \equiv f \leq minimum(MAX_1, MAX_2)$$

```
5 <= v
6 <= v
max(5, 6) = 6 <= v
5 <= v
```

```
v <= 10
v <= 11
v <= minimum(10, 11) = 10
```

# 3.2 Global Elimination

## Algo for modifying checks to create redundant checks

Def 3.1: A bound check C is very busy at a program point p if it's guaranteed that, along each path starting at point p, either C is performed, or a check that subsumes C is performed

Steps:
1. compute local info for all basic blocks
	1. C_GEN[B] = checks performed inside B
	2. EFFECT(B, v) = effect of block B on variable v
		1. ```
		   EFFECT(B, v) = {unchanged: v_after = v_before
							increment: v_after = v_before + 1
							decrement: v_after = v_before - 1
							multiply: v_after = v_before * c
							div > 1: v_after = v_before div c where c is a pos int
							div < 1: v_after = v_before div c where 0 < c < 1
							changed: relation b/t v_after and v_before unknown }
		   ```
2. compute very busy checks
	1. C_IN[B] = set of very busy checks at the start of B
	2. C_OUT[B]  = set of very busy checks at exit of B
	3. sets are computed using dataflow equations:
		1. ```
		   C_IN[B] = C_GEN[B] V backward(C_OUT[B], B),
		   C_OUT[B] = ^_{S in Succ(B)} C_IN[S], B not the terminating block
		   C_OUT[B] = emptyset, where B is the terminating block
		   ```
		2. $S_1 \wedge S_2 \wedge \dots \wedge S_n:$
			1. $= \{C:\forall S_i, 1 \leq i \leq n, (C \in S_i \vee \exists C' \in S_i \wedge C' \text{ subsumes } C)\}$
		3. $S_1 \vee S_2 \vee \cdots \vee S_n$:
			1. $= \{C: (\exists S_i, 1 \leq i \leq n, C \in S_i) \wedge (\not\exists C' \in S_i, 1 \leq i \leq n, C'\text{ subsumes } C)\}$
	4. ```
	   backward(C_OUT[B], B) {
	       S = emptyset
	       for each check C in C_OUT[B] do
		       case C of
		       lb <= v:
			       case AFFECT(B, v) of
				       unchanged: S = S U {lb <= v}
				       increment: /* the check is killed */
				       decrement: S = S U {lb <= v}
				       multiply: /* the check is killed */
				       div>1: S = S U {lb <= v}
				       div<1: /* the check is killed */
				       changed: /* the check is killed */
			       end case
			    v <= ub:
				    case AFFECT(B, v) of
						unchanged: S = S U {lb <= v}
				       increment: S = S U {lb <= v}
				       decrement: /* the check is killed */
				       multiply: S = S U {lb <= v}
				       div>1: /* the check is killed */
				       div<1: S = S U {lb <= v}
				       changed: /* the check is killed */
				    end case
				lb <= f(v):
					case AFFECT(B, v) of
						unchanged: S = S U {lb <= f(v)}
						inc, mult, div<1: if f(v) decreases when v increases then {S = S U {lb <= f(v)}} fi
						dec, div>1: if f(v) decreases when v decreases then {S = S U {lb <= f(v)}} fi
						changed: /* the check is killed */
					end case
				f(v) <= ub:
					case AFFECT(B, v) of
						unchanged: S = S U {lb <= f(v)}
						inc, mult, div<1: if f(v) increases when v increases then {S = S U {lb <= f(v)}} fi
						dec, div>1: if f(v) increases when v decreases then {S = S U {lb <= f(v)}} fi
						changed: /* the check is killed */
					end case
	       od
	       return(S)
	   }
	   ```
3. modify checks
	1. check C is modified if we determine that there's another check C' that's very busy at the point immediately following C and C' subsumes C

## Algo for elimination of redundant checks



```
5 <= v1 - 1, 5 <= v1, 5 <= 6 

v2 = v1 - 1, 5 = 6 - 1

5 <= v2, 5 <= 5
```

```
typedef std::pair<Value *, SCEV *> IndexPair;
```

# Algo for propagation

assume that prior to applying algo, we've identified loops and computed use-def chains and dominator sets

## Step 1: Identify candidates for propagation

check C is a candidate for moving out of the loop if once executed, it need not be executed in subsequent loop iterations

4 cases that identify candidates for propagation:
1. Invariants: 
	- A check C is a candidate for propagation if it uses only definitions from outside the loop body (C is a loop invariant). Determined from use-def information
2. Increasing values: 
	- A check C is a candidate for propagation if it's of the form $lb \leq i$ and if all defs of i inside the loop are of the form $i = i + c$, $i = c + i$,  $i = i * c$, or $i = c * i$, where c is a pos int const
3. Decreasing values:
	- A check C is a candidate for propagation if it's of the form $i \leq ub$ and if all the definitions of $i$ inside the loop are of the form $i = i - c$, $i = -c + i$, or $i = i/c$ where $c$ is a pos int const
4. Loops with inc/dec of one:
	- Consider a check C of the form $-lb \leq \text{i op c}, \text{i op c} \leq ub$ , where $i$ is incremented/decremented by one during each iteration, $op \in \{+, -, div, *\}$, $c$ is a const, and min and max are the min and max values taken by var $i$
	- This check is a candidate for propagation b/c it can replaced by $lb \leq \text{min op c}, \text{max op c} \leq ub$ outside the loop
## Step 2: Check hoisting

A check that's a candidate for propagation can be propagated only if it's in a block that dominates loop exits. 

So, we hoist checks from blocks that are conditionally executed to blocks that unconditionally executed during each loop iteration to increase the number of checks propagated out of a loop

ND = set of blocks from which checks will be hoisted
C(n) = contains the checks in block n that are candidates for hoisting

hoist {
	ND = {n: block n does not dominate all loop exits}
	for each block n do
		C(n) = {c: at the entry to n we can assert that candidate check c will be executed in n}
	od
	change = true
	while change do
		change = false
		for each block $n \in Succ(n) \cap ND \neq \emptyset \wedge n$  is the unique predecessor of nodes in $Succ(n)$ do
			 prop = $\bigwedge_{S \in Succ(n)} C(s)$
			 if prop $\neq \emptyset$ then
				 change = true
				 hoist checks in prop to n
				 for each check $c \in prop$ do
					 if $c \in S$, $S \in Succ(n)$ then eliminate c from S fi
				 od
			 fi
		 od
	od
}

## Step 3: Propagate checks out of the loop

propagate all candidate checks from blocks that dominate all loop exits
according to step 1, some checks are modified, but others propagate unchanged

