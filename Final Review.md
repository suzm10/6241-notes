
AST: automatically derived from parse trees— first intermediate form
- can do some optimizations in source to source manner
- if l-value is never used (no edge from l-value to use), can get rid of it
	- unnecessary overhead for further analysis, could create security issues

parallelizing compilers: put in pragmas, give it to parallelizing backends to split it, put in sync statements
- soure-to-source

next we looked at IR: algos for generating code
- translating each statement
- allocate new temp name during IR code generation
	- fits in single-assignment form. value-numbering becomes simple
	- first attempt at single-assignment form

go through textbook material behind slides
- muchnik handouts, kennedy

IR statements: linear IR
- put it in BB-oriented form
- BB's help us understand control-flow order
- once you know you're inside BB, you know control will flow from top-to-bottom

CFG: summarizes all possible executions that could happen under some inputs
- why over-approximation in static analysis? 
- maintain precision in terms of where control could go from

where do you lose precision? in terms of program paths, not targets

software that reconstructs CFG from binary

pointers/aliases— memory safety of code

2 types of pointer analyses:
- the one in LLVM: confined to 1 procedure, not context-sensitive, no interprocedural analysis
- flow-sensitive: 

in the presence of aliases, how do we want to modify their analysis?

production-side: 

cannot rule of infeasible paths easily, can only correlate predicates:
- given this branch prediction, can rule one out downhill

path-based analysis: can get around approximation of static analysis, not very high yield, don't use it in production

constant propagation: 

138 parameters that control different analyses in LLVM
premier ones: vectorization, GVN, constant prop, sparse conditional CP, loop-based optimizations, loop-invariant code motion
- these analyses have very high yield

sparse-conditional CP: taking advantage of deadness introduced, in path-based manner

SSA-based SCCP: why do we need SSA? lots of navigation, traversals that are facilitated by SSA edges, reachability condition

availability of an expression: forward-analysis, coming from uphill

anticipatability: backward-analysis, info coming from downhill

all analyses carried out at basic block boundaries

backward-analysis— register allocation, anticipatability is an all-paths issue (should be eval on all paths), liveness is at least one 1 path

current value: could be unsound (undef on 1 path), the one which could get used, should not be redefined

soundness: 

PRE— very important analysis in terms of deciding placements
- conjunction of avail + antic to determine paths of non-availability, stop where antic stops you
- if we give you infeasible paths, PRE can improve— can open up opportunities

expect certain integration questions in finals, application questions, new problems, simple questions

SSA— should be clear about reasons behind it, reason which value is associated with what name at a given program point
- can't do this in un-aliased way in original IR
- if both values are a constant, then this is a constant...

generation of SSA as an IR: very fast, renaming very fast (pre-order walk on dominator tree to rename)

focus on 3 optimizations: GVN (why we need SSA for GVN, local-value numbering for BB, to reason across phi node)
- GVN: has superseded CSE + other stuff, is the final say
- can GVN be extended? yes (dynamic SSA, reason which values along fork edges...)

dead-code elimination: backward-analysis based on SSA
- unreachable code— BBs dangling, not reachable from root block
- deadness from values being defined + not used (AST-based, IR level by using DU-UD chains)
- all values which don't contribute to return (used at call site) or critical (linked?) values

function-inlining: at call-site, only specialized version of func is necessary, results in great code savings
- inlining: gets rid of core of callee

parallelization, dependence analysis— approximation in integer domain, showed through I-test that Banerjee inequality: when it says there's independence, it's very likely there's independence
- I-test: no proofs, just understand

looked at loop transformations, specific cases, looked at locality, parallelization of things
- most subscripts are +- 1/c, can get complicated b/c of substitution + other loop transformations

register allocation: biggest impact in reg alloc, if you have infinite registers, will we have 0 spill code? except boundary LD/STs
- spill: taken from register + put into memory
- no— it's not safe to keep values in registers b/c if you have an alias to that vlaue + you keep updating reg value, the real value which is supposed to be in memory will be stale + not safe
- do promotion pass before: is value safe to be kept in register? yes, it'll be only updated through this name or alias

question 2: if you have an extremely fast computer to execute full algo for reg allocation, would a compiler writer be able to beat them using simpler analyses?
- range-splitting: simplification of interference graph, much better than NP-hard algorithms, very powerful transformation
- coalescing of values
- pre-spilling
- re-materializing values: out of registers, new values that we need
	- famous paper: re-materialization 

splitting paper: hennessy + chao

preparing for exam:
- it'll be 5 questions, bring your own paper
- May 5th, 2.5 hours expected
- new formulations, apply these concepts
- cross-connecting question between reg alloc + data-flow analysis or something like that; integration concepts
- knowledge of data-flow analysis, knowledge of safety + precision
	- ex: infeasible paths, precision
- straight-forward application of PRE, SCCV
- allow cheat-sheet— small index card
- checking new analysis— very simple one, is this problem forward or backward? show me why, formulate it, ...

compiler uses ARM