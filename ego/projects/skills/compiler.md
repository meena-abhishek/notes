this will make you stronger programmer, and smarter about how you use data structures and algorithms in your work.

lex - lexical analyzer (lexer)
reads raw characters and groups them into tokens.
we define patterns using regular expressions.
`[0-9]+  return NUMBER;`

yacc - compiler-compiler (syntax analysis, parser)
checks if tokens follow grammar rules. produces a parse tree (AST).
`expr -> expr + expr`


you start off at the bottom with the program as raw source text, literally just a string of characters. each phase analyzes the program and transforms it to some higher-level representation where the semantics - what the author wants the computer to do - become more apparent.
##### The Parts of a Language
![[compiler_mountain.png]]


![[sample_statement.png|500]]
###### 1. Scanning
also known as lexing or lexical analysis.

a scanner (or lexer) takes in a linear stream of characters and chunks them together into a series of words, each word is known as a *token*.

tokens can be of single character `(`  or may be several characters long, like numbers(`123`), string literals(`"hi"`), identifiers(`min`).

![[tokens.png|500]]
###### 2. Parsing
this is where our syntax gets *grammar* - the abiility to compose larger expressions and statements out of smaller parts.

a parser takes the flat sequence of tokens and builds a tree structure that mirrors the nested nature of the grammar. parse tree, abstract syntax tree.

![[ast.png|500]]

incorrect grammar - `syntax error`
###### 3. Static analysis
at this point we know the syntactic structure of the code - things like which expressions are nested in which.

in an expression like `a+b`, we know that we are adding `a` and `b`, but we don't know what those name refers to. are they local variables? Global? Where are they defined?

the first bit of analysis that most languages do is called *binding* or *resolution*.
for each *identifier*, we find out where that name is defined and wire the two together. this is where *scope* comes into play - the region of source code where a certain name can be used to refer to a certain declaration.

this is where we can type check if language is statically typed. if `a` and `b` don't support a operation, we report a `type error`

this semantic insight needs to be stored somewhere :

- can be stored as *attributes* on the syntax tree itself - extra fields in the nodes that aren't initialized during parsing but gets filled in later. 

- we may store data in a lookup table known as *symbol table*. keys to this table are identifiers - names of variables and declarations. 

- transform tree into a new data structure that more directly expresses the semantics of the code.

###### 4. Intermediate representations
compiler can be seen as a pipeline where each stage's job is to organize the data representing the user's code in a way that makes the next stage simpler to implement.

the front end of the pipeline is specific to the source language the program in written in.
the back end is concerned with final architecture where the program will run.

in the middle, the code may be stored in some *intermediate representation (IR)* that isn't tightly tied to either the source or destination forms. IR acts as an interface between these two languages.

this lets you support multiple source languages and target platforms with less effort.

GCC - GNU compiler collection  supports many IRs eg. RTL - register transfer language, GIMPLE

###### 5. Optimization
once we understand what the user's program means, we are free to swap it out with a different program that has the same semantics but implements them more efficiently - we can optimize it.

a simple example is *constant folding:* if some expression always evaluates to the exact same value, we can do the evaluation at compile time and replace the code for the expression with its result. if the user typed in this:
`pennyArea = 3.14159 * (0.75 / 2) * (0.75 / 2);`
we could do all of that arithmetic in the compiler and change the code to: 
`pennyArea = 0.4417860938;`
###### 6. Code generation
last step is converting user's program to a form the machine can actually run. *generating code*, where code refers to the kind of primitive assembly-like instructions a CPU runs.

we need to decide whether do we generate instructions for a real cpu or a virtual one.
speaking chip's language means your compiler is tied to a specific architecture.
native code is lightening fast, but generating it is a lot of work.

we can make compilers produce virtual(hypothetical, idealized machine) machine code. *p-code* portable code or *bytecode*.

source -> bytecode -> VM/JIT -> native machine code
VM handles the architecture differences

eg. each arch has its own JVM implementation. so we can take any bytecode and use jvm of a specific arch. to get its machine code.

figuring out which parts of your compiler can be shared and which should be target-specific is an art.
###### 7. Virtual machine
VM, a program that emulates a hypothetical chip supporting your virtual architecture at runtime.
running bytecode in a VM is slower than translating it to native code ahead of time because every instruction must be simulated at runtime each time it executes.
implement your VM in, say C, and you can run your language on any platform that has a C compiler.

VM is manually simulating what a CPU would do, but in C.
this C is compiled to real architecture.
###### 8. Runtime
start up VM and load the program into that.
we usually need some services that our language provides while the program is running. eg. garbage collection, "instance of" tests

in a fully compiler language, the code implementing the runtime gets inserted directly into the resulting executable.
in Go, each compiled application has its own copy of Go's runtime directly embedded in it.
if the language is run inside a interpreter or VM, then the runtime lives there.
##### Shortcuts and Alternate Routes
###### 1. Single-pass compilers
some simple compilers produce output directly in the parser, without ever allocating any syntax trees or other IRs.

*syntax-directed translation* is a structured technique for building these all-at-once compilers. you associate an action with each piece of grammar, usually one that generates output code. then, whenever the parser matches that chunk of syntax, it executes the action, building the target code one rule at a time.

###### 2. Tree-walk interpreters
some programming languages begin executing code right after parsing it to an AST. to run the program, the interpreter traverses the syntax tree one branch and leaf at a time, evaluating each node as it goes.
slow.
###### 3. Transpilers
you write a front end for your language. then, in the back end, you produce a string of valid source code for some other language thats about as high level as yours. then, you use the existing compilation tools for that language.
that language is only the deployment target. eg. typescript compiles to javascript, but it adds several features like static typing, interfaces etc.
###### 4. Just-in-time compilation
you might not know what architecture your end user's machine supports.
when the program is loaded on end user's machine - either from the source or as bytecode - you compile it to native code for the architecture their computer supports.
##### Compiler and Interpreters
*compiling* is an implementation technique that involves translating a source language to some other - usually lower level - form.
generating bytecode, machine code, transpiling to other language are all compiling.

when we say a language implementation *"is a compiler"*, we mean it translates source code to some other form but doesn't execute it. user need to run it.

when we say a language implementation *"is a interpreter"*, we mean it takes in source code and executes it immediately. it runs programs "from source".

CPython is an interpreter, and it has a compiler.
it compiles source to bytecode and then interprets it in VM.

