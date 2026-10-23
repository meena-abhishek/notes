`std::cout` is buffered

The opposite of buffered output is unbuffered output. With unbuffered output, each individual output request is sent directly to the output device.

Writing data to a buffer is typically fast, whereas transferring a batch of data to an output device is comparatively slow. Buffering can significantly increase performance by batching multiple output requests together to minimize the number of times output has to be sent to the output device.

`std::end` vs `\n`

Using `std::endl` is often inefficient, as it actually does two jobs: it outputs a newline (moving the cursor to the next line of the console), and it flushes the buffer (which is slow). If we output multiple lines of text ending with `std::endl`, we will get multiple flushes, which is slow and probably unnecessary.

When outputting text to the console, we typically don’t need to explicitly flush the buffer ourselves. C++’s output system is designed to self-flush periodically, and it’s both simpler and more efficient to let it flush itself.

To output a newline without flushing the output buffer, we use `\n` (inside either single or double quotes), which is a special symbol that the compiler interprets as a newline character. `\n` moves the cursor to the next line of the console without causing a flush, so it will typically perform better. `\n` is also more concise to type and can be embedded into existing double-quoted text.

*brace initialization*
```cpp
int x{};    // x = 0 (brace initialization)
int x = 0;  // x = 0 (copy initialization)
int x(0);   // x = 0 (direct initialization)
int x = {}; // x = 0
```
helps prevent uninitialized variables.

Adding data to the end of a buffer and removing it from the front of a buffer ensures data is processed in the same order in which it was added. This is sometimes called FIFO (first in, first out).

Each line of input data in the input buffer is terminated by a `'\n'` character.

Run #2: When `std::cin >> x` is encountered, the program will wait for input. Enter `4 5`. The input `4 5\n` goes into the input buffer, but only the `4` is extracted to variable `x` (extraction stops at the space).

When `std::cin >> y` is encountered, the program will _not_ wait for input. Instead, the `5` that is still in the input buffer is extracted to variable `y`. The program then prints `You entered 4 and 5`.

Note that in run 2, the program didn’t wait for the user to enter additional input when extracting to variable `y` because there was already prior input in the input buffer that could be used.

`std::cin` is buffered because it allows us to separate the entering of input from the extract of input. We can enter input once and then perform multiple extraction requests on it.

Here’s a simplified view of how operator `>>` works for input.

1. If `std::cin` is not in a good state (e.g. the prior extraction failed and `std::cin` has not yet been cleared), no extraction is attempted, and the extraction process aborts immediately.
2. Leading whitespace characters (spaces, tabs, and newlines at the front of the buffer) are discarded from the input buffer. This will discard an unextracted newline character remaining from a prior line of input.
3. If the input buffer is now empty, operator `>>` will wait for the user to enter more data. Any leading whitespace is discarded from the entered data.
4. operator `>>` then extracts as many consecutive characters as it can, until it encounters either a newline character (representing the end of the line of input) or a character that is not valid for the variable being extracted to.

The result of the extraction process is as follows:

- If the extraction aborted in step 1, then no extraction attempt occurred. Nothing else happens.
- If any characters were extracted in step 4 above, extraction is a success. The extracted characters are converted into a value that is then copy-assigned to the variable.
- If no characters could be extracted in step 4 above, extraction has failed. The object being extracted to is copy-assigned the value `0` (as of C++11), and any future extractions will immediately fail (until `std::cin` is cleared).

**constants**

- **Named constants** are constant values that are associated with an identifier. These are also sometimes called **symbolic constants**.
- **Literal constants** are constant values that are not associated with an identifier.

```cpp
const double gravity { 9.8 };  // preferred use of const before type
int const sidesInSquare { 4 }; // "east const" style, okay but not preferred
```
```cpp
const variables must be initialized
```
```cpp
 const int constAge { age }; // initialize const variable using non-const value
```

two functions can have the same name. This is called function overloading, provided their parameter lists differ. same parameter's but different return type is not allowed.

Don’t use `const` for value parameters.
Later in this tutorial series, we’ll talk about two other ways to pass arguments to functions: pass by reference, and pass by address. When using either of these methods, proper use of `const` is important.

For other types (which we’ll cover later), there is typically little point in returning const objects by value, because they are temporary copies that will be destroyed anyway. Returning a const value can also impede certain kinds of compiler optimizations (involving move semantics), which can result in lower performance.
Don’t use `const` when returning by value.

Every moving part in a system increases complexity and the risk of defect or failure. Non-constant variables are moving parts, while constant variables are not.
Make variables constant whenever possible. Exception cases include by-value function parameters and by-value return types, which should generally not be made constant.

Second, it is often harder to debug code using macros. Although your source code will have the macro’s name, the compiler and debugger never see the macro because it has already been replaced before they run. Many debuggers are unable to inspect a macro’s value, and often have limited capabilities when working with macros.

Instead of redefining these every time they are needed, it’s better to declare them once in a central location and use them wherever needed

- A **cv-unqualified** type is a type with no type qualifiers (e.g. `int`).
- A **cv-qualified** type is a type with one or more type qualifiers applied (e.g. `const int`).
- A **possibly cv-qualified** type is a type that may be cv-unqualified or cv-qualified. volatile

**references**

In C++, a **reference** is an alias for an existing object. Once a reference has been defined, any operation on the reference is applied to the object being referenced. This means we can use a reference to read or modify the object being referenced.

You can also create references to functions, though this is done less often.
Modern C++ contains two types of references: lvalue references, and rvalue references

Value categories (lvalues and rvalues) :

we defined an expression as “a combination of literals, variables, operators, and function calls that can be executed to produce a singular value”.

Increment/decrement operators, and side effects:

|                                    |     |     |                                                |
| ---------------------------------- | --- | --- | ---------------------------------------------- |
| Prefix increment (pre-increment)   | ++  | ++x | Increment x, then return x                     |
| Prefix decrement (pre-decrement)   | ––  | ––x | Decrement x, then return x                     |
| Postfix increment (post-increment) | ++  | x++ | Copy x, then increment x, then return the copy |
| Postfix decrement (post-decrement) | ––  | x–– | Copy x, then decrement x, then return the copy |
|                                    |     |     |                                                |
Favor the prefix versions, as they are more performant and less likely to cause surprises.

The assignment operators, prefix operator, and postfix operator have side effects that permanently change the value of an object.  
Other operators (such as the arithmetic operators) return a value, and do not modify their operands.
```cpp
   int x { 5 };
    int value{ add(x, ++x) }; // undefined behavior: is this 5 + 6, or 6 + 6?
    // It depends on what order your compiler evaluates the function arguments in
```
The C++ standard intentionally does not define these things so that compilers can do whatever is most natural (and thus most performant) for a given architecture.

In many cases, C++ also does not specify when the side effects of operators must be applied. This can lead to undefined behavior in cases where an object with a side effect applied is used more than once in the same statement.

For example, the expression `x + ++x` is unspecified behavior. When `x` is initialized to `1`, Visual Studio and GCC evaluate this as `2 + 2`, and Clang evaluates it as `1 + 2`! This is due to differences in when the compilers apply the side effect of incrementing `x`.

Even when the C++ standard does make it clear how things should be evaluated, historically this has been an area where there have been many compiler bugs. These problems can generally _all_ be avoided by ensuring that any variable that has a side-effect applied is used no more than once in a given statement.

The type of an expression is equivalent to the type of the value, object, or function that results from the evaluated expression.
```cpp
int main()
{
    auto v1 { 12 / 4 }; // int / int => int
    auto v2 { 12.0 / 4 }; // double / int => double

    return 0;
}
```

In the above program, the `print(int)` function is expecting an `int` parameter. However, the type of the expression we’re passing in (the string literal `"foo"`) does not match, and no conversion can be found. So a compile error results.
Note that the type of an expression must be determinable at compile time (otherwise type checking and type deduction wouldn’t work) -- however, the value of an expression may be determined at either compile time (if the expression is constexpr) or runtime (if the expression is not constexpr).

One of these assignment statements is valid (assigning value `5` to variable `x`) and one is not (what would it mean to assign the value of `x` to the literal value `5`?). So how does the compiler know which expressions can legally appear on either side of an assignment statement?

The answer lies in the second property of expressions: the `value category`. The **value category** of an expression (or subexpression) indicates whether an expression resolves to a value, a function, or an object of some kind.

Prior to C++11, there were only two possible value categories: `lvalue` and `rvalue`.

In C++11, three additional value categories (`glvalue`, `prvalue`, and `xvalue`) were added to support a new feature called `move semantics`.

An **lvalue** (pronounced “ell-value”, short for “left value” or “locator value”, and sometimes written as “l-value”) is an expression that evaluates to an identifiable object or function (or bit-field).

The term “identity” is used by the C++ standard, but is not well-defined. An entity (such as an object or function) that has an identity can be differentiated from other similar entities (typically by comparing the addresses of the entity).

Entities with identities can be accessed via an identifier, reference, or pointer, and typically have a lifetime longer than a single expression or statement.

An **rvalue** (pronounced “arr-value”, short for “right value”, and sometimes written as `r-value`) is an expression that is not an lvalue. Rvalue expressions evaluate to a value. Commonly seen rvalues include literals (except C-style string literals, which are lvalues) and the return value of functions and operators that return by value. Rvalues aren’t identifiable (meaning they have to be used immediately), and only exist within the scope of the expression in which they are used.

```cpp
int return5()
{
    return 5;
}

int main()
{
    int x{ 5 }; // 5 is an rvalue expression
    const double d{ 1.2 }; // 1.2 is an rvalue expression

    int y { x }; // x is a modifiable lvalue expression
    const double e { d }; // d is a non-modifiable lvalue expression
    int z { return5() }; // return5() is an rvalue expression (since the result is returned by value)

    int w { x + 1 }; // x + 1 is an rvalue expression
    int q { static_cast<int>(d) }; // the result of static casting d to an int is an rvalue expression

    return 0;
}
```
You may be wondering why `return5()`, `x + 1`, and `static_cast<int>(d)` are rvalues: the answer is because these expressions produce temporary values that are not identifiable objects.

Lvalue expressions evaluate to an identifiable object.  
Rvalue expressions evaluate to a value.

Now we can answer the question about why `x = 5` is valid but `5 = x` is not: an assignment operation requires its left operand to be a modifiable lvalue expression. The latter assignment (`5 = x`) fails because the left operand expression `5` is an rvalue, not a modifiable lvalue.

```cpp
int main()
{
    int x{ 1 };
    int y{ 2 };

    x = y; // y is not an rvalue, but this is legal

    return 0;
}
```
An lvalue will implicitly convert to an rvalue. This means an lvalue can be used anywhere an rvalue is expected.  
An rvalue, on the other hand, will not implicitly convert to an lvalue.

In this statement, the variable `x` is being used in two different contexts. On the left side of the assignment operator (where an lvalue expression is required), `x` is an lvalue expression that evaluates to variable `x`. On the right side of the assignment operator, `x` undergoes an lvalue-to-rvalue conversion and is then evaluated so that its (`2`) can be used as the left operand of `operator+`. `operator+` returns the rvalue expression `3`, which is then used as the right operand for the assignment.

```cpp
// regular types
int        // a normal int type (not an reference)
int&       // an lvalue reference to an int object
double&    // an lvalue reference to a double object
const int& // an lvalue reference to a const int object
```
A type that specifies a reference (e.g. `int&`) is called a **reference type**. The type that can be referenced (e.g. `int`) is called the **referenced type**.

When defining a reference, place the ampersand next to the type (not the reference variable’s name).
For those of you already familiar with pointers, the ampersand in this context does not mean “address of”, it means “lvalue reference to”.
```cpp
    ref = 7; // the object being referenced (x) now has value 7
```
Much like constants, all references must be initialized
```cpp
int main()
{
    int x { 5 };
    int& ref { x };         // okay: non-const lvalue reference bound to a modifiable lvalue

    const int y { 5 };
    int& invalidRef { y };  // invalid: non-const lvalue reference can't bind to a non-modifiable lvalue
    int& invalidRef2 { 0 }; // invalid: non-const lvalue reference can't bind to an rvalue

    return 0;
}
```
If you try to bind a reference to an object that does not match its referenced type, the compiler will try to implicitly convert the object to the referenced type and then bind the reference to that.

```cpp
int main()
{
    int x { 5 };
    int& ref { x };            // okay: referenced type (int) matches type of initializer

    double d { 6.0 };
    int& invalidRef { d };     // invalid: conversion of double to int is narrowing conversion, disallowed by list initialization
    double& invalidRef2 { x }; // invalid: non-const lvalue reference can't bind to rvalue (result of converting x to double)

    return 0;
}
```
Once initialized, a reference in C++ cannot be **reseated**, meaning it cannot be changed to reference another object.

With one exception (that we’ll cover next lesson), the lifetime of a reference and the lifetime of its referent are independent. In other words, both of the following are true:

- A reference can be destroyed before the object it is referencing.
- The object being referenced can be destroyed before the reference.

When a reference is destroyed before the referent, the referent is not impacted. The following program demonstrates this:
```cpp

int main()
{
    int x { 5 };

    {
        int& ref { x };   // ref is a reference to x
        std::cout << ref << '\n'; // prints value of ref (5)
    } // ref is destroyed here -- x is unaware of this

    std::cout << x << '\n'; // prints value of x (5)

    return 0;
} // x destroyed here
```
When an object being referenced is destroyed before a reference to it, the reference is left referencing an object that no longer exists. Such a reference is called a **dangling reference**. Accessing a dangling reference leads to undefined behavior.

Perhaps surprisingly, references are not objects in C++. A reference is not required to exist or occupy storage. If possible, the compiler will optimize references away by replacing all occurrences of a reference with the referent. However, this isn’t always possible, and in such cases, references may require storage.

you can’t have a reference to a reference, since an lvalue reference must reference an identifiable object

Consider the following variables:

```cpp
int var{};
int& ref1{ var };  // an lvalue reference bound to var
int& ref2{ ref1 }; // an lvalue reference bound to var
```

Because `ref2` (a reference) is initialized with `ref1` (a reference), you might be tempted to conclude that `ref2` is a reference to a reference. It is not. Because `ref1` is a reference to `var`, when used in an expression (such as an initializer), `ref1` evaluates to `var`. So `ref2` is just a normal lvalue reference (as indicated by its type `int&`), bound to `var`.

A reference to a reference (to an `int`) would have syntax `int&&` -- but since C++ doesn’t support references to references, this syntax was repurposed in C++11 to indicate an rvalue reference (which we cover in lesson [22.2 -- R-value references](https://www.learncpp.com/cpp-tutorial/rvalue-references/)).

```cpp

int main()
{
    const int x { 5 };    // x is a non-modifiable lvalue
    const int& ref { x }; // okay: ref is a an lvalue reference to a const value

    std::cout << ref << '\n'; // okay: we can access the const object
    ref = 6;                  // error: we can not modify an object through a const reference

    return 0;
}
```
Lvalue references to const can also bind to modifiable lvalues. In such a case, the object being referenced is treated as const when accessed through the reference (even though the underlying object is non-const):
```cpp
#include <iostream>

int main()
{
    int x { 5 };          // x is a modifiable lvalue
    const int& ref { x }; // okay: we can bind a const reference to a modifiable lvalue

    std::cout << ref << '\n'; // okay: we can access the object through our const reference
    ref = 7;                  // error: we can not modify an object through a const reference

    x = 6;                // okay: x is a modifiable lvalue, we can still modify it through the original identifier

    return 0;
}
```
Perhaps surprisingly, lvalues references to const can also bind to rvalues:

```cpp
#include <iostream>

int main()
{
    const int& ref { 5 }; // okay: 5 is an rvalue

    std::cout << ref << '\n'; // prints 5

    return 0;
}
```

When this happens, a temporary object is created and initialized with the rvalue, and the reference to const is bound to that temporary object.

Warning

We normally assume that a reference is identical to the object it is bound to -- but this assumption is broken when a reference is bound to a temporary copy of the object or a temporary resulting from the conversion of the object instead. Any modifications subsequently made to the original object will not be seen by the reference (as it is referencing a different object), and vice-versa.

Here’s a silly example showing this:

```cpp
#include <iostream>

int main()
{
    short bombs { 1 };         // I can has bomb! (note: type is short)

    const int& you { bombs };  // You can has bomb too (note: type is int&)
    --bombs;                   // Bomb all gone

    if (you)                   // You still has?
    {
        std::cout << "Bombs away!  Goodbye, cruel world.\n"; // Para bailar la bomba
    }

    return 0;
}
```

In the above example, `bombs` is a `short` and `you` is a `const int&`. Because `you` can only bind to an `int` object, when `you` is initialized with `bombs`, the compiler will implicitly convert `bombs` to an `int`, which results in the creation of a temporary `int` object (with value `1`). `you` ends up bound to this temporary object rather than `bombs`.

When `bombs` is decremented, `you` is not affected because it is referencing a different object. So although we expect `if (you)` to evaluate to `false`, it actually evaluates to `true`.

If you would stop blowing up the world, that would be great.

To avoid dangling references in such cases, C++ has a special rule: When a const lvalue reference is _directly_ bound to a temporary object, the lifetime of the temporary object is extended to match the lifetime of the reference.

Lvalue references can only bind to modifiable lvalues.

Lvalue references to const can bind to modifiable lvalues, non-modifiable lvalues, and rvalues. This makes them a much more flexible type of reference.

When applied to a reference, `constexpr` allows the reference to be used in a constant expression. Constexpr references have a particular limitation: they can only be bound to objects with static duration (either globals or static locals). This is because the compiler knows where static objects will be instantiated in memory, so it can treat that address as a compile-time constant.

A constexpr reference cannot bind to a (non-static) local variable. This is because the address of local variables is not known until the function they are defined within is actually called.
```cpp
int g_x { 5 };

int main()
{
    [[maybe_unused]] constexpr int& ref1 { g_x }; // ok, can bind to global

    static int s_x { 6 };
    [[maybe_unused]] constexpr int& ref2 { s_x }; // ok, can bind to static local

    int x { 6 };
    [[maybe_unused]] constexpr int& ref3 { x }; // compile error: can't bind to non-static object

    return 0;
}
```
Most of the types provided by the standard library (such as `std::string`) are class types. Class types are usually expensive to copy. Whenever possible, we want to avoid making unnecessary copies of objects that are expensive to copy, especially when we will destroy those copies almost immediately.

Pass by reference allows us to pass arguments to a function without making copies of those arguments each time the function is called.
Passing values by reference to non-const allows us to write functions that modify the value of arguments passed in.

```cpp
    const int z { 5 };
    printValue(z); // error: z is a non-modifiable lvalue
```

Favor passing by const reference over passing by non-const reference unless you have a specific reason to do otherwise (e.g. the function needs to change the value of an argument).

With pass by reference, ensure the type of the argument matches the type of the reference, or it will result in an unexpected (and possibly expensive) conversion.

Second, we need to consider the cost of using the function parameter. When setting up a function call, the compiler may be able to optimize by placing a reference or copy of a passed-by-value argument (if it is small in size) into a CPU register (which is fast to access) rather than into RAM (which is slower to access).
Each time a value parameter is used, the running program can directly access the storage location (CPU register or RAM) of the copied argument. However, when a reference parameter is used, there is usually an extra step. The running program must first directly access the storage location (CPU register or RAM) allocated to the reference, in order to determine which object is being referenced. Only then can it access the storage location of the referenced object (in RAM).
Therefore, each use of a value parameter is a single CPU register or RAM access, whereas each use of a reference parameter is a single CPU register or RAM access plus a second RAM access.

Third, the compiler can sometimes optimize code that uses pass by value more effectively than code that uses pass by reference. In particular, optimizers have to be conservative when there is any chance of **aliasing** (when two or more pointers or references can access the same object). Because pass by value results in the copy of argument values, there is no chance for aliasing to occur, allowing optimizers to be more aggressive.

We can now answer these question of why we don’t pass everything by reference:

- For objects that are cheap to copy, the cost of copying is similar to the cost of binding, but accessing the objects is faster and the compiler is likely to be able to optimize better.
- For objects that are expensive to copy, the cost of the copy dominates other performance considerations.

An object of type T is cheap to copy if `sizeof(T) <= 2 * sizeof(void*)` and has no additional setup costs.

Prefer passing strings using `std::string_view` (by value) instead of `const std::string&`, unless your function calls other functions that require C-style strings or `std::string` parameters.

Although you generally should not declare multiple variables on a single line, if you do, the asterisk has to be included with each variable.
```cpp
int main()
{
    int x{ 5 };

    int* ptr;        // an uninitialized pointer (holds a garbage address)
    int* ptr2{};     // a null pointer (we'll discuss these in the next lesson)
    int* ptr3{ &x }; // a pointer initialized with the address of variable x

    return 0;
}
```
When we use a pointer without a dereference (`ptr`), we are accessing the address held by the pointer. Modifying this (`ptr = &y`) changes what the pointer is pointing at.

When we dereference a pointer (`*ptr`), we are accessing the object being pointed at. Modifying this (`*ptr = 6;`) changes the value of the object being pointed at.

There are some other differences between pointers and references worth mentioning:

- References must be initialized, pointers are not required to be initialized (but should be).
- References are not objects, pointers are.
- References can not be reseated (changed to reference something else), pointers can change what they are pointing at.
- References must always be bound to an object, pointers can point to nothing (we’ll see an example of this in the next lesson).
- References are “safe” (outside of dangling references), pointers are inherently dangerous (we’ll also discuss this in the next lesson).

It’s worth noting that the address-of operator (&) doesn’t return the address of its operand as a literal (as C++ doesn’t support address literals). Instead, it returns a pointer to the operand (whose value is the address of the operand). In other words, given variable `int x`, `&x` returns an `int*` holding the address of `x`.

Objects returned by reference must live beyond the scope of the function returning the reference, or a dangling reference will result. Never return a (non-static) local variable or temporary by reference.

Lifetime extension doesn’t work across function boundaries

Let’s take a look at an example where we return a temporary by reference:

```cpp
#include <iostream>

const int& returnByConstReference()
{
    return 5; // returns const reference to temporary object
}

int main()
{
    const int& ref { returnByConstReference() };

    std::cout << ref; // undefined behavior

    return 0;
}
```

In the above program, `returnByConstReference()` is returning an integer literal, but the return type of the function is `const int&`. This results in the creation and return of a temporary reference bound to a temporary object holding value 5. This returned reference is copied into a temporary reference in the scope of the caller. The temporary object then goes out of scope, leaving the temporary reference in the scope of the caller dangling.

By the time the temporary reference in the scope of the caller is bound to const reference variable `ref` (in `main()`), it is too late to extend the lifetime of the temporary object -- as it has already been destroyed. Thus `ref` is a dangling reference, and use of the value of `ref` will result in undefined behavior.

Here’s a less obvious example that similarly doesn’t work:

```cpp
#include <iostream>

const int& returnByConstReference(const int& ref)
{
    return ref;
}

int main()
{
    // case 1: direct binding
    const int& ref1 { 5 }; // extends lifetime
    std::cout << ref1 << '\n'; // okay

    // case 2: indirect binding
    const int& ref2 { returnByConstReference(5) }; // binds to dangling reference
    std::cout << ref2 << '\n'; // undefined behavior

    return 0;
}
```

In case 2, a temporary object is created to hold value `5`, which function parameter `ref` binds to. The function just returns this reference back to the caller, which then uses the reference to initialize `ref2`. Because this is not a direct binding to the temporary object (as the refrence was bounced through a function), lifetime extension doesn’t apply. This leaves `ref2` dangling, and its subsequent use is undefined behavior.

If a function returns a reference, and that reference is used to initialize or assign to a non-reference variable, the return value will be copied (as if it had been returned by value).

C++11 adds a new type of reference called an r-value reference. An r-value reference is a reference that is designed to be initialized with an r-value (only). While an l-value reference is created using a single ampersand, an r-value reference is created using a double ampersand:

R-value references have two properties that are useful. First, r-value references extend the lifespan of the object they are initialized with to the lifespan of the r-value reference (l-value references to const objects can do this too). Second, non-const r-value references allow you to modify the r-value!

The answer might surprise you. This calls `fun(const int&)`.

Although variable `ref` has type `int&&`, when used in an expression it is an lvalue (as are all named variables). The type of an object and its value category are independent.

You already know that literal `5` is an rvalue of type `int`, and `int x` is an lvalue of type `int`. Similarly, `int&& ref` is an lvalue of type `int&&`.

So not only does `fun(ref)` call `fun(const int&)`, it does not even match `fun(int&&)`, as rvalue references can’t bind to lvalues.

Recapping the binding combinations:

- Non-const l-value references can only bind to non-const l-values.
- Const l-value references can bind to non-const l-values, const l-values, and r-values.
- R-value references can only bind to r-values.

`decltype()` determines the type of an expression without evaluating it.

A string literal is not a `std::string`, but a `const char[]` . If the compiler was to choose `f(const std::string&)`, it would have to go through a user defined conversion and create a temporary `std::string`. Instead, it prefers `f(const void*)`, which requires no user defined conversion.

**function overloading**
Because the parameter types of these functions differ, the compiler is able to differentiate these functions, and will treat them as separate functions that just happen to share a name.
```cpp
void print(int);
void print(const int); // not differentiated from print(int)
```
This was an intentional choice, as it ensures the behavior of a function call can be determined independently from the rest of the expression, making understanding complex expressions much simpler. Put another way, we can always determine which version of a function will be called based solely on the arguments in the function call. If return values were used for differentiation, then we wouldn’t have an easy syntactic way to tell which overload of a function was being called -- we’d also have to understand how the return value was being used, which requires a lot more analysis.

When the compiler compiles a function, it performs **name mangling**, which means the compiled name of the function is altered (“mangled”) based on various criteria, such as the number and type of parameters, so that the linker has unique names to work with.

For example, a function with prototype `int fcn()` might compile to mangled name `__fcn_v`, whereas `int fcn(int)` might compile to mangled name `__fcn_i`. So while in the source code, the two overloaded functions share the name `fcn()`, in compiled code, the mangled names are unique (`__fcn_v` vs `__fcn_i`).

There is no standardization on how names should be mangled, so different compilers will produce different mangled names.

ambiguous match, if multiple matches found

promotion -> conversion -> user-defined conversion

**templates**

Having to create overloaded functions with the same implementation for each set of parameter types we want to support is a maintenance headache, a recipe for errors, and a clear violation of the DRY (don’t repeat yourself) principle. There’s a less-obvious challenge here as well: a programmer who wishes to use the `max` function may wish to call it with an argument type that the author of the `max` did not anticipate (and thus did not write an overloaded function for).

What we are really missing is some way to write a single version of `max` that can work with arguments of any type (even types that may not have been anticipated when the code for `max` was written). Normal functions are simply not up to the task here. Fortunately, C++ supports another feature that was designed specifically to solve this kind of problem.

Instead of manually creating a bunch of mostly-identical functions or classes (one for each set of different types), we instead create a single _template_. Just like a normal definition, a **template** definition describes what a function or class looks like. Unlike a normal definition (where all types must be specified), in a template we can use one or more placeholder types. A placeholder type represents some type that is not known at the time the template is defined, but that will be provided later (when the template is used).

Once a template is defined, the compiler can use the template to generate as many overloaded functions (or classes) as needed, each using different actual types!

The end result is the same -- we end up with a bunch of mostly-identical functions or classes (one for each set of different types). But we only have to create and maintain a single template, and the compiler does all the hard work to create the rest for us.

A **function template** is a function-like definition that is used to generate one or more overloaded functions, each with a different set of actual types. This is what will allow us to create functions that can work with many different types. The initial function template that is used to generate other functions is called the **primary template**, and the functions generated from the primary template are called **instantiated functions**.

When we create a primary function template, we use **placeholder types** (technically called **type template parameters**, informally called **template types**) for any parameter types, return types, or types used in the function body that we want to be specified later, by the user of the template.

C++ supports 3 different kinds of template parameters:

- Type template parameters (where the template parameter represents a type).
- Non-type template parameters (where the template parameter represents a constexpr value).
- Template template parameters (where the template parameter represents a template).

Type template parameters are by far the most common, so we’ll focus on those first. We’ll also discuss non-type template parameters, which are seeing increased usage in modern C++.

```cpp
template <typename T> // this is the template parameter declaration defining T as a type template parameter
T max(T x, T y) // this is the function template definition for max<T>
{
    return (x < y) ? y : x;
}
```
Generally, prefer to define a non-template function if that option is available. inplace of template specialization.

The types to which the references refer are `int` for S1 and `const int` for S2. Those are indeed the same types except for top-level cv-qualifiers, and `const int` is indeed more cv-qualified than `int`, so S1 is a better conversion sequence and the primary function template is the best viable function.

A string literal is not a `std::string`, but a `const char[]` . If the compiler was to choose `f(const std::string&)`, it would have to go through a user defined conversion and create a temporary `std::string`. Instead, it prefers `f(const void*)`, which requires no user defined conversion.

In C++, the comma operator evaluates its left operand first, then its right operand, and the whole expression takes the value of the right operand. Don't confuse this with commas used to declare multiple variables, such as `int a = 10, b = 20;`.
```cpp
x = a, b; // a
x = (a, b); //b
```
The comma operator has the lowest precedence of all C++ operators (specifically lower than `=`).  
In this example it separates the two expressions `x = a` and `b`.

First `x = a` is evaluated, setting `x` to `10`.  
Then, `b` is evaluated, which does nothing.
```cpp
int j = 5;
int i = j, j;
```
This is also invalid: the second `j` redeclares a variable already declared in the same scope.

```cpp
int j = 1;

int main() {
  int& i = j, j;
  j = 2;
  std::cout << i << j;
}
```
The line `int& i = j, j;` declares two variables `i` and `j`. (If this is surprising, remember that you can declare multiple variables at once, e.g. `int i, j, k;`. `i` here just happens to have an initializer.)

So this program has two variables named `j`. The first is the global one, the second is local to `main`. Which one is used to initialize `i` in the expression `int& i = j`?

When initializing `i` with the expression `j`, we need to do name lookup for `j`. If the local `j` is in scope at that point in the program, it will be used, otherwise, the global `j` will be used.

_[§[basic.scope.pdecl]¶1](https://timsong-cpp.github.io/cppwp/std23/basic.scope.pdecl#1)_:

> The _locus_ of a declaration (_[§[basic.pre]](https://timsong-cpp.github.io/cppwp/std23/basic.pre)_) that is a declarator is immediately after the complete declarator (_[§[dcl.decl]](https://timsong-cpp.github.io/cppwp/std23/dcl.decl)_).

So the _locus_ of the local `j` is here:  

`int& i = j, j;             /\`

And _[§[basic.lookup.general]¶2](https://timsong-cpp.github.io/cppwp/std23/basic.lookup.general#2)_ says:

> A program point _P_ is said to follow any declaration in the same translation unit whose locus (_[§[basic.scope.pdecl]](https://timsong-cpp.github.io/cppwp/std23/basic.scope.pdecl)_) is before _P_.

The locus of the local `j` is _after_ the program point _P_ where the reference `i` is bound, so _P_ does not follow the declaration of the local `j`.

This means that name lookup for `j` in the expression `int& i = j` does not find the local `j`, but rather the global `j`, and `i` is bound to the global `j`.

However, the program point where we set `j = 2` does follow the locus of the local `j`, so we modify the _local_ `j`, and the `j` that `i` refers to is not affected.