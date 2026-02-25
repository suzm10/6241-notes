
compiler: software system that translates program into form that can be executed by computer

## 1.1 Language Processors

compiler: program that reads program in 1 language + translates it into an equivalent program in another language (target program)
- report any errors that it detects in source program during translation process

interpreter: instead of producing target program, interpreter executes operations specified in source program on inputs supplied by user

preprocessor: collects source programs + expands macros into source-language statements

modified source program -> compiler -> assembly-language program -> assembler -> relocatable machine code -> linker/loader -> target machine code

large programs usually compiled in pieces, so relocatable machine code may have to be linked together with other relocatable obj files + lib files

linker: resolved external memory addresses where the code in 1 file may refer to a location in another file

loader: puts together all executable object files into memory for execution


## 1.2 The Structure of a Compiler

compiler does analysis + synthesis

analysis: breaks up source program into pieces + imposes grammatical structure on them
- uses structure to create IR of source program
- if it detects program is ill-formed or syntactically unsound
- creates symbol table
- IR + symbol table passed to synthesis
- compiler front-end

synthesis: 
- constructs desired target program from IR + symbol table

some compilers have machine-independent optimization phase between front-end and back-end

## 1.2.1 Lexical Analysis

1st phase: lexical analysis/scanning: reads stream of characters making up source program+ groups the characters into lexemes 

for each lexeme, lexical analyzer produces a token: <token-name, attribute-value>
- token-name: abstract symbol used during syntax analysis
- attribute-value: points to symbol table entry for this token, symbol table contains info about identifier such as name and type

assignment statement gets converted to sequence of tokens by lexical analysis


## 1.2.2 Syntax Analysis

2nd phase: syntax analysis/parsing, parser uses first components of tokens produced by lexical analyzer to create a tree-like IR that depicts grammatical structure of token stream

syntax tree: intermediate node = operation, children of node = arguments

## 1.2.3 Semantic Analysis

- semantic analyzer: uses syntax tree + info in symbol table to check source program for semantic consistency with language definition
- gathers type info + saves it in either syntax tree or symbol table for use during intermediate-code generation
- does type checking: each operator has matching operands
- language spec may permit some coercions: type conversions (ex: int -> float)

## 1.2.4 Intermediate Code Generation

- syntax trees are a common intermediate form used during syntax and semantic analysis
- after syntax + semantic analysis, compilers generate explicit low-level or machine-like IR
	- program for abstract machine
- three-address code: sequence of assembly-like instructions with 3 operands

## 1.2.5 Code Optimization

machine-independent code-optimization phase improves intermediate code so that better code is produced (faster, less power, shorter, etc.)

## 1.2.6 Code Generation

code generator takes as input the IR of source program + maps it into the target language

if target language is machine code, registers or memory locations are selected for each var used by program

## 1.2.7 Symbol-Table Management

compiler records variable names used in source program + collects info on attributed (storage, type, scope, arguments + method of passing each arg for procedure names)

## 1.2.8 The Grouping of Phases into Passes

activities from several phases may be grouped together into a pass that reads an input file + writes an output file

front-end phases of lexical, syntax, semantic analysis, + intermediate code generation may be grouped together into a pass

back-end pass: code generation for a target machine

## 1.2.9 Compiler-Construction Tools

# 1.3 The Evolution of Programming Languages

## 1.3.1 The Move to Higher-level Languages

imperative language: specifies how a computation is to be done
declarative language: specifies what computation is to be done

## 1.3.2 Impacts on Compilers

# 1.4 The Science of Building a Compiler

## 1.4.1 Modeling in Compiler Design and Implementation

fundamental models are finite-state machines + regular expressions + context-free grammars (describe syntactic structure of programming languages like nesting parentheses or control constructs)

## 1.4.2 The Science of Code Optimization

# 1.5 Applications of Compiler Technology

## 1.5.1 Implementation of High-Level Programming Languages

data-flow optimizations: analyze flow of data through program + remove redundancies across these constructs

key ideas behind object orientation: data abstraction + inheritance of properties

object oriented programs consist of many smaller methods
- procedural inlining is a useful optimization
- optimizations to speed up virtual method dispatches

dynamic optimization: info extracted dynamically at run-time and used to produce better optimized code

## 1.5.2 Optimizations for Computer Architectures

- high-performance systems take advantage of parallelism + memory hierarchies

### Parallelism

hardware dynamically checks for dependencies in sequential instruction stream + issues them in parallel whenever possible

compilers can rearrange instructions to make instruction-level parallelism more effective

### Memory Hierarchies

performance of system limited by both speed of processor + performance of memory subsystem

using registers effectively is most important in optimizing

can improve effectiveness of memory hierarchy by changing layout of data or order of instructions that access data


## 1.5.3 Design of New Computer Architectures

### RISC

compilers influenced design of RISC

### Specialized Architectures

for embedded systems

## 1.5.4 Program Translations

can translate between different types of languages

### Binary Translation

compiler technology can translate binary code for one machine to that of another, allowing machine to run programs originally compiled for another instruction set

### Hardware Synthesis

hardware designs described at RTL (register transfer level) where variables represent registers + expressions represent combinational logic

hardware synthesis tools translate RTL descriptions automatically into gates which are mapped to transistors + eventually to physical layout

### Database Query Interpreters

SQL to search databases 
### Compiled Simulation

compile design to produce machine code that simulates design natively

## 1.5.5 Software Productivity Tools

data-flow analysis can find errors statically (before program is run), can find errors along all possible execution paths

### Type Checking

### Bounds Checking

data-flow analysis used to eliminate redundant range checks + locate buffer overflow

### Memory-Management Tools

garbage collection automatically gets rid of all memory-management errors but trades efficiency

# 1.6 Programming Language Basics

## 1.6.1 The Static/Dynamic Distinction

- static policy: issue decided at compile time
- dynamic policy: requires decision at run-time

scope of a declaration of x: region of program in which uses of x refer to this declaration

language uses static/lexical scope if it's possible to determine scope of declaration by lookinh only at the program

dynamic scope— as the program runs, the same use of x could refer to any of several different declarations of x

## 1.6.2 Environments and States

names are associated with locations in memory (the store) + with values

two-stage mapping from names to values
environment: mapping from names to locations in store (l-values)/variables
state: mapping from locations in store to their values, l-values to r-values

most binding of names to locations is dynamic. some declarations such as global i can be given permanent location in store as compiler generates object code

binding of locations to values is generally dynamic

identifier: string of characters that refers to an entity
- all identifiers are names but not all names are identifiers cause names can also be expressions (x.y is a name but not an identifier)
- variable: particular location of store
	- same identifier can be declared more than once with new variables

## 1.6.3 Static Scope and Block Structure

## 1.6.4 Explicit Access Control

public, private, + protected keywords provide control over access to member names in a superclass— support encapsulation by restricting access

## 1.6.5 Dynamic Scope

declarations: tell us about types of things
- int i
definitions: tell us about their values
- i = 1

## 1.6.6 Parameter Passing Mechanisms

## 1.6.7 Aliasing



