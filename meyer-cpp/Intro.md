

rvalues = temp objects returned from functions
lvalues = objects you can refer to, either by name or by following a pointer l-value reference
- can take its address

when an object is initialized with another object of the same type, the new object is a copy of the initializing object, even if it was created via move constructor

```
void someFunc(Widget w); // someFunc's parameter w is passed by value
Widget wid; 
someFunc(wid); // w is a copy of wid that's created via copy construction
someFunc(std::move(wid)) // w is a copy of wid that's created via move construction
```

copies of r-values are generally move constructed while copies of l-values are usually copy constructed

in a function call, the expressions passed at the call site are the function's arguments. args are used to initialize function's parameters
- parameters are l-values
- args with which params are initialized may be l-values or r-values

basic exception safety guarantee: even if an exception is thrown, program invariants remain intact (no data structures are corrupted) + no resources are leaked

strong exception safety guarantee: even if an exception arises, state of program remains as it was prior to the call

closures = func objects created through lambdas

raw pointers = built-in pointers like those returned from new
