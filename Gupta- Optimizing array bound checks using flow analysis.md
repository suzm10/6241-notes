
cd Project1
cmake -S . -B build -DCMAKE_INSTALL_PREFIX=./install -GNinja
cmake --build build --target install

cd install
./run_pass.sh global_1d_array
clang -o "global_1d_array.exe" "global_1d_array-transformed.bc" "../part1/boundscheck.cpp"
./global_1d_array.exe 1 2

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
