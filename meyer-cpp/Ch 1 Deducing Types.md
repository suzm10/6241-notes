
## Item 1: Understand template type deduction

general form for templates + call to it:

```
template<typename T>
void f(ParamType param);

f(expr)
```

during compilation, compilers use expr to deduce type of T and ParamType (types can be diff)

```
template<typename T>
void f(const T& param);

int x = 0;
f(x);
```

T is deduced to be `int` + ParamType is deduced to be `const int&`

## Case 1: ParamType is a Reference or Pointer, but not a Universal Reference

type deduction for lvalue reference parameters:

```
template<typename T>
void f(T& param);

int x = 27;
const int cx = x;
const int& rx = x;

f(x) // T is int, param's type is int&
f(cx) // T is const int, param's type is const int&
f(rx) // T is const int, param's type is const int&
```

even though rx is a reference, T is deduced to be a non-reference because rx's reference-ness is ignored during type deduction

if we change param type to `const T&`, there's no need for const to be deduced as a part of T

```
template<typename T>
void f(const T& param);

int x = 27;
const int cx = x;
const int& rx = x;

f(x) // T is int, param's type is const int&
f(cx) // T is int, param's type is const int&
f(rx) // T is int, param's type is const int&
```

if param were a pointer instead of a reference:
```
template<typename T>
void f(T* param);

int x = 27;
const int *px = &x

f(&x); // T is int, param's type is int*
f(px); // T is const int, param's type is const int*
```

## Case 2: ParamType is a Universal Reference

if expr is an lvalue, both T and ParamType are deduced to be lvalue references

if expr is an rvalue, its type is deduced normally

```
template<typename T>
void f(T&& param); // param is a universal reference

int x = 27;
const int cx = x;
const int& rx = x;

f(x); // x is lvalue, so T is int& + param's type is int&

f(cx); // cx is lvalue, so T is const int& + param's type is const int&

f(rx); // rx is lvalue, so T is const int& + param's type is const int&

f(27); // 27 is rvalue, so T is int + param's type is int&&
```

## Case 3: ParamType is Neither a Pointer nor a Reference

- pass-by-value: ParamType is neither a pointer nor a reference

```
template<typename T>
void f(T param)
```

param will be a copy of whatever's passed in— a new object

rules are:
- if expr's type is a reference, ignore the ref part
- if expr is a const or volatile, ignore that too

```
int x = 27;
const int cx = x;
const int& rx = x;

f(x); // T's and param's types are both int
f(cx); //
f(rx); // 
```

since cx and rx are copied into param, their constness is ignored

for references to or pointers to const, the constness of expr is preserved during type deduction. consider when expr is a const pointer to a const object:

```
template<typename T>
void f(T param);

const char* const ptr = "Fun with ptrs"; // const ptr to const object

f(ptr); // param's type is const char *
```

the pointer itself will be passed by value. the constness of ptr will be ignored, and the type deduced for param will be `const char *`

## Array Arguments

array types are different from pointer types

```
template<typename T>
void f(T param);

const char name[] = "J. P. Briggs"

const char * ptrToName = name;

f(name); // name is array but type is deduced to const char*
```

functions can't declare parameters that are truly arrays, but they can declare parameters that are references to arrays

```
template<typename T>
void f(T& param);

f(name); 
```

T is deduced to the actual type of the array, const char[13], the type of f's parameter is const char (&)[13]

ability to declare refs to arrays enables creation of a template that deduces the number of elements that an array contains:
```
template<typename T, std::size_t N>
constexpr std::size_t arraySize(T (&)[N]) nonexcept {
	return N
}
```

makes it possible to declare an array with the same number of elements as a second array whose size is computed from a braced initializer:

```
int keyVals[] = {1, 3, 7, 9, 11, 22, 35}; // keyVals has 7 elements
int mappedVals[arraySize(keyVals)]; // so does mappedVals
```

## Function Arguments

function types can decay into function pointers too

```
void someFunc(int, double);

template<typename T>
void f1(T param); // param passed by value

template<typename T>
void f2(T& param); // param passed by ref

f1(someFunc);

f2(someFunc);
```

## Item 2: Understand auto type deduction

```
template<typename T>
void f(ParamType param);

f(expr);
```

when a variable is declared using auto, auto plays the role of T in the template + the type specifier for the variable acts as ParamType

```
auto x = 27;
```

the type specifier for x is simply auto by itself

in the declaration below, the type specifier is `const auto`

```
cont auto cx = x;
```

and here, the type specifier is `const auto&`

```
const auto& rx = x;
```

to deduce types for x, cx, and rx, the compiler acts as if there were a template for each declaration + a call to that template with the corresponding initializing expression

```
template<typename T>
void func_for_x(T param);

func_for_x(27); // param's deduced type is x's type

template<typename T>
void func_for_cx(const T param);

func_for_cx(x);

template<typename T>
void func_for_rx(const T& param);

func_for_rx(x);
```

3 cases for type specifier just like ParamType:
- case 1: the type specifier is a pointer or a reference, but not a universal reference
- case 2: type specifier is a universal reference
- case 3: the type specifier is neither a pointer nor a ref

```
auto x = 27; // case 3
const auto cx = x; // case 3
const auto& rx = x; // case 1
```

case 2:
```
auto&& uref1 = x; // x is int + lvalue, so uref1's type is int&
auto&& uref2 = cx; // cx is const int + lvalue, so uref2's type is const int &
auto&& uref3 = 27; // 27 is int + rvalue, so uref3's type is int&&
```

for auto type deduction, array + func names decay into pointers for non-reference type specifiers too:

```
const char name[] = "R. N. Briggs";

auto arr1 = name; // arr1's type is const char (&)[13]
auto& arr2 = name // arr2's type is const char (&)[13]
```

```
void someFunc(int, double); // type of someFunc is void(int, double)

auto func1 = someFunc; // func1's type is void (*)(int, double)

auto& func2 = someFunc; // func2's type is void (&)(int, double)
```

auto type deduction differs from template type deduction in 1 way: treatment of braced initializers

```
auto x1 = 27; // type is int
auto x2(27); //
auto x3 = { 27 }; // type is std::initializer_list<int>, value is { 27 }
auto x4{ 27 };
```

special type deduction rule for auto: when the initializer list for an auto-declared variable is enclosed in braces, the deduced type is a std::initializer_list. if such a type can't be deduced ( values in braced initializers are diff types), code will be rejected:

```
auto x5 = {1, 2, 3.0} // error, can't deduce T for std::initializer_list<T>
```

when an auto-declared variable is initialized with a braced initializer, the deduced type is an instantiation of `std::initializer_list`. if the corresponding template is passed the same initializer, type deduction fails:

```
auto x = {11, 23, 9};

template<typename T>
void f(T param);

f({11, 23, 9});
```

if you specify in the template that param is a `std::initializer_list<T>` for some unknown T, template type deduction will deduce what T is:

```
template<typename T>
void f(std::initializer_list<T> initList);

f({11, 23, 9}); // T is deduced as int, and initList's type is std::initializer_list<int>
```

auto assumed a braced `initializer` represents a `std::initializer_list`, but template type deduction doesn't

C++14 permits auto to indicate that a function's return type should be deduced + lambdas may use auto in parameter declarations— these uses of auto employ template type deduction, not auto type deduction

so a function with an auto return type that returns a braced initializer won't compile:
```
auto createInitList() {
	return { 1, 2, 3 }; // error! can't deduce type for { 1, 2, 3 }
}
```

```
std::vector<int> v;

auto resetV = [&v](const auto& newValue) { v = newValue; }

resetV({1, 2, 3}); // error! can't deduce type for {1, 2, 3}
```

## Item 3: Understand decltype

given a name or an expression, decltype tells you the name or expression's type

```
const int i = 0;         // decltype(i) = const int

bool f(const Widget& w); // decltype(w) is const Widget&
                         // decltype(f) is bool(const Widget&)
                         
struct Point {
	int x, y;            // decltype(Point::x) is int
}                        // decltype(Point::y) is int

Widget w;                // decltype(w) is Widget

is (f(w))...             // decltype(f(w)) is bool

template<typename T>
class vector {
public:
	T& operator[](std::size_t index);
}

vector<int> v;            // decltype(v) is vector<int>

if (v[0] == 0) ...        // decltype(v[0]) is int&
```

primary use for decltype: declaring function templates where the function's return type depends on its parameter types

write a function that takes a container that supports indexing via [] plus an index, then authenticates the user before returning the result of the indexing operation
```
template<typename Container, typename Index>
auto authAndAccess(Container& c, Index i) -> decltype(c[i]) {
  authenticateUser();
  return c[i];
}
```

C++11's trailing return type syntax allows a function's parameters to be used in the specification of the return type

In C++14, compilers can deduce the function's return type from the function's implementation:
```
template<typename Container, typename Index> // not correct
auto authAndAccess(Container& c, Index i) {
	authenticateUser();
	return c[i];
}
```

during template type deduction, the reference-ness of an initializing expression is ignored

```
std::deque<int> d;

authAndAccess(d, 5) = 10; // d[5] returns int&, but auto return type deduction for authAndAccess will strip off the reference, yielding an int return type
```

to get auto to work, we need it to use decltype rules, so authAndAccess will return whatever `c[i]` returns

```
template<typename Container, typename Index>
decltype(auto)
authAndAccess(Container& c, Index i) {
	authenticateUser();
	return c[i];
}
```

if `c[i]` returns a `T&`, authAndAccess will also return a `T&`. same for if it returns an object

`decltype(auto)` can be convenient for declaring variables when you want to apply decltype type deduction rules to the initializing expression:

```
Widget w;
const Widget& cw = w;
auto myWidget1 = cw; // myWidget1's type is Widget
decltype(auto) myWidget2 = cw; // myWidget2's type is const Widget&
```

to have authAndAccess employ a reference parameter that can bind to lvalues + rvalues, use a universal reference:

```
template<typename Container, typename Index>
decltype(auto) authAndAccess(Container &&c, Index i);
authAndAccess(Container&& c, Index i) {
	authenticateUser();
	return std::forward<Container>(c)[i];
}
```

In C++11, you have to specify the return type yourself
```
template<typename Container, typename Index>
auto authAndAccess(Container&& c, Index i) -> decltype(std::forward<Container>(c)[i]) {
	authenticateUser();
	return std::forward<Container>(c)[i];
}
```

the way you write a return statement can affect the deduced type for a function:

```
decltype(auto) f1() {
	int x = 0;
	return x; // decltype(x) is int, so f1 returns int
}
decltype(auto) f2() {
	int x = 0;
	return (x); // decltype((x)) is int&, so f2 returns int&
}
```

## Item 4: Know how to view deduced types

# Ch 2: auto

## Item 5: Prefer auto to explicit type declarations.

auto variables have their type deduced from their initializer, so they must be initialized

```
int x1; // potentially uninit
auto x2; // error! initializer required
auto x3 = 0; // fine, x's value is well-defined
```

easy to declare local variables:

```
template<typename It>
void dwim(It b, It e) {
	while (b != e) {
		auto currValue = *b;
	}
}
```

because auto uses type deduction, it can represent types known only to compilers:

```
auto derefUPLess = [](const std::unique_ptr<Widget>& p1, const std::unique_ptr<Widget>& p2) {
	return *p1 < *p2;
};
```

In C++14, parameters to lambda expressions may involve auto:

```
auto derefLess = [](const auto& p1, const auto& p2) {
	return *p1 < *p2; // C++14 comparison func for values pointed to 
					  // by anything pointer-like
};
```

std::function objects use more memory than auto-declared objects

using auto avoids problems related to type shortcuts:

```
std::vector<int> v;
unsigned sz = v.size();
```

the official return type of `v.size()` is `std::vector<int>::size_type`, so if there's a difference in number of bits between unsigned and this, it can cause problems on some machines

using auto is better:
```
auto sz = v.size();
```

another example of explicitly specifying types leading to implicit conversions that you don't want:

```
std::unordered_map<std::string, int> m;

for (const std::pair<std::string, int>& p : m) {
	// do something with p
}
```

type of `std::pair` in the hashtable is `std::pair<const std::string, int>`, so a compiler has to convert this object into the one without a const key. it'll succeed by creating a temp object of the type p wants to bind to. at the end of each loop iter, this temp object gets destroyed. we intended to bind the reference p to each element in m though

type mismatches can be auto'd away:

```
for (cont auto& p: m) {
	//
}
```

if you take p's address, you're sure to get a pointer to an element within `m` 


## Item 6: Use the explicitly typed initializer idion when auto deduces undesired types

invisible proxy classes don't play well with auto because objects of such classes aren't designed to live longer than a single statement, so creating variables of those types violates library design assumptions

avoid code of this form:
```
auto someVar = expression of "invisible" proxy class type
```

for this example:

```
std::vector<bool> features(const Widget& w);

Widget w;

bool highPriority = features(w)[5]; // bit 5: whether Widget has high priority

processWidget(w, highPriority);
```

however this code leads to undefined behavior when we call `processWidget` because the type of `highPriority` is `std::vector<bool>::reference`. this type doesn't get implicitly converted to bool anymore:

```
auto highPriority = features(w)[5];

processWidget(w, highPriority);
```

in this case, you can force a different type deduction using the explicitly typed initializer idiom:

```
auto highPriority = static_cast<bool>(features(w)[5])
```

the cast is used to force `highPriority` into being a `bool` 

the explicitly typed initializer idiom can also make it clear that you're creating a variable of type different from the one generated by the initializing expression. for example:

```
double calcEpsilon();

float ep = calcEpsilon(); // explicitly convert double -> float

auto ep = static_cast<float>(calcEpsilon());
```

the auto way makes it clear you're deliberately reducing the precision of the value returned by the function

# Ch 3: Moving to Modern C++

## Item 7: Distinguish between () and {} when creating objects

initialization values may be specified with parentheses, an equals sign, or braces:

```
int x(0);
int y = 0;
int z{ 0 };
```

for user-defined types, it's important to distinguish initialization from assignment b/c different function calls are involved:

```
Widget w1; // calls defualt constructor
Widget w2 = w1; // not an assignment; calls copy ctor
w1 = w2; // an assignment; calls copy operator=
```

to address the confusion of multiple initialization syntaxes, C++11 introduces uniform initialization: a single initialization syntax that can be used anywhere to express everything, braced initialization

braced initialization lets you use braces to specify the initial content of a container:
```
std::vector<int> v{1, 3, 5};
```

braces can also be used to specify default initialization values for non-static data members. this capability (new C++11) is shared with the = initialization syntax:
```
class Widget {
	private:
		int x{ 0 }; // x's default value is 0
		int y = 0; // same
		int z(0); // error!
};
```

un-copyable objects may be initialized using braces or parentheses but not =:
```
std::atomic<int> ai1{ 0 }; // fine
std::atomic<int> ai2(0); // fine
std::atomic<int> ai3 = 0; // error!
```

braced initialization is uniform because it works everywhere

braced initialization prohibits implicit narrowing conversions among built-in types:
```
double x, y, z;
int sum1{x + y + z}; // error! sum of double may not be expressible as int
int sum2(x + y + z); // value of expression truncated to an int
int sum3 = x + y + z; // same
```

initialization using () and = doesn't check for narrowing conversions b/c that could break legacy code

braced initialization helps call a function with no args b/c functions can't be declared using braces for the param list

```
Widget w1(10); // call Widget ctor with argument 10
Widget w2(); // most vexing parse! declares a func named w2 that rets a Widget
Widget w3{}; // calls Widget ctor with no args
```

in summary, braced initialization can be used in a wide variety of contexts, prevents implicit narrowing conversion, + it's immune to C++'s most vexing parse

drawback: when an auto-declared variable has a braced initializer, the type deduced is `std::initializer_list`

in constructor calls, () and {} have the same meaning as long as `std::initializer_list` params aren't involved

```
class Widget {
public:
	Widget(int i, bool b); // ctors not declaring std::initializer_list params
	Widget(int i, double d)
};

Widget w1(10, true); // calls 1st ctor
Widget w2{10, true}; // same
Widget w3(10, 5.0); // calls 2nd ctor
Widget w4{10, 5.0}; // same
```

if 1+ constructors declare a param of type std::initializer_list, calls using the braced initialization syntax strongly prefer overloads taking std::initializer_lists

for example, with this new constructor, widgets w2 and w4 will call std::initializer_list constructor
```
class Widget {
public:
	Widget(int i, bool b);
	Widget(int i, double d);
	Widget(std::initializer_list<long double> il); // added
};
```

what would normally be copy + move construction can be hijacked by `std::initializer_list` constructors:

```
class Widget {
public:
	Widget(int i, bool b);
	Widget(int i, double d);
	Widget(std::initializer_list<long double> il);
	operator float() const; // convert to float
};

Widget w5(w4); // calls copy ctor
Widget w6{w4}; // calls std::initializer_list ctor, w4 converts to float + float                // converts to long double
Widget w7(std::move(w4)) // calls move ctor
Widget w8{std::move(w4)}; // calls std::initializer_list ctor
```

compilers matches braced initializers to constructors taking std::initializer_lists even if std::initializer_list constructor can't be called

```
class Widget {
public:
	Widget(int i, bool b);
	Widget(int i, double d);
	Widget(std::initializer_list<bool> il);
};

Widget w{10, 5.0}; // error! requires narrowing conversions from double to bool, 
				   // but narrowing conversions are prohibited inside braced                        // initializers, so the call is invalid
```

only if there's no way to convert the types of the arguments in a braced initializer to the type in a `std::initializer_list` do compilers fall back on normal overload resolution. for example,

```
class Widget {
public:
	Widget(int i, bool b);
	Widget(int i, double d);
	Widget(std::initializer_list<std::string> il); //no implicit conversion funcs
};

Widget w1(10, true); // calls 1st ctor
Widget w2{10, true}; // same
Widget w3(10, 5.0); // calls 2nd ctor
Widget w4{10, 5.0}; // same
```

empty braces mean no arguments, and you get default construction, not a `std::initializer_list` with no elements

```
class Widget {
public:
	Widget(); // default ctor
	Widget(std::initializer_list<int> il); // std::initializer_list ctor
	... // no implicit conversion funcs
};

Widget w1; // calls default ctor
Widget w2{}; // same
Widget w3();  // most vexing parse, declares a func
```

if you want to call a `std::initializer_list` constructor with an empty `std::initializer_list`, you do it by making the empty braces a constructor arg

```
Widget w4({}); // calls std::initializer_list ctor with empty list
Widget w5{{}}; // same
```

for vectors: () vs {} matter because std::vector has a constructor that takes std::initializer_list that permits you to specify the initial values in the container

```
// use non-std::initializer list ctor: create 10-elem std::vector with values 20
std::vector<int> v1(10, 20);

// use std::initializer_list ctor: create 2-element std::vector, elems are 10, 20
std::vector<int> v2{10, 20};
```

## Item 8: Prefer nullptr to 0 and NULL

C++'s primary policy is that 0 is an int, not a pointer, NULL doesn't have a pointer type either

```
void f(int); // 3 overloads of f
void f(bool);
void f(void *);

f(0); // calls f(int), not f(void *)

f(NULL) // might not compiler but typically calls f(int), not f(void *)
```

`nullptr`'s actual type is `std::nullptr_t`. `std::nullptr_t` implicitly converts to all raw pointer types

calling overloaded function `f` with `nullptr` calls the `void *` overload b/c `nullptr` can't be viewed as anything integral:
```
f(nullptr); // calls f(void *) overload
```

`nullptr` can also improve code clarity. in this example, we won't know result's type:

```
auto result = findRecord(/* args */);
if (result == nullptr) {
	...
}
```

but now it's clear result is a pointer type:

```
auto result = findRecord(/* args */);
if (result == nullptr) {
	...
}
```

suppose you have some function that should be called only when the appropriate mutex has been locked:
```
int f1(std::shared_ptr<Widget> spw);
double f2(std::unique_ptr<Widget> upw);
bool f3(Widget *pw);
```

template type deduction deduces int for `0` and `NULL` instead of a pointer type like it would for `std::nullptr_t`

## Item 9: Prefer alias declarations to typedefs

