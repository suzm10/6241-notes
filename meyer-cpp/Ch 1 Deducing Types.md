
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

here's a typedef:

```
typedef std::unique_ptr<std::unordered_map<std::string, std::string>> UptrMapSS;
```

C++11 also offers alias declarations that do the same thing:

```
using UPtrMapSS = std::unique_ptr<std::unordered_map<std::string, std::string>>;
```

alias declarations are easier to deal with when it comes to types involving function pointers:

```
typedef void (*FP)(int, const std::string&);
```

same as:
```
using FP = void (*)(int, const std::string&);
```

alias declarations may be templatized (alias templates), but typedefs cannot

consider defining a synonym for a LL that uses a custom allocator, `MyAlloc`

```
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;
// MyAllocList<T> is a synonym for std::list<T, MyAlloc<T>>

MyAllocList<Widget> lw; // client code
```

using a typedef is more work:
```
template<typename T>
struct MyAllocList {
	typedef std::list<T, MyAlloc<T>> type;
};
// MyAllocList<T> is synonym for std::list<T, MyAlloc<T>> type;

MyAllocList<Widget>::type lw;
```

if you want to use typedefs inside a template for the purpose of creating a LL holding objects of type T, you have to use typename because `MyAllocList<T>::type` is a dependent type (its type is dependent on a  template type parameter `T`):
```
template<typename T>
class Widget {
private:
	typename MyAllocList<T>::type list;
}

// Widget<T> contains a MyAllocList<T> member
```

if `MyAllocList` is defined as an alias template, we don't need `typename`

```
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;

template<typename T>
class Widget {
private:
	MyAllocList<T> list;
}
```

`MyAllocList<T>` is non-dependent type, so the `typename` specifier is not needed or allowed. Compilers know `MyAllocList<T>` is the name of a type because `MyAllocList` is an alias template (alias templates must name a type).

When compilers see `MyAllocList<T>::type` in the `Widget` template, they can't know for sure that it names a type because there might be specialization of `MyAllocList` that names something other than a type

C++11 gives you the tools to perform type transformations in the form of type traits, template insider the header `<type_traits>`

Given a type `T` to which you'd like to apply a transformation,  the resulting type is `std::transformation<T>::type`:
```
std::remove_const<T>::type // yields T from const T
std::remove_reference<T>::type // yields T from T& and T&&
std::add_lvalue_reference<T>::type // yields T& from T
```

If you applied these to a type parameter inside a template, you'd have to precede it with `typename` because in C++11, type traits are implemented as typedefs inside templatized structs

In C++14, there's a corresponding alias template named `std::transformation_t` for each C++11 `std::transformation<T>::type`

```
std::remove_const_t<T>

std::remove_reference_t<T>

std:add_lvalue_reference_t<T>
```

In C++11, implementing these yourself is very easy:
```
template<class T>
using remove_const_t = typename remove_const<T>::type;

template<class T>
using remove_reference_t = typename remove_reference<T>::type;

template<class T>
using add_lvalue_reference_t = typename add_lvalue_reference<T>::type;
```

## Item 10: Prefer scoped enums to unscoped enums

for C++98-style enums, the names of enums belong to the scope containing the enum, so nothing else in that scope may have the same name:

```
enum Color { black, white, red }; // black, white, red in same scope as Color
auto white = false; // error! white already declared in this scope
```

unscoped enums: leak into the scope containing their enum def

scoped enums: don't leak names this way, declared via enum class, aka enum classes

```
enum class Color { black, white, red }; // black, white, red are scoped to Color
auto white = false; // fine, not other white in scope
Color c = white; // error! no enum named white in this scope
Color c = Color::white; // fine
auto c = Color::white; // fine
```

another advantage: scoped enums enumerators are much more strongly typed
- enumerators for unscoped enums implicitly convert to integral types

another advantage: scoped enums may be forward-declared (named declared without specifying enumerators)
```
enum Color; // error
enum class Color; // fine
```

for un-scoped enums, compilers want to choose the smallest underlying type for an enum that's sufficient to represent its range of enumerator values, before enum is used

being able to forward-declare enums in C++11 eliminates having to recompile an entire system when only 1 new value is introduced to an enum that isn't used inside a function that takes that enum

how can C++11's enums get away with forward declarations but C++98's enums can't? the underlying type for a scoped enum is always known

the underlying type for scoped enums is `int`, but you can change it:
```
enum class Status; // ints
enum class Status: std::uint32_t; // changed underlying type for Status
```

you can do the same thing for an unscoped enum, and the result may be forward-declared:
```
enum Color: std::uint8_t;
enum class Status: std::uint32_t { good = 0, 
								   failed = 1, 
								   incomplete = 100, 
								   corrupt = 200, 
								   audited = 500, 
								   indeterminate = 0xFFFFFFFF };
```

scoped enums help avoid namespace pollution + nonsensical type conversions, but there's 1 situation where unscoped enums may be useful:

```
enum UserInfoFields { uiName, uiEmail, uiReputation };
UserInfor uInfo;

auto val = std::get<uiEmail>(uInfo);
```

this works because of the implicit conversion from `userInfoFields` to `std::size_t`. using a scoped enum would be more verbose

using a scoped enum is more typing because it requires a cast to pass into `std::get`:

```
enum class UserInfoFields { uiName, uiEmail, uiReputation };

UserInfo uInfo;

auto val = std::get<static_cast<std::size_t>(UserInfoFields::uiEmail)>(uInfo);
```

or you can write a function to perform the cast:
```
template<typename E>
constexpr auto 
	toUType(E enumerator) noexcept
{
	return static_cast<std::underlying_type_t<E>>(enumerator);
}
```

`toUType` permits us to access a field of the tuple like this:
```
auto val = std::get<toUType(UserInfoFields::uiEmail)>(uInfo);
```

## Item 11: Prefer deleted functions to private undefined ones

C++98 approach to preventing use of these functions is to declare them private + not define them

to render istream + ostream classes uncopyable, `basic_ios` (the parent class) is defined like this:

```
template<class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
	...
private:
	basic_ios(const basic_ios& ); // not defined
	basic_ios& operator=(const basic_ios&); // not defined
};
```

declaring functions private prevents clients from calling them. if member functions of friends of the class use them, linking will fail due to missing function definitions

In C++11, you can achieve the same thing with  `= delete` to mark the copy constructor and the copy assignment operator as deleted functions

```
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
	... // deleted funcs are declare public, not private, by convention
	basic_ios(const basic_ios& ) = delete; 
	basic_ios& operator=(const basic_ios&) = delete;
	...
};
```

deleted functions may not be used in any way, so code that's in member and friend functions will fail to compile if it tries to copy `basic_ios` objects. that's an improvement over C++98 because improper usage isn't diagnosed until link-time

advantage of deleted functions: any function may be deleted while only member functions may be private

example: suppose we have a non-member function that takes an int + returns whether it's a lucky number, and we want to prevent calls that implicitly convert to int args

```
bool isLucky(int number);
bool isLucky(char) = delete; // reject chars
bool isLucky(bool) = delete; // reject bools
bool isLucky(double) = delete; // reject double
```

deleted functions are taken into account during overload resolution, so undesirable calls will get rejected:
```
if (isLucky('a')) ... // error! call to deleted func
is (isLucky(true)) ... // error!
if (isLucky(3.5f)) ... // error!
```

deleted functions can also prevent use of template instantiations that should be disabled:
```
template<typename T>
void processPointer(T* ptr);
```

to disable calling `processPointer` with `void *` and `char *`, use:

```
template<>
void processPointer<void>(void *) = delete;

template<>
void processPointer<char>(char *) = delete;
```

you can also delete overloads with `const void *`, `const char *`, `const volatile void *`, and `const volatile char *` to be thorough.

template specializations must be declared at namespace scope, not class scope, so the C++98 way wouldn't compile:

```
class Widget {
public:
	...
	template<typename T>
	void processPointer(T* ptr)
	{ ... }
private:
	template<> // error!
	void processPointer<void>(void *); 
}
```

deleted functions however don't need a different access level and can be deleted from outside the class (namespace scope):
```
class Widget {
public:
	...
	template<typename T>
	void processPointer(T* ptr)
	{ ... }
};

template<>
void Widget::processPointer<void>(void *) = delete;
```

in summary, any functions may be deleted, including non-member functions and template instantiations

## Item 12: Declare overriding functions override

virtual function implementations in derived classes override the implementations of their base class counterparts

virtual function overriding makes it possible to invoke a derived class function through a base class interface

```
class Base {
public:
	virtual void doWork();
	...
};

class Derived: public Base {
public:
	virtual void doWork();
	...
};
// create base class pointer to derived class object
std::unique_ptr<Base> upb = std::make_unique<Derived>();
...
// derived class func is invoked
upb->doWork();
```

for overriding to occur, several reqs must be met:
- base class func must be virtual
- base + derived func names must be identical (except in the case of destructors)
- parameter types of base + derived class must be identical
- constness of base + derived functions must be identical
- return types + exception specifications of base + derived functions must be compatible

to constraints which were already part of C++98, C++11 adds 1 more:
- the functions' reference qualifiers must be identical. ref qualifiers make it possible to limit a function to lvalues only or rvalues only

```
class Widget {
public:
	void doWork() &; // this ver of doWork only applies when *this is an lvalue
	
	void doWork() &&; // only applies when *this is an rvalue
};

Widget makeWidget(); // returns rvalue
Widget w; // an lvalue

w.doWork(); // calls Widget::doWork &

makeWidget().doWork(); // calls Widget::doWork &&
```

if you declare an overriding derived class functions with override, it'll generate compiler errors if there's an error with the overriding

```
class Base {
public:
	virtual void mf1() const;
	virtual void mf2(int x);
	virtual void mf3() &;
	virtual void mf4() const;
};

class Derived: public {
public:
	virtual void mf1() const override;
	virtual void mf2(int x) override;
	virtual void mf3() & override;
	void mf4() const override;
}
```

member function reference qualifiers:
```
void doSomething(Wiget& w); // accepts only lvalue widgets

void doSomething(Widget&& w); // accepts only rvalue widgets
```

we can use reference qualifiers to overload functions for lvalue and rvalue objects so that they return lvalues and rvalues respectively:

```
class Widget {
public:
	using DataType = std::vector<double>;
	...
	// for lvalue widgets, return lvalues
	DataType& date() & { return values; }
	
	// for rvalue widgets, return rvalues
	DataType data() && { return std::move(values); } 
	...
	
private:
	DataType values;
};
```

## Item 13: Prefer const_iterators to iterators

in c++11, the container member functions `cbegin` and `cend` produce `const_iterators` for non-const containers + STL member functions that use iterators to identify positions (insert + erase) actually use const_iterators

```
std::vector<int> values;

auto it = std::find(values.cbegin(), values.cend(), 1983);

values.insert(it, 1998);
```

we could generalize the code into a find and insert template using C++14's non-member functions. C++11 doesn't have non-member functions for cbegin, cend, ... so this code isn't valid

```
template<typename C, typename V>
void findAndInsert(C& container, const V& targetVal, const V& insertVal) {
	using std::cbegin;
	using std::cend;
	
	auto it = std::find(cbegin(container), cend(container), targetVal);
	
	container.insert(it, insertVal);
}
```

in C++11, you can add non-member functions:

```
template<class C>
auto cbegin(const C& container)->decltype(std::begin(container)) {
	return std::begin(container);
}
```

## Item 15: Use constexpr whenever possible

constexpr objects: const objects that are known at compile time

values known during compilation may be placed in read-only memory

integral values that are constant + known during compilation can be used in contexts where C++ requires an integral constant expression (i.e. for array sizes, enum values, etc.)
- if you want to sue a variable for these, you want to declare it using constexpr

```
int sz; // non-constexpr variable
constexpr auto arraySize1 = sz; // error! sz's value not known @ compilation
std::array<int, sz> data1; // same error!
constexpr auto arraySize2 = 10; // fine, 10 is a compile-time constant
std::array<int, arraySize2> data2; // arraySize2 is constexpr
```

`const` doesn't offer the same guarantee as `constexpr` cause `const` objects need not be initialized with values known at compile-time
```
int sz;
const auto arraySize = sz; // fine, arraySize is a const copy of sz
std::array<int, arraySize> data; // error! arraySize's value not known @ compil
```

all `constexpr` objects are `const` but not all `const` objects are `constexpr`

if you want compilers to guarantee that a variable has a value that can be used in contexts requiring compile-time constants, use `constexpr`

constexpr functions produce compile-time constants when they're called with compile-time constants. if they're called with values not known until run-time, they produce run-time values

for example, we can write a constexpr pow function:

```
constexpr int pow(int base, int exp) noexcept {
	...
}
constexpr auto numConds = 5;
std::array<int, pow(3, numConds)> results;
```

constexpr in front of pow says: if base and exp are compile-time constants, pow's result may be used as a compile-time constant

this means pow can also be called in run-time constants:
```
auto base = readFromDB("base");
auto exp = readFromExp("exponent");

auto baseToExp = pow(base, exp);
```

restrictions on contents of constexpr functions differ between C++11 and C++14

in C++11, constexpr functions may contain no more than a single executable statement: return

```
constexpr int pow(int base, int exp) noexcept {
	return (exp == 0 ? 1 : base * pow(base, exp - 1));
}
```

in C++14, the restrictions on constexpr functions are looser, so this works:
```
constexpr int pow(int base, int exp) noexcept {
	auto result = 1;
	for (int i = 0; i < exp; ++i) result *= base;
	return result;
}
```

constexpr functions can only take + return literal types (types that can have values determined during compilation). this includes all built-in types except void, but user-defined types may be literal too with constexpr constructors + member functions

for example,

```
class Point {
public:
	constexpr Point(double xVal = 0, double yVal = 0) noexcept 
	: x(xVal), y(yVal) 
	{}
	
	constexpr double xValue() const noexcept { return x; }
	constexpr double yValue() const noexcept { return y; }
	
	void setX(double newX) noexcept { x = newX };
	void setY(double newY) noexcept { y = newY };

private:
	double x, y;
};
```

Point constructor can be declared constexpr because if the arguments passed to it are known during compilation, the value of the data members of the constructed Point can also be known at compile-time.

Points so initialized could be constexpr:
```
constexpr Point p1(9.4, 27.7);
constexpr Point p2(28.8, 5.3);
```

getters `xValue` + `yValue` can be `constexpr` because if they're invoked on a Point object with a value known during compilation, the value of data members x + y can be known at compile-time
```
constexpr Point midpoint(const Point& p1, const Point& p2) noexcept {
	return { (p1.xValue() + p2.xValue()) / 2,
			 (p1.yValue() + p2.yValue()) / 2 }; // call constexpr member funcs
}
// init constexpr object with result of constexpr function
constexpr auto mid = midpoint(p1, p2);
```

in C++11, setX + setY can't be constexpr b/c they modify the object they operate on, and they have void return types (not a literal type)
- both restrictions are lifted in C++14, so Point's setters can be constexpr

I/O statements are not permitted in constexpr functions

## Item 16: Make const member functions thread-safe

const member functions don't modify the object like this root-finding function for polynomials:

```
class Polynomial {
public:
	using RootsType = std::vector<double>;
	
	RootsType roots() const;
};
```

if we implement roots to return a cached value, it looks like this:
```
class Polynomial {
public:
	using RootsType = std::vector<double>;
	
	RootsType roots() const {
		std::lock_guard<std::mutex> g(m);
		
		if (!rootsAreValid) {
			... // compute roots + store in rootVals
			rootsAreValid = true;
		}
		return rootVals;
	}
private:
	mutable std::mutex m;
	mutable bool rootsAreValid{ false };
	mutable RootsType rootVals{};
};
```

the mutable keyword allows a data member to be modified by a const member function

imagine 2 threads call Roots on a polynomial object:

```
Polynomial p;

/* Thread 1 */                /* Thread 2 */
auto rootsOfP = p.roots();    auto valsGivingZero = p.roots();
```

this code has a data race without the mutex b/c both threads will try to read + write to rootsAreValid and rootVals

you can also use a std::atomic counter to count calls

```
class Point {
public:
	...
	double distanceFromOrigin() const noexcept {
		++callCount;
		return std::sqrt((x * x) + (y * y));
	}
private:
	mutable std::atomic<unsigned> callCount{ 0 };
	double x, y;
};
```

std::mutexes and std::atomics are move-only types, so the call count in Point means Point can only be moved, not copied

for a single variable or memory location requiring synchronization, use of a std::atomic is adequate, but once you get 2+ variables or memory locations that require syncs, use a mutex

```
class Widget {
public:
	int magicValue() const {
		std::lock_guard<std::mutex> guard(m);
		
		if (cacheValid) return cachedValue;
		else {
			auto val1 = expensiveComputation1();
			auto val2 = expensiveComputation2();
			cachedValue = val1 + val2;
			cacheValid = true;
			return cachedValue;
		}
	}
private:
	mutable std::mutex m;
	mutable int cachedValue;
	mutable bool cacheValid{ false }
}
```