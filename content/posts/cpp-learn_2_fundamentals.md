---
author: "Phong Nguyen"
title: "C++ - Chapter 2: Fundamentals"
date: "2026-05-24"
description: "C++ Notes"
tags: ["cpp"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---

C++ Basic Concepts
- `Data` is any information processed or stored by a computer.
- A `Value` is a specific piece of data (e.g. `42`, `'A'`, `3.14`).
- An `Object` is a region of storage (memory) with a type and a value.
- A `Variable` is a named object used to store data.
- `Initialization` is the process of giving an initial value to an object or variable.
- `Literals`: fixed values like 42, 3.14, 'A', "Hello", true, nullptr.
- `Operators`: symbols that act on values (+ - * / %, == != < >, && || !, = += -=, etc.).
- An `Expression` is a combination of operators, constants and variables that evaluate to a value.
- A `function` is a reusable block of code that performs a specific task.

---
## 1. Initialization
C++ provides several ways to initialize objects. Each form has different semantics and use cases.

### 1.1. Default Initialization
```cpp
int a;
T object;
new T;
```

Performed when an object is created **without any initializer**.
  - **Built-in types:** uninitialized
  - **Class types:** default constructor is called

<br>

### 1.2. Value Initialization
```cpp
int a{};

T();
new T();
T object{};
T{};
new T{};

/// @brief Also used in constructors
Class::Class() : member() {}
Class::Class() : member{} {}
```

Performed when an object is created with an **empty initializer**.
  - **Built-in types**: zero-initialized
  - **Class types**: default constructor is called

<br>

### 1.3 Direct Initialization
```cpp
T object(arg);
T object(arg1, arg2);

T object{arg};          // since C++11
new T(args...);
static_cast<T>(other);

/// @brief Used in constructors and lambdas:
Class::Class() : member(args...) {}
[arg]() {};
```

Initializes an object using **explicit constructor arguments**.

<br>

### 1.4 Copy Initialization
```cpp
T object = other;
f(other);
return other;
throw object;
catch (T object);

/// @brief Also used for arrays
T array[N] = { /* values */ };
```
Initializes an object **from another object or expression**.

### 1.5 List Initialization (since C++11)
```cpp
/// @brief Direct List Initialization
T object{arg1, arg2};
new T{arg1, arg2};

Class::Class() : member{arg1, arg2} {}

/// @brief Copy List Initialization
T object = {arg1, arg2};
return {arg1, arg2};
function({arg1, arg2});


/// @brief Designated Initializers (since C++20)
T object{ .des1 = arg1, .des2 = arg2 };
T object = { .des1 = arg1, .des2 = arg2 };
```

Initializes objects using **brace-enclosed initializer lists { }**.

<br>

### 1.6 Aggregate Initialization
```cpp
T object = {arg1, arg2};
T object{arg1, arg2};          // since C++11

/// @brief Designated initializers for aggregates (since C++20)
T object = { .des1 = arg1, .des2 = arg2 };
T object{ .des1 = arg1, .des2 = arg2 };
```
Initializes **aggregate types** (no user-defined constructors).

---
## 2. Functions and Files
### 2.1. Function
```cpp

returnType functionName(); ///< Forward Declaration

/// @brief Function Definition
returnType functionName(){
    

} 
```

- A `forward declaration` allows us to tell the compiler about the existence of an identifier before actually defining the identifier.

<br>

### 2.2. Namespace
```cpp
namespace nested_config{
    namespace config{
        int value = 1;
    }
}

using namespace cfg = nested_config::config;

void f(){
    std::cout << cfg::value;
}
```
- `namespace` is a way to group names (variables, functions, classes, ...) together and avoid name conflicts. It guarantees that all identifiers within the namespace are unique
- `using namespace`: this is a using-directive that allows us to access names in the std namespace with no namespace prefix
- The `scope resolution operator (::)` used to access namespace member
- Can create `namespace aliases`, which allow us to temporarily shorten a long sequence of namespaces into something shorter.
- namespaces can be nested inside other namespaces.

<br>

### 2.3. Preprocessor
The `preprocessor` is a process that runs on the code before it is compiled.
```cpp
#include <iostream>     // insert file contents
#define NAME "Alex"     // replace NAME -> "Alex"

#ifdef NAME_DEFINED     // only compile if defined
std::cout << NAME;
#endif
```

<br>

### 2.3. Header Files
```cpp
#include <iostream>     // search system only 
#include "my_header.h"  // search local first, then system

/// @brief Add include directories with -I
// g++ -o main -I./source/includes -I/home/abc/moreHeaders main.cpp
```

`Header files` are files designed to propagate declarations to code files. 
- `Header guards` prevent the contents of a header from being included more than once into a given code file. 
- For cross-platform library code, `#ifndef` is safest.
- For modern projects, `#pragma once` is simpler and safe.

---
## 3. Fundamental Data Types
Memory can only store bits. `Data type` help compiler and CPU take care of encoding the value into the sequence of bits.
    - Fundamental Data Types
    - Compound Data Types

### 3.1 Primitive Type
| Types                                                              | Category             | Meaning                                          | Example |
| ------------------------------------------------------------------ | -------------------- | ------------------------------------------------ | ------- |
| float, double, long double                                         | Floating Point       | a number with a fractional part                  | 3.14159 |
| bool                                                               | Integral (Boolean)   | true or false                                    | true    |
| char, wchar_t, char8_t (C++20), char16_t (C++11), char32_t (C++11) | Integral (Character) | a single character of text                       | 'c'     |
| short int, int, long int, long long int (C++11)                    | Integral (Integer)   | positive and negative whole numbers, including 0 | 64      |
| std::nullptr_t (C++11)                                             | Null Pointer         | a null pointer                                   | nullptr |
| void                                                               | Void                 | no type                                          | n/a     |

- `sizeof` used to get the size of a type in bytes. (pointer has 4 or 8 bytes based on the arch)
- `signed integers` can hold both positive and negative numbers (and 0). 
- `unsigned integers` can only hold non-negative whole numbers.
- `fixed-width integers` are the  set of integer types that are guaranteed to be the same size on any architecture **`#include <cstdint>`**
  - `fast integers`: guarantee at least # bits, but pick the type that the CPU can process fastest (even if it uses more memory).
  - `least integers`: guarantee at least # bits, but pick the type that uses the least memory (even if it’s slower).
- `size_t` vs `std::size_t` is an unsigned integral type that is used to represent the size or length of objects. (<stddef.h>,  <cstddef>)
- `scientific notation e/E` used to present the times 10 to the power of the equation. e.g.  **(e.g. 5.9722 x 10²⁴ -> 5.9722e24)**
- `Inf` represents infinity. Inf is signed, and can be positive (+Inf) or negative (-Inf). (5/0)
- `NaN` stands for “Not a Number”. (mathematically invalid)

<br>

### 3.2. Chars
- A `char` variable are interpreted as an `ASCII character`.
- `'t'`: Text between single quotes is treated as a char literal, which represents a single character.
- `"text"`: Text between double quotes (e.g. “Hello, world!”) is treated as a C-style string literal, which can contain multiple characters.

---
## 4. Constant
- A `constant` is a value that is not be changed during the program’s execution. There are two types of constants:
  - **Named constants**: are constant values that are associated with an identifier.
    - Constant variables
    - Macros with substitution text
    - Enumerated constant
  - **Literal** are constant values that are not associated with an identifier.Literals are values that are inserted directly into the code.

<br>

### 4.1. Named constants
```cpp
const double gravity = 9.8; ///< Const variable

#define MY_NAME "Phong"  ///< Object-like macros with substitution text

enum Color {
    RED,    // Assigned 0
    GREEN,  // Assigned 1
    BLUE = 5, // Manually assigned 5
    YELLOW  // Assigned 6 (5 + 1)
};///< Enumerated constant
```

### 4.2. Literals

```cpp
return 5;                       ///< 5 is an integer literal -> type: int
bool myNameIsAlex { true };     ///< true is a boolean literal -> type: bool
double d { 3.4 };               ///< 3.4 is a double literal -> type: double
cout << "Hello, world!";        ///< "Hello, world!" is a C-style string literal -> type: const char[14]
cout << 5.0 << '\n';            ///< 5.0 (no suffix) is type double (by default)
cout << 5.0f << '\n';           ///< 5.0f is type float
```

- `Type of a literal` is deduced from the literal's value.
- `Literal suffixes` used to explicitly declare the type for a literal.

<br>

### 4.3.  Numeral systems (decimal, binary, hexadecimal, and octal)
- Numeral system literals:
    - Decimal (no prefix, 42)
    - Binary (0b101010) `b`
    - Hexadecimal (0x2A) `x`
    - Octal (052) — all represent the same value.
- Can change the output format via use of the `std::dec, std::oct, and std::hex` I/O manipulators:
- Can define a `std::bitset` variable and tell `std::bitset` how many bits we want to store.
```cpp
    int bin{};          // assume 16-bit int
    bin = 0x0001;       // assign binary 0000 0000 0000 0001 to the variable
    bin = 0b1;          // assign binary 0000 0000 0000 0001 to the variable
    bin = 0b11;         // assign binary 0000 0000 0000 0011 to the variable

    cout << std::bitset<4>{ 0b1010 } << '\n'; // create a temporary std::bitset and print it

    // C++14: quotation mark (‘) as a digit separator
    int bin { 0b1011'0010 };        // assign binary 1011 0010 to the variable
    long value { 2'132'673'462 };   // much easier to read than 2132673462
```

<br>

### 4.4.  Constant Expression
- `Constant expressions` is an expressions whose values can be fully determined at **compile time**.
- `constexpr` is used to declare compile-time constants
- Benefits:
  - Safer code
  - Optimizations
  - Compile-time evaluation

- `constexpr` function is is a function that is allowed to be called in a constant expression, can be evaluated at compile time or runtime.
- `consteval` function is a function that must evaluate at compile-time.

```cpp
constexpr int square(int x) {
    return x * x;
}

consteval int cube(int x) {
    return x * x * x;
}

constexpr int a = square(5); // compile-time
int b = square(5);           // runtime allowed

constexpr int c = cube(5);   // OK
// int d = cube(5);          // Error if not compile-time
```

---
## 5. Operators & Bit Manipulation
### 5.1. Operators
- Refer : https://www.learncpp.com/cpp-tutorial/operator-precedence-and-associativity/
- `Increment/decrement`:
  -  `++x`: increment x, then return x
  -  `x++`: copy x, then increment x, return the copy
- `Comma`: 
  - `(x, y)`: Evaluate x then y, returns value of y
  - avoid use this

### 5.2. Bit manipulation.
- To define a set of bit flags, use `uint8/16/32`... or `std::bitset`
- Refers: https://www.learncpp.com/cpp-tutorial/bit-flags-and-bit-manipulation-via-stdbitset/

---
## 6. Control Flows
### 6.1. Constexpr if statement (C++17)
- Condition will be evaluated at runtime.
- e.g. 
```cpp
int main()
{
	constexpr double gravity{ 9.8 };

	if constexpr (gravity == 9.8) // now using constexpr if
		std::cout << "Gravity is normal.\n";
	else
		std::cout << "We are not on Earth.\n";

	return 0;
}
```

<br>

### 6.2. Switch fallthrough and scoping
- The `[[fallthrough]]` attribute modifies a null statement to indicate that fallthrough is intentional (and no warnings should be triggered).
- Initialization is not allowed before case labels because control flow in a switch may jump over the initializer, leaving the variable uninitialized.
- Declarations without an initializer are allowed before case labels because they only reserve space for the variable in the function’s stack frame (decided at compile time). No runtime initialization code is generated, so nothing can be “skipped” by jumping to a case.
- e.g.
```cpp
int main() {
    int x = 2;

    switch (x) {
        int a;       //  allowed (no initializer, just reserves space)
        // int b{5}; //  not allowed (initializer may be skipped)

    case 1:
        a = 10; // safe: 'a' exists, we assign here
        std::cout << "Case 1, a = " << a << '\n';
        [[fallthrough]]; //  intentional fallthrough to case 2

    case 2:
        a = 20; // reassign
        std::cout << "Case 2, a = " << a << '\n';
        break;

    default:
        std::cout << "Default case\n";
        break;
    }

    return 0;
}
```

<br>

### 6.3. Halts
- Halts allow us to terminate our program.Only use a halt if there is no safe or reasonable way to return normally from the main function. If you haven’t disabled exceptions, prefer using exceptions for handling errors safely.
- `std::exit` is called implicitly when main() returns, it does not clean up local variables in the current function or up the call stack.
- `std::abort()` function causes your program to terminate abnormally. Abnormal termination means the program had some kind of unusual runtime error and the program couldn’t continue to run. 
- `std::terminate()` function is typically used in conjunction with exceptions . By default, it calls `std::abort()`
- e.g.
```cpp
#include <iostream>
#include <cstdlib>    // for std::exit, std::abort
#include <exception>  // for std::terminate

void cleanup() {
    std::cout << "Cleaning up...\n";
}

void riskyFunction(bool fatalError) {
    if (fatalError) {
        std::cout << "Fatal error occurred!\n";

        // std::abort: abnormal termination, no cleanup
        std::abort();

        // Or: std::terminate(); // usually called when exception handling fails
    }
}

int main() {
    cleanup(); // local function call

    // Example 1: return normally from main
    std::cout << "Program running normally...\n";

    // Example 2: using std::exit (implicit when main returns)
    if (false) {
        std::cout << "Exiting via std::exit...\n";
        std::exit(0); // does not call destructors of locals in main()
    }

    // Example 3: risky code that might abort/terminate
    riskyFunction(true);

    // Example 4: normal end of main calls std::exit implicitly
    std::cout << "Main returns normally.\n";
    return 0; // std::exit(0) is called implicitly here
}
```