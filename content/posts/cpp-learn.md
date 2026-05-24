---
author: "Phong Nguyen"
title: "Cpp"
date: "2025-02-09"
description: "Cpp Notes"
tags: ["cpp"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
See plus plus :) .

<br>

[Refer](https://www.learncpp.com/)

1. [Introduce](cpp-learn_1_Introduction.md.md)
2. [Fundamentals](cpp-learn_2_fundamentals.md)
2. [String](cpp-learn_3_string.md)

## 11. Scope, duration, and linkage summary
> A variable’s duration determines when it is created and destroyed.
Variables with automatic duration: are created at the point of definition, and destroyed when the block they are part of is exited. This includes:
    - Local variables
    - Function parameters
Variables with static duration: are created when the program begins and destroyed when the program ends. This includes:
    - Global variables
    - Static local variables
Variables with dynamic duration:  are created and destroyed by programmer request. This includes:
    - Dynamically allocated variables

> extern	static (or thread_local) storage duration and external linkage	
static	static (or thread_local) storage duration and internal linkage	
thread_local	thread storage duration	
mutable	object allowed to be modified even if containing class is const	
auto	automatic storage duration	Deprecated in C++11
register	automatic storage duration and hint to the compiler to place in a register	Deprecated in C++17

- When you write an implementation file (.cpp, .cxx, etc) your compiler generates a translation unit. This is the source file from your implementation plus all the headers you #included in it.
Internal linkage refers to everything only in scope of a translation unit.
External linkage refers to things that exist beyond a particular translation unit. In other words, accessible through the whole program, which is the combination of all translation units (or object files).

![image](/images/learncpp_1.png)

<br>

### 11.1. Internal linkage
- An identifier’s linkage determines whether a declaration of that same identifier in a different scope refers to the same entity (object, function, reference, etc…) or not.

- An identifier with no linkage means another declaration of the same identifier refers to a unique entity. Entities whose identifiers have no linkage include:
Local variables
Program-defined type identifiers (such as enums and classes) declared inside a block

- An identifier with `internal linkage` means a declaration of the same identifier within the same translation unit refers to the same object or function. Entities whose identifiers have internal linkage include:
    - Static global variables (initialized or uninitialized)
    - Static functions
    - Const global variables
    - Unnamed namespaces and anything defined within them
- To make things to internal linkage, we can:
  - `static` global variables/functions
  - `const` and `constexpr` globals ((and thus don’t need the static keyword -- if it is used, it will be ignored)) ## C
  - unnamed namespace { ... } (modern C++)

- `static` <global_variable>: make the global variable  to internal linkage
- `static` <local_variable>: changes its duration from automatic duration to static duration. And its initializer is only executed once.
- `static` <const/constexpr local_varialbe>: used to avoid expensive local object initialization each time a function is called because it inits once time.

- Internal linkage for const global variables can change to external with the keyword `extern`. e.g. `extern const PI = 3.14;`
- e.g.
```cpp
// a.cpp ============================================================
#include <iostream>

static int g_internal { 42 };    // internal linkage (only in a.cpp)
static void helper() {           // internal function
    std::cout << "Helper in a.cpp\n";
}

int main() {
    std::cout << g_internal << '\n'; // OK
    helper();                         // OK
    return 0;
}

// main.cpp ============================================================
extern int g_internal; //  ERROR: g_internal not visible outside a.cpp
void helper();         //  ERROR: helper not visible outside a.cpp

int main() {
    // g_internal; // linker error if uncommented
    // helper();   // linker error if uncommented
    return 0;
}
```

### 11.2. External linkage
- An identifier with `external linkage` means a declaration of the same identifier within the entire program refers to the same object or function. Entities whose identifiers have external linkage include:
    - Non-static functions
    - Non-const global variables (initialized or uninitialized)
    - Extern const global variables
    - Inline const global variables
    - Namespaces
- `functions`: default external linkage
- `global variables`: 
  - `non-const globals`: external by default.
  - `const/constexpr globals`: internal by default
- To access an external global variable from another file, use `extern` without initializer (forward declaration).

- e.g.
```cpp
// a.cpp ============================================================
#include <iostream>

int g_external { 100 };            // external by default
extern const int g_limit { 200 };  // const made external

void sayHello() {                  // external by default
    std::cout << "Hello from a.cpp\n";
}

// main.cpp ============================================================
#include <iostream>

extern int g_external;       // forward declaration 
extern const int g_limit;    // forward declaration > < const int g_limit: definition
void sayHello();             // forward declaration

int main() {
    sayHello();
    std::cout << g_external << " / " << g_limit << '\n';
    return 0;
}
```

### 11.3. Inline functions and variables
> When a call to min() is encountered, the CPU must store the address of the current instruction it is executing (so it knows where to return to later) along with the values of various CPU registers (so they can be restored upon returning). Then parameters x and y must be instantiated and then initialized. Then the execution path has to jump to the code in the min() function. When the function ends, the program has to jump back to the location of the function call, and the return value has to be copied so it can be output. This has to be done for each function call.
All of the extra work that must happen to setup, facilitate, and/or cleanup after some task (in this case, making a function call) is called overhead.
#include <iostream>
int min(int x, int y)
{
    return (x < y) ? x : y;
}
int main()
{
    std::cout << min(5, 6) << '\n';
    std::cout << min(3, 2) << '\n';
    return 0;
}
For functions that are large and/or perform complex tasks, the overhead of the function call is typically insignificant compared to the amount of time the function takes to run. However, for small functions (such as min() above), the overhead costs can be larger than the time needed to actually execute the function’s code! In cases where a small function is called often, using a function can result in a significant performance penalty over writing the same code in-place.
However, inline expansion has its own potential cost: if the body of the function being expanded takes more instructions than the function call being replaced, then each inline expansion will cause the executable to grow larger.

- `inline-expansion`:
  - is a process where a function call is replaced by the code from the called function’s definition.
  - use this to avoid such overhead cost.
  - do not use the `inline` keyword to request inline expansion for your functions, because optimizing compilers.
- `modernly-inline`: 
  - should not implement functions (with external linkage) in header files because it lead to `multiple definitions` error.
  - so we can use `inline-function`, it's useful for header-only libraries

### 11.4. Sharing global constants
- 1. `global constants as internal variables`: 
  - Advantages:
    - Works prior to C++16.
    - Can be used in constant expressions in any translation unit that includes them.
  - Downsides:
    - Changing anything in the header file requires recompiling files including the header.
    - Each translation unit including the header gets its own copy of the variable.
- e.g.
```cpp
// constants.h:============================================================
#ifndef CONSTANTS_H
#define CONSTANTS_H

// Define your own namespace to hold constants
namespace constants
{
    // Global constants have internal linkage by default
    constexpr double pi { 3.14159 };
    constexpr double avogadro { 6.0221413e23 };
    constexpr double myGravity { 9.2 }; // m/s^2 -- gravity is light on this planet
    // ... other related constants
}
#endif

// main.cpp::============================================================
#include "constants.h" // include a copy of each constant in this file
#include <iostream>

int main()
{
    std::cout << "Enter a radius: ";
    double radius{};
    std::cin >> radius;

    std::cout << "The circumference is: " << 2 * radius * constants::pi << '\n';

    return 0;
}
```

<br>

- 2. `global constants as external variables`: 
  - Advantages:
    - Works prior to C++16.
    - Only one copy of each variable is required.
    - Only requires recompilation of one file if the value of a constant changes.
  - Downsides:
    - Forward declarations and variable definitions are in separate files, and must be kept in sync.
    - Variables not usable in constant expressions outside of the file in which they are defined.
- e.g.
```cpp
// constants.h:============================================================
#ifndef CONSTANTS_H
#define CONSTANTS_H

namespace constants
{
    // Since the actual variables are inside a namespace, the forward declarations need to be inside a namespace as well
    // We can't forward declare variables as constexpr, but we can forward declare them as (runtime) const
    extern const double pi;
    extern const double avogadro;
    extern const double myGravity;
}

#endif

// constants.cpp:============================================================
#include "constants.h"

namespace constants
{
    // We use extern to ensure these have external linkage
    extern constexpr double pi { 3.14159 };
    extern constexpr double avogadro { 6.0221413e23 };
    extern constexpr double myGravity { 9.2 }; // m/s^2 -- gravity is light on this planet
}

// main.cpp::============================================================
#include "constants.h" // include all the forward declarations
#include <iostream>

int main()
{
    std::cout << "Enter a radius: ";
    double radius{};
    std::cin >> radius;

    std::cout << "The circumference is: " << 2 * radius * constants::pi << '\n';

    return 0;
}
```
- 3. `global constants as inline variables`: 
  - If you need global constants and your compiler is C++17 capable, prefer defining inline constexpr global variables in a header file.
  - Advantages:
    - Can be used in constant expressions in any translation unit that includes them.
    - Only one copy of each variable is required.
  - Downsides:
    - Only works in C++17 onward.
    - Changing anything in the header file requires recompiling files including the header.
- e.g.
```cpp
// constants.h:============================================================
#ifndef CONSTANTS_H
#define CONSTANTS_H

// define your own namespace to hold constants
namespace constants
{
    inline constexpr double pi { 3.14159 }; // note: now inline constexpr
    inline constexpr double avogadro { 6.0221413e23 };
    inline constexpr double myGravity { 9.2 }; // m/s^2 -- gravity is light on this planet
    // ... other related constants
}
#endif

// main.cpp::============================================================
#include "constants.h"
#include <iostream>

int main()
{
    std::cout << "Enter a radius: ";
    double radius{};
    std::cin >> radius;

    std::cout << "The circumference is: " << 2 * radius * constants::pi << '\n';

    return 0;
}
```



## 13. Error Detection and Handling

## 14. Type Conversion, Type Aliases, and Type Deduction
```xml
Type conversions
│
├── Implicit conversions (compiler tự làm)
│   ├── Numeric promotions             ← an toàn, không mất dữ liệu
│   │    ├── char → int
│   │    └── float → double
│   │
│   └── Numeric conversions            ← có thể mất dữ liệu
│        ├── Widening conversions      ← mở rộng, thường an toàn (int → double)
│        └── Narrowing conversions     ← thu hẹp, có thể mất dữ liệu (double → int, int → char)
│
└── Explicit conversions (do lập trình viên ép kiểu)
     ├── static_cast<int>(3.14)
     ├── reinterpret_cast
     ├── const_cast
     └── (int)3.14   // C-style cast
```
### 14.1. Implicit type conversion
- **Implicit type conversion** is performed automatically by the compiler when an expression of some type is supplied in a context where some other type is expected.
-  **numeric promotion**  is the conversion of certain smaller numeric types to certain larger numeric types (typically int or double).no data loss.
   -  `floating point promotion`:  a value of type float can be converted to a value of type double.
   -  `integral promotions`:
      -  signed char or signed short can be converted to int.
      -  unsigned char, char8_t, and unsigned short can be converted to int if int can hold the entire range of the type, or unsigned int otherwise.
      -  If char is signed by default, it follows the signed char conversion rules above. If it is unsigned by default, it follows the unsigned char conversion rules above.
      -  bool can be converted to int, with false becoming 0 and true becoming 1.
- **numeric conversion** is a type conversion between fundamental types that isn’t a numeric promotion. A narrowing conversion is a numeric conversion that may result in the loss of value or precision.
  - :( https://www.learncpp.com/cpp-tutorial/narrowing-conversions-list-initialization-and-constexpr-initializers/)

### 14.2. Explicit type conversion
- C++ supports 5 different types of casts: `static_cast`, `dynamic_cast`, `const_cast`, `reinterpret_cast`, and `C-style` casts
- 1. `C-style` casts
  - Using operator`(<type>)value`. and the value to convert to placed immediately to the right of the closing parenthesis `)`. e.g. `(int)7/3`
- 2. `static_cast` casts
  - Using `static_cast<type>(value)
  - provides compile-time type checking
  - less powerful than a C-style cast
  - direct-initialized vs list-initialization

|Purpose|Example|Explaination|
|---|---|---|
|Cast object → object	|static_cast<Base>(derived)	|Creates a new Base object copied from Derived (object slicing).|
|Cast object → reference	|static_cast<Base&>(derived)	|No copy — just changes the view of the same object to treat it as a Base.|
|Cast pointer → pointer	|static_cast<Base*>(&derived)	|Upcasts a Derived* pointer to a Base* pointer.|
### 14.3. Typedefs and type aliases
- `type aliases`: use the `using` keyworld
  - `using <NewtypeAlias> = <type>`
- `typedefs`: is an old way of creating an alias for a type. Using `typedef` keyworld.
  - `typedef <type> <NewtypeAlias> ` 
- Prefer `type alias` over `typedef`
```cpp
using MyDouble = double;
typedef double MyDouble;

typedef int (*FcnType)(double, char); // FcnType hard to find
using FcnType = int(*)(double, char); // FcnType easier to find
```
### 14.5. Type deduction using auto keyworld
- Type deduction allows the compiler to deduce the type of an object from the object’s initializer. 
- using `auto` keyworld.
- Type deduction must have something to deduce from
- Type deduction drops const from the deduced type
- ... (more)
- Use type deduction for your variables when the type of the object doesn’t matter.
- Auto can also be used as a function return type to have the compiler infer the function’s return type from the function’s return statements, though this should be avoided for normal functions.
- The auto keyword can also be used to declare functions using a trailing return syntax, where the return type is specified after the rest of the function prototype.
- e.g.
```cpp
int add(int x, int y)
{
  return (x + y);
}

// Using the trailing return syntax, this could be equivalently written as:
auto add(int x, int y) -> int
{
  return (x + y);
}

#include <type_traits> // for std::common_type

std::common_type_t<int, double> compare(int, double);         // harder to read (where is the name of the function in this mess?)
auto compare(int, double) -> std::common_type_t<int, double>; // easier to read (we don't have to read the return type unless we care)
```

<br>


## 15. Function Overloading and Function Templates
### 15.1. Function overloading
- `function overloading` allows us to create multiple functions with the same name, so long as each identically named function has different parameter types/numbers.  Return types are not considered for 
- `function delete` using `delete` keywork. delete means “I forbid this”, not “this doesn’t exist”. 
- e.g.
```cpp
#include <iostream>

// This function will take precedence for arguments of type int
void printInt(int x)
{
    std::cout << x << '\n';
}

// This function template will take precedence for arguments of other types
// Since this function template is deleted, calls to it will halt compilation
template <typename T>
void printInt(T x) = delete;

int main()
{
    printInt(97);   // okay
    printInt('a');  // compile error
    printInt(true); // compile error

    return 0;
}
```
- `default-arguments`: is a default value provided for a function parameter. Parameters with default arguments must always be the rightmost parameters, and they are not used to differentiate functions when resolving overloaded functions.

### 15.2. Function Templates
- The template system was designed to simplify the process of creating functions (or classes) that are able to work with different data types (that are compiled and executed).
- `template types` are sometimes called generic types, and programming using templates is sometimes called generic programming.
- `placeholder types` use for any parameter types, return types, or types used in the function body that we want to be specified later, by the user of the template.
- `template parameter declaration` defines any template parameters that will be subsequently used.
- `function templates` allow us to create a function-like definition that serves as a pattern for creating related functions. In a function template, we use type template parameters as placeholders for any types we want to be specified later. The syntax that tells the compiler we’re defining a template and declares the template types is called a template parameter declaration.
- Using function templates in multiple files. should be defined in a header file, and then #included wherever needed. 
- `template argument deduction` to have the compiler deduce the actual type that should be used from the argument types in the function call.
- e.g. 
```cpp
template <typename T> // this is the template parameter declaration defining T as a type template parameter , `typename` or `class` can be used
T max(T x, T y) // this is the function template definition for max<T>
{
    return (x < y) ? y : x;
}

template<>
int max<int>(int x, int y) // the generated function max<int>(int, int)
{
    return (x < y) ? y : x;
}

template<>
double max<double>(double x, double y) // the generated function max<double>(double, double)
{
    return (x < y) ? y : x;
}

int main()
{
    std::cout << max<int>(1, 2) << '\n'; // calls max<int>(int, int)
    std::cout << max<>(1, 2) << '\n';    // deduces max<int>(int, int) (non-template functions not considered)
    std::cout << max(1, 2) << '\n';      // calls max(int, int)

    return 0;
}
```

- `function templates with multiple template` types example:
```cpp
#include <iostream>

template <typename T, typename U>
auto max(T x, U y) // ask compiler can figure out what the relevant return type is
{
    return (x < y) ? y : x;
}

int main()
{
    std::cout << max(2, 3.5) << '\n';

    return 0;
}
```

- `non-type template parameter` is a template parameter with a fixed type that serves as a placeholder for a constexpr value passed in as a template argument.
```cpp
#include <iostream>

template <int N> // int non-type template parameter
void print()
{
    std::cout << N << '\n';
}

int main()
{
    print<5>();   // no conversion necessary
    print<'c'>(); // 'c' converted to type int, prints 99

    return 0;
}
```

<br>

## 16. Compound Types: References and Pointers

| Type            | Meaning                                                                                                           | Examples                              |
|-----------------|-------------------------------------------------------------------------------------------------------------------|---------------------------------------|
| **Fundamental** | A basic type built into the core C++ language                                                                    | `int`, `std::nullptr_t`               |
| **Compound**    | A type defined in terms of other types                                                                            | `int&`, `double*`, `std::string`, `Fraction` |
| **User-defined** | A class type or enumerated type <br> (Includes those defined in the standard library or implementation) <br> (In casual use, typically used to mean program-defined types) | `std::string`, `Fraction`             |
| **Program-defined** | A class type or enumerated type <br> (Excludes those defined in standard library or implementation)            | —                                     |

- `Compound data types` (also called composite data type) are data types that can be constructed from fundamental data types (or other compound data types).
### 16.1. lvalues and rvalues
- `lvalues` is an expression that evaluates to an identifiable object or function (or bit-field). Can be accessed via an identifier, reference, or pointer, and typically have a lifetime longer than a single `expression` or `statement`.
- `rvalues` is an expression that evaluate to a value. Only exist within the scope of the expression in which they are used.
- `lvalues` can be used anywhere an `rvalue` is expected.
- An **assignment operation** requires its `left` operand to be a modifiable `lvalue` expression. And its `right` operand to be a `rvalue` expression.

### 16.2. References
- **references** is an alias for an existing object/function. `reference` itself is like a `const pointer`
  - Declared as `<type>& reference_name`
  - Any operation on the reference is applied to the object being referenced.
  - All references must be initialized.
  - Cannot be reseated.
  - They aren't objects
  - Can only accept modifiable lvalue arguments (const or non-const)

- **pass-by-reference** allows us:
  - to pass arguments to a function without making copies of those arguments each time the function is called. (class types)
  - to change the value of an argument

- **pass-by-const-reference** guaranteeing that the function can not change the value being referenced.
- **lvalue-reference** just a reference for an existing lvalue.
- **lvalue-reference-types** determines what type of object it can reference by using a single ampersand  `<type>&` .
- **lvalue-reference-variable** is a variable that acts as a reference to an lvalue.
- **lvalue-reference-to-const** can bind with const or non-const objects. `const <type>& name`

>The const applies to what is immediately to its left, unless there’s nothing to its left, in which case it applies to what’s on the right.

- E.g.
```cpp
#include <iostream>
#include <string>

// ---------------------------
// Example of references
// ---------------------------
void increment(int& x) {   // pass-by-reference (modifiable lvalue reference)
    x += 1;                // changes original argument
}

void printConstRef(const std::string& s) { // pass-by-const-reference
    // s cannot be modified here
    std::cout << "Const-ref: " << s << "\n";
}

int main() {
    int a = 10;
    
    // ---- Reference basics ----
    int& ref = a;       // reference must be initialized
    ref = 20;           // modifies 'a', since ref is just an alias
    std::cout << "a = " << a << "\n";  // prints 20
    
    // Cannot reseat: once 'ref' is bound to 'a', it cannot be bound to another variable
    int b = 30;
    // ref = &b;   invalid, would assign value instead of rebinding
    
    // ---- Pass by reference ----
    increment(a);  // modifies original 'a'
    std::cout << "a after increment = " << a << "\n"; // prints 21

    // ---- Pass by const reference ----
    std::string text = "Hello";
    printConstRef(text);  // avoids making a copy

    // ---- Lvalue reference variable ----
    int& lref = a;  // lref is an lvalue-reference-variable to 'a'
    lref += 5;      // modifies 'a'
    std::cout << "a after lref += 5: " << a << "\n";

    // ---- Lvalue reference to const ----
    const int x = 100;
    const int& cref1 = x; // bind to const object
    const int& cref2 = a; // also works with non-const object
    std::cout << "cref1 = " << cref1 << ", cref2 = " << cref2 << "\n";
    
    // ---- They aren't objects ----
    // sizeof(ref) == sizeof(a), because ref is just an alias
    std::cout << "sizeof(a) == sizeof(ref): "
              << (sizeof(a) == sizeof(ref)) << "\n";

    return 0;
}
```

### 16.3. Pointer
- **address-of-operator (&<variable>)** returns the memory address of its operand, **but not as an address literal. Instead, it returns a pointer to the operand.** This pointer holds the address value, and when passed to cout, the stream simply prints that value.
- **dereference-operator  (*<address>)** returns the value at a given memory address as an lvalue, used to access the object being pointed at.
- **pointer** is an object that holds a memory address as its value:
  - declared as `<type>* ptr_name`
  -  This allows us to store the address of some other object to use later.
  -  we should init the pointers.
  -  the size of pointer is allways the same (32 or 64-bit architecture)
  -  can assign an invalid pointer a new value, such as `nullptr`
  -  `wild pointer`: pointer that has not been initialized is sometimes called a .
  -   `dangling pointer`: pointer that is holding the address of an object that is no longer valid
- **pointer-type** is a type that specifies a pointer (like reference-type) by using an asterisk `(<type>*)`.The type of the pointer has to match the type of the object being pointed at.
- **null-pointer** (its type is `std::nullptr_t`) means something has no value. It often associated with memory address 0. 

- **pointer-to-const**: that points to a value that cannot be modified through the pointer, but the pointer itself is not const.
  - declared as `const <type> ptr_name*`.
  - cannot change the value being pointed to, but can make the pointer point to a different address.
  - may also point to non-const variables.
- **const-pointer**: whose stored address cannot be changed after initialization, but the value at that address can be modified.
  - declared as `<type>* const ptr_name`.
  - fixed to one address, but we can change the value at that address.
- **const-pointer-to-const**: cannot be reseated (address fixed) and cannot modify the value it points to.
  - declared as `const <type>* const ptr_name`.
  - can only be dereferenced to read the value.
>pointer-to-pointer
void allocateArray(int** ptr, int size) {
    *ptr = new int[size]; // allocate memory and update the original pointer
}
int* myArray = nullptr;
allocateArray(&myArray, 5);
myArray[0] = 10; // works
delete[] myArray;
- e.g.
```cpp
#include <iostream>
#include <cstdint>  // for uintptr_t

int main() {
    int x = 42;

    // address-of operator (&) returns a pointer to x (not an address literal)
    int* ptr = &x;  
    
    // Printing the pointer: cout prints the stored address value
    std::cout << "Address of x (&x): " << &x << "\n";
    std::cout << "Value stored in pointer (ptr): " << ptr << "\n";

    // dereference operator (*) gives access to the value at the stored address
    std::cout << "Value of x via *ptr: " << *ptr << "\n";

    // a pointer is an object that holds a memory address
    // we can change its value
    ptr = nullptr;  
    std::cout << "Pointer reset to nullptr: " << ptr << "\n";

    // pointer type is declared with '*'
    double d = 3.14;
    double* pd = &d;    // type matches: double* for double
    // uintptr_t lets us see the raw numeric value of the address
    std::cout << "Numeric address of d: " << (uintptr_t)pd << "\n";
    std::cout << "Value of d via *pd: " << *pd << "\n";
    
    // NullPTR =======
    int* myNullPtr {};        // value-initialized to nullptr
    int* myNullPtr2 {nullptr}; // explicitly initialized to nullptr

    // Old C-style (still works, but less safe in C++):
    // int* myNullPtr3 {NULL};   // requires <cstddef>
    
    // Const ptr =======
    int a = 10;
    int b = 20;
    // 1. pointer-to-const (const int*)
    const int* p1 = &a;      // can point to non-const variable
    // *p1 = 15;             //  error: cannot modify value through p1
    p1 = &b;                 //  can point to another address
    std::cout << "p1 points to: " << *p1 << '\n';
    // 2. const-pointer (int* const)
    int* const p2 = &a;      // must be initialized, fixed address
    *p2 = 30;                //  can modify the value at that address
    // p2 = &b;              //  error: cannot change stored address
    std::cout << "p2 points to: " << *p2 << '\n';
    // 3. const-pointer-to-const (const int* const)
    const int* const p3 = &b; // fixed address + read-only value
    // *p3 = 40;             //  error: cannot modify value
    // p3 = &a;              //  error: cannot reseat pointer
    std::cout << "p3 points to: " << *p3 << '\n';
       

    return 0;
}
```

- **pointer-to-pointers**:
	- a pointer that holds the address of another pointer.
	- Using two asterisks to declare a pointer to pointer.
	- e.g. `int** ptrptr;`
	- Usages:
		- Dynamically allocate an array of pointers
		- e.g. `int** array { new int*[10] }; // allocate an array of 10 int pointers`
		- Two-dimensional dynamically allocated arrays
		- e.g. `int x { 7 }; // non-constant
int (*array)[5] { new int[x][5] }; // rightmost dimension must be constexpr`

<br>

- **void-pointers**: Also known as the generic pointer, is a special type of pointer that can be pointed at objects of any data type
	- Dereferencing a void pointer is illegal. Instead, the void pointer must first be cast to another pointer type before the dereference can be performed.
	- We do not know what type of object it is pointing to, deleting a void pointer will result in undefined behavior.

<br>

- **function-pointers**:
  - e.g.
    ```cpp
        // fcnPtr is a pointer to a function that takes no arguments and returns an integer
        int (*fcnPtr)();
    ```

  - Assigning a function to a function pointer: just like a normal pointer, and the type (parameters and return type) of the function pointer must match the type of the function. 
  - e.g.
    ```cpp
    // function prototypes
    int foo();
    double goo();
    int hoo(int x);
    
    // function pointer initializers
    int (*fcnPtr1)(){ &foo };    // okay
    int (*fcnPtr2)(){ &goo };    // wrong -- return types don't match!
    double (*fcnPtr4)(){ &goo }; // okay
    fcnPtr1 = &hoo;              // wrong -- fcnPtr1 has no parameters, but hoo() does
    int (*fcnPtr3)(int){ &hoo }; // okay
    ```

  - Calling a function using function pointer: There are two way to do this
    - Explicitly derefence
    - Implicitly derefence
  - e.g.
    ```cpp
    int foo(int x)
    {
        return x;
    }
    
    int main()
    {
        int (*fcnPtr)(int){ &foo }; // Initialize fcnPtr with function foo
        (*fcnPtr)(5); // call function foo(5) through fcnPtr.
    
    	int (*fcnPtr2)(int){ &foo }; // Initialize fcnPtr with function foo
        fcnPtr2(5); // call function foo(5) through fcnPtr2.
    	
        return 0;
    }
    ```
  - Passing functions as arguments to other functions: One of the most useful things to do with function pointers is pass a function as an argument to another function. Functions used as arguments to another function are sometimes called **callback functions**.

<br>
	
### 16.4. Pass by value/reference/address

```cpp
#include <iostream>
#include <string>

void printByValue(std::string val) // The function parameter is a copy of str
{
    std::cout << val << '\n'; // print the value via the copy
}

void printByReference(const std::string& ref) // The function parameter is a reference that binds to str
{
    std::cout << ref << '\n'; // print the value via the reference
}

void printByAddress(const std::string* ptr) // The function parameter is a pointer that holds the address of str
{
    std::cout << *ptr << '\n'; // print the value via the dereferenced pointer
}

int main()
{
    std::string str{ "Hello, world!" };

    printByValue(str); // pass str by value, makes a copy of str
    printByReference(str); // pass str by reference, does not make a copy of str
    printByAddress(&str); // pass str by address, does not make a copy of str

    return 0;
}
```
- **pass-by-address** allows us: ~ **pass-by-references**
  - to pass arguments to a function without making copies of those arguments each time the function is called. (class types)
  - to change the value of an argument
  - #null checking
> Pass by reference when you can, pass by address when you must

- **!! C++ really passes everything by value**

### 16.5. Return by value/reference/address
- **T returnValue(...)**: returns a copy (or move) of the object. The caller gets its own value.
- **T& returnReference(...)** returns a reference to an existing object. The caller does not own it, so the object must outlive the reference.
- **T * returnAddress(...)** returns a pointer (an address) to an object. The caller must handle the pointer carefully (ensure it’s valid and points to a live object).

- `return-by-reference`:
  - avoids making a copy of the object.
  - the referenced object must live beyond the scope of the function, otherwise the reference will dangle.
  - never return a non-static local variable or temporary by reference.
- `return-by-address` works almost identically to return-by-reference.
- `return-by-value` just make a copy
> Prefer return by reference over return by address unless the ability to return “no object” (using nullptr) is important.

- e.g.
```cpp
#include <iostream>

int global = 42;

// Return by value: makes a copy
int returnValue() {
    int x = 10;
    return x; // copy returned
}

// Return by reference: must refer to existing object
int& returnReference() {
    return global; // safe: global outlives the function
}

// Return by address: returns a pointer
int* returnAddress(bool valid) {
    if (valid)
        return &global; // valid pointer
    else
        return nullptr; // no object
}

int main() {
    int a = returnValue();
    std::cout << "By value: " << a << '\n';

    int& b = returnReference();
    std::cout << "By reference: " << b << '\n';
    b = 100; // modifies global
    std::cout << "Global after modification: " << global << '\n';

    int* c = returnAddress(true);
    if (c) std::cout << "By address: " << *c << '\n';

    int* d = returnAddress(false);
    if (!d) std::cout << "By address: got nullptr\n";
}
```

### 16.6. In/Out Params
- `in-parameters`:are typically passed `by value` or `by const reference`
- `out-parameters`:a function parameter that is used only for the purpose of returning information back to the caller.
  - Avoid out-parameters (except in the rare case where no better options exist).
  - Prefer pass by reference for non-optional out-parameters.

- e.g.
```cpp
#include <iostream>
#include <string>

// In-parameter by value (cheap to copy)
void greet(std::string name) {
    std::cout << "Hello, " << name << "!\n";
}

// In-parameter by const reference (avoid copy for large objects)
int length(const std::string& text) {
    return text.size();
}

// Out-parameter by reference (rare case)
void square(int input, int& output) {
    output = input * input;
}

int main() {
    // in-parameter by value
    greet("Alice");

    // in-parameter by const reference
    std::string msg = "Hello World";
    std::cout << "Length = " << length(msg) << "\n";

    // out-parameter by reference (not preferred, but possible)
    int result;
    square(5, result);
    std::cout << "Square = " << result << "\n";
}
```

### 16.7. Type deduction (auto) with pointers, references, and const
https://www.learncpp.com/cpp-tutorial/type-deduction-with-pointers-references-and-const/

### 16.8. std::optional
https://www.learncpp.com/cpp-tutorial/stdoptional/

## 18. Compound Types: Enums and Structs
- `program-defined-types` are types that programmers create themself.
> In C++, struct, class, and union automatically create a new type name, so you don’t need to prefix variables with the keywords struct or union as in C.
### 18.1. Enumerations
- `enum` is a compound types where every possible value is defined as a symbolic constant.
- Named starting with a capital letter. Named enumerators starting with a lower case letter.
- `unscoped-enum`: put their enumerator names into the same scope as the enumeration definition itself 
- `scoped-enum`: keep their enumerators inside the enum’s own scope.Using `enum class` keyworld.
- `using enum <EnumName>` statement imports all the emnumerators from an enum into the current scope. 
- putting your enumerations inside a named scope region (such as a namespace or class) so the enumerators don’t pollute the global namespace.
- Specify the base type of an enumeration only when necessary.
- e.g.
```cpp
#include <iostream>
#include <cstdint>   // for uint8_t

//  Good naming style:
// Enum name: Capitalized
// Enumerator names: lowercase

enum Color {
    red,
    green,
    blue
};

//  Scoped enum — enumerators are inside the enum’s scope
enum class Shape {
    circle,
    square,
    triangle
};

//  Scoped enum inside a namespace — prevents name pollution
namespace Game {
    enum class Direction {
        up,
        down,
        left,
        right
    };
}

//  Scoped enum with explicit base type
enum class Status : uint8_t {
    ok = 0,
    error = 1,
    unknown = 2
};

int main() {
    // Using unscoped enum
    Color c = red;               //  direct access (same scope)
    std::cout << "Color value: " << c << "\n";

    // Using scoped enum
    Shape s = Shape::circle;     //  must use scope name
    if (s == Shape::circle)
        std::cout << "Shape is circle\n";

    // Scoped enum inside namespace
    Game::Direction dir = Game::Direction::up;
    if (dir == Game::Direction::up)
        std::cout << "Direction is up\n";

    // Scoped enum with base type
    Status st = Status::ok;
    if (st == Status::ok)
        std::cout << "Status OK (base type uint8_t)\n";

    return 0;
}
```

- **Union**: https://www.geeksforgeeks.org/cpp/cpp-unions/

### 18.2. Struct
- A **struct** is a *class type* (just like `classes` or `union`),  allows us to bundle multiple variables together into a single type. As such, anything that applies to class types applies to structs.
- **Defining structs** using `struct` keywords.
- **Access struct members**:
  - Use member selection operator(dot operator) `.` for reference/object.
  - Use arrow operator `->` for pointers. `ptr->id = (*ptr).id`
- **Initialization**: 
  - using brace-initialization `{}` or by defining default member values.
  - should provide a default value for all members
  - Struct aggregate initializations:
```cpp
    Employee frank = { 1, 32, 60000.0 }; // copy-list initialization using braced list
    Employee joe { 2, 28, 45000.0 };     // list initialization using braced list (preferred)
``` 
- **Passing and returning structs**:
  - Passingy reference (efficient and avoids copying)
  - Passing temporary 
  - Create a struct variable and return
  - Returning a temporary (unnamed/anonymous) object 
- **Struct size and data structure alignment**:the size of a struct will be at least as large as the size of all the variables it contains. But it could be larger! For performance reasons, the compiler will sometimes add gaps into structures this is called **padding**. We can minimize padding by defining your members in **decreasing order of size**.(e.g., double → int → char).

- e.g.
```cpp
#include <iostream>
#include <string>
using namespace std;

// Define a struct
struct SensorData {
    double voltage{0.0};   // Default initialization
    int id{0};
    char status{'N'};      // 'N' = normal, 'E' = error
    string label{"Unknown"};

    // Member function
    void print() const { // Const class objects and const member functions
        cout << "Sensor " << id 
             << " [" << label << "] "
             << "Voltage: " << voltage 
             << " Status: " << status << endl;
    }
};

// Function that accepts struct by reference
void updateVoltage(SensorData &data, double newV) {
    data.voltage = newV;
}

// Function returning a temporary struct
SensorData makeSensor(int id, double v, const string &label) {
    return {v, id, 'N', label};
}

int main() {
    // Initialization using braces
    SensorData s1{3.3, 1, 'N', "Temperature"};
    s1.print();

    // Pointer access
    SensorData *ptr = &s1;
    ptr->status = 'E';
    ptr->print();

    // Passing by reference
    updateVoltage(s1, 4.8);
    s1.print();

    // Returning temporary struct
    SensorData s2 = makeSensor(2, 5.0, "Pressure");
    s2.print();

    cout << "Size of struct = " << sizeof(SensorData) << " bytes" << endl;
    return 0;
}
```

### 18.3. Class template
- a **class template** is a template definition for instantiating class types (structs, classes, or unions). Class template argument deduction (CTAD) is a C++17 feature that allows the compiler to deduce the template type arguments from an initializer.
- Using class template in a function:
- e.g.
```cpp
#include <iostream>

template <typename T>
struct Pair
{
    T first{};
    T second{};
};

template <typename T>
constexpr T max(Pair<T> p)
{
    return (p.first < p.second ? p.second : p.first);
}

int main()
{
    Pair<int> p1{ 5, 6 };        // instantiates Pair<int> and creates object p1
    std::cout << p1.first << ' ' << p1.second << '\n';

    Pair<double> p2{ 1.2, 3.4 }; // instantiates Pair<double> and creates object p2
    std::cout << p2.first << ' ' << p2.second << '\n';

    Pair<double> p3{ 7.8, 9.0 }; // creates object p3 using prior definition for Pair<double>
    std::cout << p3.first << ' ' << p3.second << '\n';

    std::cout << max<int>(p1) << " is larger\n"; // explicit call to max<int>

    return 0;
}

// Compiler ===================================================================================
#include <iostream>

// A declaration for our Pair class template
// (we don't need the definition any more since it's not used)
template <typename T>
struct Pair;

// Explicitly define what Pair<int> looks like
template <> // tells the compiler this is a template type with no template parameters
struct Pair<int>
{
    int first{};
    int second{};
};

// Explicitly define what Pair<double> looks like
template <> // tells the compiler this is a template type with no template parameters
struct Pair<double>
{
    double first{};
    double second{};
};

int main()
{
    Pair<int> p1{ 5, 6 };        // instantiates Pair<int> and creates object p1
    std::cout << p1.first << ' ' << p1.second << '\n';

    Pair<double> p2{ 1.2, 3.4 }; // instantiates Pair<double> and creates object p2
    std::cout << p2.first << ' ' << p2.second << '\n';

    Pair<double> p3{ 7.8, 9.0 }; // creates object p3 using prior definition for Pair<double>
    std::cout << p3.first << ' ' << p3.second << '\n';

    return 0;
}
```

- **Using class templates in multiple files**:Just like function templates, class templates are typically defined in header files

- **Alias templates**: is a template that can be used to instantiate type aliases.
- e.g.
```cpp
// Alias templates must be defined in global scope
template <typename T>
using Coord = Pair<T>; // Coord is an alias for Pair<T>
```
## 19. Classes
- A **class** is a `user-defined blueprint` used to create objects. It defines the `properties and behaviors` that `all objects of that type` share.
- An **object** is an `instance of a class`. It represents `a real entity` and contains `actual values` for the class’s attributes.
- An **instance** is a `specific object` created from a class. (In practice, “object” and “instance” are often used interchangeably.)

- **Four Pillars of OOP in C++**:
  - **Abstraction** is the `process of hiding the implementation details` and only `showing the essential details or features` to the user. It `allows to focus on` what an object `does` rather than `how it does` it.
    It is achieved using `abstract classes` (classes that have at least one pure virtual function).

  - **Encapsulation** is the `process of bundling data and methods into a single unit` (class) and `restricting direct access` to some components.
    Data is hidden and accessed through public methods.
    It is achieved using access specifiers like `private, protected, and public`.

  - **Inheritance**  is `a mechanism` where a derived class acquires the properties and behaviors of a base class, forming an `is-a` relationship.
    It improves code reuse and extensibility.
    It is implemented using `:` followed by an access specifier `public, private, protected`.

  - **Polymorphism** means `many forms`. It allows the `same interface (function or method)` to `behave differently` depending on the context.
    It is achieved through:
        - `Compile-time polymorphism`: `function overloading, operator overloading`
        - `Runtime polymorphism`: `virtual functions`

- **Overview:**
```cpp
// ===== File: main.cpp =====
#include <iostream>
#include <string>
#include <utility>
#include <vector>
using namespace std;

// ===== File: Person.h =====
class Person {
private:                       // (14.5) Data hiding and encapsulation
    string name;
    int age;

public:
    // (14.9) Constructor
    Person(string n, int a) : name{std::move(n)}, age{a} {
        cout << "Constructor: " << name << endl;
    }

    // (14.12) Delegating constructor
    Person() : Person("Unknown", 0) {}

    // (14.14) Copy constructor
    Person(const Person& other) : name{other.name}, age{other.age} {
        cout << "Copy constructor called for " << name << endl;
    }

    // (15.4) Destructor
    ~Person() {
        cout << "Destructor: " << name << endl;
    }

    // (14.3) Member function
    void print() const {  // (14.4) const member function
        cout << name << " (" << age << ")\n";
    }

    // (14.6) Access function
    string getName() const { return name; }
    int getAge() const { return age; }

    // (14.7) Return reference to data member
    int& getAgeRef() { return age; }

    // (14.16) Converting constructor (implicit)
    Person(int a) : name{"Anon"}, age{a} {}

    // (14.17) constexpr constructor (simple example)
    constexpr Person(const char* n, int a, bool) : name{n}, age{a} {}

    // (15.1) Function chaining using hidden this pointer
    Person& setName(const string& n) { name = n; return *this; }
    Person& setAge(int a) { age = a; return *this; }

    // (15.7) Static member function
    static void hello() { cout << "Hello from Person class!\n"; }

    // (15.8) Friend non-member function
    friend void showSecret(const Person& p);
};

// ===== File: Person.cpp =====
void showSecret(const Person& p) {   // (15.8) Friend function
    cout << "Friend: " << p.name << " is " << p.age << " years old.\n";
}

// ===== File: Employee.h =====
class Employee : public Person {    // Inheritance for demonstration
private:
    double salary;

public:
    Employee(string n, int a, double s)
        : Person(std::move(n), a), salary{s} {}

    void print() const {            // Override
        cout << "[Employee] ";
        Person::print();
        cout << "Salary: $" << salary << "\n";
    }

    // (15.9) Friend class
    friend class HR;
};

// ===== File: HR.h =====
class HR {
public:
    void adjustSalary(Employee& e, double newSalary) {
        e.salary = newSalary;
        cout << "HR adjusted salary!\n";
    }
};

// ===== File: Counter.h =====
class Counter {
private:
    static int count;   // (15.6) Static member variable

public:
    Counter() { ++count; }
    ~Counter() { --count; }
    static int getCount() { return count; } // (15.7)
};
int Counter::count = 0;

// ===== File: Box.h =====
// (15.5) Class template with member functions
template <typename T>
class Box {
private:
    T value;
public:
    explicit Box(T v) : value{v} {}
    T get() const { return value; }
    void set(T v) { value = v; }
};

// ===== File: main.cpp (continued) =====
int main() {
    cout << "=== OOP Example ===\n";

    // Constructors
    Person p1("Alice", 25);
    Person p2 = p1;              // Copy constructor
    Person p3;                   // Delegating constructor

    p1.print();

    // Function chaining
    p1.setName("Bob").setAge(30).print();

    // Static function
    Person::hello();

    // Friend function
    showSecret(p1);

    // Employee & HR
    Employee e1("Charlie", 35, 50000);
    HR hr;
    e1.print();
    hr.adjustSalary(e1, 60000);
    e1.print();

    // Static member variable usage
    Counter c1, c2;
    cout << "Counter count: " << Counter::getCount() << endl;

    // Template class
    Box<int> intBox(123);
    cout << "Box value: " << intBox.get() << endl;

    // (14.16) Implicit converting constructor
    Person p4 = 42;  // age=42, name="Anon"
    p4.print();

    // (14.17) constexpr example
    constexpr Person p5("ConstExpr", 10, true);
    p5.print();

    cout << "=== End ===\n";
    return 0;
}
```

### 19.0. Shallow copying & Deep copying
- **Shallow copying:**
  - C++ does not know much about our class, so the `default copy constructor` and `default assignment operators` use memberwise copy and then `copy each member of the class individually`.
  - Simple classes => work well.
  - Classes handling dynamically allocated memory => just copy the address of the pointer => does not allocate any memory => causes problems.

- **Deep copying:**
  - Allocates memory for the copy and then copies the actual values, so that the copy lives in memory distinct from the source.
  - The original and the copy will not affect each other in any way.
  - This requires write our own `default copy constructor` and `default assigment operators`

- **Role of three:**
>If a class requires a user-defined destructor, a user-defined copy constructor, or a user-defined copy assignment operator, it almost certainly requires all three.This ensures proper resource management and avoids shallow copy problems.
- **Role of five:**
>Extends the Rule of Three in C++11 and later. In addition to the destructor, copy constructor, and copy assignment operator, it includes the move constructor and move assignment operator. This allows efficient transfer of resources instead of copying.
- **Role of zero:**
>The best practice is to write classes that do not manage resources directly, letting the compiler generate all special member functions automatically. This avoids the need to define destructors or copy/move operations manually.

### 19.1. Introduce
- A class is a program-defined compound type that can have many member variables with **different types** (this point is different from structure).
- A class is a program-defined compound type that bundles both data and functions that work on that data.

- Examples:
```cpp
#include <iostream>
class MyClass{
private:
     int variable1;
     int variable2;
public:
    MyClass(int v1, int v2){
        variable1 = v1;
        variable2 = v2;
    }

    void printVariables(){
        std::cout << variable1 << "-" <<variable2;
    }
};

int main()
{
    std::cout<<"Hello Classes";
    MyClass myObject = MyClass(3,4);
    myObject.printVariables();
    return 0;
}
```

### 19.2. Member Variables/ Functions:
- The variable/functions that belong to a class type are called member variables/ functions.
- In C, structs only have data members, not member functions.

### 19.3. Const class objects and const member functions
- Const class objects: just like with normal variables, we can make our class type objects const or constexpr.
- Const member function: is a member function that guarentees it will not modify the object or call any non-constant member functions.
	- Syntax `<returnType> <nameFunction> const(<params>){}`

### 19.4. Temporary class objects
-  A temporary object (sometimes called an `anonymous object` or an `unnamed object`) is an object that has no name and exists only for the duration of a single expression.
- e.g.
```cpp
#include <iostream>

class IntPair
{
private:
    int m_x{};
    int m_y{};

public:
    IntPair(int x, int y)
        : m_x { x }, m_y { y }
    {}

    int x() const { return m_x; }
    int y() const{ return m_y; }
};

void print(IntPair p)
{
    std::cout << "(" << p.x() << ", " << p.y() << ")\n";
}

int main()
{
    // Case 1: Pass variable
    IntPair p { 3, 4 };
    print(p);

    // Case 2: Construct temporary IntPair and pass to function
	// When the function call returns, the temporary object is destroyed.
    print(IntPair { 5, 6 } );

    // Case 3: Implicitly convert { 7, 8 } to a temporary Intpair and pass to function
    print( { 7, 8 } );

    return 0;
}
```

### 19.5. Public and private members and access specifiers
- By default, all members of a `struct` are public members.
- By default, the members of a class are `private`.

### 19.6. Access functions
TODO:
### 19.7.  Member functions returning references to data members
TODO:
### 19.8. Encapsulation - The benefits of data hiding
TODO:
### 19.9. Constructors
- We are normally using the aggregate to initialize the class type(Aggregate initialization means the use of brace-enclosed initializer lists to initialize all members of an aggregate (ie an array or struct)), but it does not work as soon as we make member variables private.
- A constructor is a special member function that is used to initialize class type objects. That is automatically called after a non-aggregate class type object is created.
	- perform initialization of any member variables (via a member initialization list)
    - perform other setup functions (via statements in the body of the constructor). This might include things such as error checking the initialization values, opening a file or database, etc…
- **Syntax**:
	- Constructors must have the same name as the class (with the same capitalization).
	- For template classes, this name excludes the template parameters.
    - Constructors have no return type (not even void).

### 19.10. Constructor - member initializer lists
- The **member initializer list** is defined after the constructor parameters. It begins with a colon (`:`), and then lists each member to initialize along with the initialization value for that variable, separated by a comma (`,`). We must use a direct form of initialization here (preferably using braces(`{}`), but parentheses(`()`) works as well) .Using copy initialization (with an equals(`=`)) does not work here.

- e.g.:
```cpp
Foo(int x, int y) : m_x { x }, m_y { y }
{
	// m_x  = x; this is an assignment
}
```
- Members in the member initializer list should be listed in the order in which they are defined in the class
- Prefer using the member initializer list to initialize your members over assigning values because in case where members are required to be initialized (such as for data members that are const or references) assignment will not work.

### 19.11. Constructor - Default and default arguments
- A **default constructor** is a constructor that accepts no arguments.
- Because constructors are functions, we can:
	- Constructors with default arguments
	- Overloaded constructors
- **An implicit default constructor** is generated by the compiler when the class has *no user-declared constructors*. This constructor has nothing.
- **An explicitly default constructor** is used in case we already create the constructor ourselves, but also want the compiler to generate the default constructor. Using keywork `default`
- e.g.
```cpp
class Foo
{
private:
    int m_x {};
    int m_y {};

public:
    Foo() = default; // generates an explicitly defaulted default constructor

    Foo(int x, int y)
        : m_x { x }, m_y { y }
    {
        std::cout << "Foo(" << m_x << ", " << m_y << ") constructed\n";
    }
};

int main()
{
    Foo foo{}; // calls Foo() default constructor
		
    return 0;
}
```

### 19.12. Constructor - Delegating
- **Delegating constructors** allow to delegate (transfer responsibility for) initialization to another constructor from the same class type.
- Simply *call the constructor in the member initializer list*
- Use of the `static` keyword for the const variables member allows us to have a single  member that is shared by all class objects.
- e.g.
```cpp
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
private:
    std::string m_name { "???" };
    int m_id { 0 };

public:
    Employee(std::string_view name)
        : Employee{ name, 0 } // delegate initialization to Employee(std::string_view, int) constructor
    {
    }

    Employee(std::string_view name, int id)
        : m_name{ name }, m_id { id } // actually initializes the members
    {
        std::cout << "Employee " << m_name << " created\n";
    }

};

int main()
{
    Employee e1{ "James" };
    Employee e2{ "Dave", 42 };
}
```

### 19.13. Constructor - Copy
- **A copy constructor** is a constructor that is used to initialize an object with an existing object of the same type. After the copy constructor executes, the newly created object should be a copy of the object passed in as the initializer.
- **An implicit copy constructor**: C++ will create a public implicit copy constructor for us if we do not provide a one.
- **An explicitly copy constructor**: by explicitly define our own copy constructor
- e.g.
```cpp
    // Copy constructor
    Fraction(const Fraction& fraction)
        // Initialize our members using the corresponding member of the parameter
        : m_numerator{ fraction.m_numerator }
        , m_denominator{ fraction.m_denominator }
    {
        // do other things
    }

```
- Using `= default` to generate a default copy constructor.
- Using `= delete` to prevent copies.
```cpp
    // Explicitly request default copy constructor
    Fraction(const Fraction& fraction) = default;
    Fraction fCopy { f };
	
    // Delete the copy constructor so no copies can be made
    Fraction(const Fraction& fraction) = delete;
	Fraction f { 5, 3 };
	Fraction fCopy { f }; // compile error: copy constructor has been deleted

```
>The rule of three is a well known C++ principle that states that if a class requires a user-defined copy constructor, destructor, or copy assignment operator, then it probably requires all three. In C++11, this was expanded to the rule of five, which adds the move constructor and move assignment operator to the list.
Not following the rule of three/rule of five is likely to lead to malfunctioning code. We’ll revisit the rule of three and rule of five when we cover dynamic memory allocation


### 19.14. Class initialization and copy elision
- **For variables**:
```cpp
int a;         // no initializer (default initialization)
int b = 5;     // initializer after equals sign (copy initialization)
int c( 6 );    // initializer in parentheses (direct initialization)

// List initialization methods (C++11)
int d { 7 };   // initializer in braces (direct list initialization)
int e = { 8 }; // initializer in braces after equals sign (copy list initialization)
int f {};      // initializer is empty braces (value initialization)
```

- **For object with class types**:
```cpp
#include <iostream>

class Foo
{
public:

    // Default constructor
    Foo()
    {
        std::cout << "Foo()\n";
    }

    // Normal constructor
    Foo(int x)
    {
        std::cout << "Foo(int) " << x << '\n';
    }

    // Copy constructor
    Foo(const Foo&)
    {
        std::cout << "Foo(const Foo&)\n";
    }
};

int main()
{
    // Calls Foo() default constructor
    Foo f1;           // default initialization
    Foo f2{};         // value initialization (preferred)

    // Calls foo(int) normal constructor
    Foo f3 = 3;       // copy initialization (non-explicit constructors only)
    Foo f4(4);        // direct initialization
    Foo f5{ 5 };      // direct list initialization (preferred)
    Foo f6 = { 6 };   // copy list initialization (non-explicit constructors only)

    // Calls foo(const Foo&) copy constructor
    Foo f7 = f3;      // copy initialization
    Foo f8(f3);       // direct initialization
    Foo f9{ f3 };     // direct list initialization (preferred)
    Foo f10 = { f3 }; // copy list initialization

    return 0;
}
```
- For all types of initialization:
  - When initializing a class type, the set of constructors for that class are examined, and overload resolution is used to determine the best matching constructor. This may involve implicit conversion of arguments.
  - When initializing a non-class type, the implicit conversion rules are used to determine whether an implicit conversion exists.
  - List initialization disallows narrowing conversions.
  - Copy initialization only considers non-explicit constructors/conversion functions.
  - List initialization prioritizes matching list constructors over other matching constructors. 
	
### 19.15. Converting constructors and the explicit keyword
- For example:
```cpp
#include <iostream>

class Foo
{
private:
    int m_x{};
public:
    Foo(int x)
        : m_x{ x }
    {
    }

    int getX() const { return m_x; }
};

void printFoo(Foo f) // has a Foo parameter
{
    std::cout << f.getX();
}

int main()
{
    printFoo(5); // we're supplying an int argument

    return 0;
}
```
- The compiler will look to see if there is any constructor that it can be use to perform (5) -> Foo.
- A converting constructor is a constructor that can be used to perform an implicit conversion is called a converting constructor. By default, all constructors are converting constructors.
- Only one user-defined conversion may be applied.
  
  <br>

- **The `explicit` keyword** is used to to tell the compiler that a constructor should not be used as a converting constructor.
	- For constructors with a separate declaration (inside the class) and definition (outside the class), the explicit keyword is used only on the declaration.
	- Explicit constructors can be used for direct and direct list initialization
	- Prefer use this key work for constructors that take a single argument. 
- e.g.
```cpp
#include <iostream>

class Dollars
{
private:
    int m_dollars{};

public:
    explicit Dollars(int d) // now explicit
        : m_dollars{ d }
    {
    }

    int getDollars() const { return m_dollars; }
};

void print(Dollars d)
{
    std::cout << "$" << d.getDollars();
}

int main()
{
    print(5); // compilation error because Dollars(int) is explicit
	Dollars d1(5); // ok
    Dollars d2{5}; // ok
    return 0;
}
```

- **Return by value and explicit constructors**: when we return a value from a function, if that value does not match the return type of the function, an implicit conversion will occur. Just like with pass by value, such conversions cannot use explicit constructors.
- e.g.
```cpp
#include <iostream>

class Foo
{
public:
    explicit Foo() // note: explicit (just for sake of example)
    {
    }

    explicit Foo(int x) // note: explicit
    {
    }
};

Foo getFoo()
{
    // explicit Foo() cases
    return Foo{ };   // ok
    return { };      // error: can't implicitly convert initializer list to Foo

    // explicit Foo(int) cases
    return 5;        // error: can't implicitly convert int to Foo
    return Foo{ 5 }; // ok
    return { 5 };    // error: can't implicitly convert initializer list to Foo
}

int main()
{
    return 0;
}
```

### 19.16. Constexpr aggregates and classes:
@TODO:

### 19.17. The hidden “this” pointer and member function chaining
- Cpp utilizes a hidden pointer named `this`. This is a *const pointer* that holds the address of the current implicit object.
- e.g.
```cpp
simple.setID(2); 
Simple::setID(&simple, 2); // note that simple has been changed from an object prefix to a function argument!

// implementation
void setID(int id) { m_id = id; }
static void setID(Simple* const this, int id) { this->m_id = id; }
```
- **Explainations:**
  - When we call simple.setID(2), the compiler actually calls Simple::setID(&simple, 2), and simple is passed by address to the function.
The function has a hidden parameter named this which receives the address of simple.
Member variables inside setID() are prefixed with this->, which points to simple. So when the compiler evaluates this->m_id, it’s actually resolving to simple.m_id.
  - All non-static member functions have a this const pointer that holds the address of the implicit object. `this` always points to the object being operated on
  - Using this to create a reset function for a class back to default state:create a new object (using the default constructor), and then assign that new object to the current implicit object
```cpp
void reset()
{
    *this = {}; // value initialize a new object and overwrite the implicit object
}
```

- `this` and `const objects`:
```cpp
#include <iostream>
using namespace std;

class Something {
private:
    int value;

public:
    Something(int v) : value(v) {}

    // Non-const member function
    void setValue(int v) {
        cout << "Non-const setValue() called" << endl;
        value = v;  // allowed: can modify object
    }

    // Const member function
    int getValue() const {
        cout << "Const getValue() called" << endl;
        // value = 10; //  would be an error: cannot modify inside const function
        return value;
    }

    // Non-const getValue (overload)
    int getValue() {
        cout << "Non-const getValue() called" << endl;
        return value;
    }
};

int main() {
    Something a(5);
    const Something b(10);

    // Non-const object: can call both const and non-const functions
    cout << a.getValue() << endl;  // calls non-const version
    a.setValue(7);                 // OK
    cout << a.getValue() << endl;

    // Const object: can only call const functions
    cout << b.getValue() << endl;  // calls const version
    // b.setValue(20);             // error: cannot call non-const on const object

    return 0;
}

```

> Why this a pointer and not a reference:
Since the this pointer always points to the implicit object (and can never be a null pointer unless we’ve done something to cause undefined behavior), you may be wondering why this is a pointer instead of a reference. The answer is simple: when this was added to C++, references didn’t exist yet.

### 19.18. Classes and header files
- **Member functions** can be defined outside the class definition just like non-member functions. The only difference is that we must prefix the member function names with the name of the class type so the compiler knows we’re defining a member of that class type rather than a non-member. e.g. `void MyClass::myFnc(){//}`.
  - Putting class definitions in a header file.
  - Prefer to put your class definitions in a header file with the same name as the class. 
  - Trivial member functions (such as access functions, constructors with empty bodies,Default arguments for member functions, etc…) can be defined inside the class definition.
  - Prefer to define non-trivial member functions in a source file with the same name as the class.
- **Inline member functions**:
  - Member functions defined inside the class definition are implicitly inline. 
  - Member functions defined outside the class definition are not implicitly inline (and thus are subject to the one definition per program part of the one-definition rule). This is why such functions are usually defined in a code file (where they will only have one definition across the whole program).
  - Alternatively, member functions defined outside the class definition can be left in the header file if they are made inline.
- e.g.
```cpp
// Date.h:
#ifndef DATE_H
#define DATE_H

class Date
{
private:
    int m_year{};
    int m_month{};
    int m_day{};

public:
    Date(int year, int month, int day);

    void print() const;

    int getYear() const { return m_year; }
    int getMonth() const { return m_month; }
    int getDay() const { return m_day; }
};

#endif

// Date.cpp
#include "Date.h"

Date::Date(int year, int month, int day) // constructor definition
    : m_year{ year }
    , m_month{ month }
    , m_day{ day }
{
}

void Date::print() const // print function definition
{
    std::cout << "Date(" << m_year << ", " << m_month << ", " << m_day << ")\n";
};
```

### 19.19. Nested types (member types)
- Class types support another kind of member: `nested types` (also called member types). To create a nested type, you simply define the type inside the class, under the appropriate access specifier.
- Define any nested types at the top of your class type. 
- Outside the class, we must use the fully qualified name: `OuterClass::InerClass p{};`
- `nested classes` are members of the outer class, they can access any private members of the outer class that are in scope.
- A nested type cannot be forward declared prior to the definition of the enclosing class.\ư

### 19.20. Destructors
- For example, the classes that use a resource (most often memory, but sometimes files, databases, network connections, etc…) often need to be explicitly sent or closed before the class object using them is destroyed. In other cases, we may want to do some record-keeping prior to the destruction of the object, such as writing information to a log file, or sending a piece of telemetry to a server. The term “clean up” is often used to refer to any set of tasks that a class must perform before an object of the class is destroyed in order to behave as expected. If we have to rely on the user of such a class to ensure that the function that performs clean up is called prior to the object being destroyed, we are likely to run into errors somewhere.

<br>

- **destructor** is a special member function that is called automatically when an object of a non-aggregate class type is destroyed.
- **Syntax**:
  - The destructor must have the same name as the class, preceded by a tilde (`~`).
  - The destructor can not take arguments.
  - The destructor has no return type.
- A class can only have a single destructor.
- **An implicit destructor**: If a non-aggregate class type object has no user-declared destructor, the compiler will generate a destructor with an empty body. It is effectively just a placeholder.

- e.g.
```cpp
class NetworkData
{
private:
    std::string m_serverName{};
    DataStore m_dataQueue{};

public:
	NetworkData(std::string_view serverName)
		: m_serverName { serverName }
	{
	}

	~NetworkData()
	{
		sendData(); // make sure all data is sent before object is destroyed
	}

	void addData(std::string_view data)
	{
		m_dataQueue.add(data);
	}

	void sendData()
	{
		// connect to server
		// send all data
		// clear data
	}
};

int main()
{
    NetworkData n("someipAddress");

    n.addData("somedata1");
    n.addData("somedata2");

    return 0;
}
```

### 19.21. Class templates with member functions
-e.g.
```cpp
#include <ios>       // for std::boolalpha
#include <iostream>

template <typename T>
class Pair
{
private:
    T m_first{};
    T m_second{};

public:
    // When we define a member function inside the class definition,
    // the template parameter declaration belonging to the class applies
    Pair(const T& first, const T& second)
        : m_first{ first }
        , m_second{ second }
    {
    }

    bool isEqual(const Pair<T>& pair);
};

// When we define a member function outside the class definition,
// we need to resupply a template parameter declaration
template <typename T>
bool Pair<T>::isEqual(const Pair<T>& pair)
{
    return m_first == pair.m_first && m_second == pair.m_second;
}

int main()
{
    Pair p1{ 5, 6 }; // uses CTAD to infer type Pair<int>
    std::cout << std::boolalpha << "isEqual(5, 6): " << p1.isEqual( Pair{5, 6} ) << '\n';
    std::cout << std::boolalpha << "isEqual(5, 7): " << p1.isEqual( Pair{5, 7} ) << '\n';

    return 0;
}
```

### 19.22. Static member
- **Static member variables:** are static duration members that are shared by all objects of the class. Static members exist even if no objects of the class have been instantiated. Prefer to access them using the class name, the scope resolution operator, and the members name.
- Syntax: `static <type> <name>{<value>}`
- Static member variables are shared by all objects of the class
- Static members are not associated with class objects
- Static members are global variables that live inside the scope region of the class. 
- Access static members using the class name and the scope resolution operator (`::`).
- Defining and initializing static member variables:
	- Initialization of static member variables inside the class definition
	- Make your static members `inline` or `constexpr` so they can be initialized inside the class definition (.h).  
	
- **Static member functions:** are member functions that can be called with no object. They do not have a *this pointer, and cannot access non-static data members.
  - Can access to the satatic members via non-static function but requires us to instantiate an object to call
  - Because static member functions are not attached to an object, they have no this pointer
  - Can directly access other static members (variables or functions), but not non-static members.

- e.g.
```cpp
#include <iostream>
#include <string>

class Counter {
public:
    // Static member variable (shared by all instances)
    static inline int count {0};              // C++17+ inline initialization
    static constexpr const char* name {"App"}; // compile-time constant

    Counter() {
        ++count; // Increment static variable each time an object is created
    }

    ~Counter() {
        --count; // Decrement on destruction
    }

    // Static member function
    static void showCount() {
        std::cout << "Count: " << count << '\n';
        // std::cout << id;  not allowed — no access to non-static members
    }

    // Non-static function accessing static member
    void showInfo() const {
        std::cout << "Instance -> " << name << " | Current count: " << count << '\n';
    }

private:
    int id {0}; // non-static member variable
};

// (Alternative old-style definition — no longer needed with inline)
// int Counter::count = 0;

int main() {
    Counter c1, c2;
    c1.showInfo();     // Access static via non-static method
    Counter::showCount(); // Access static function via class name

    Counter c3;
    Counter::showCount();

    return 0;
}

```

### 19.23. Friend non-member functions
- **A friend declaration** (using the `friend` keyword) can be used to tell the compiler that some other class or function is now a friend. In C++, a friend is a class or function (member or non-member) that has been granted full access to the private and protected members of another class. In this way, a class can selectively give other classes or functions full access to their members without impacting anything else.

- **A friend function** is a function (member or non-member) that can access the private and protected members of a class as though it were a member of that class. In all other regards, the friend function is a normal function. 
```cpp
#include <iostream>

class Accumulator
{
private:
    int m_value { 0 };

public:
    void add(int value) { m_value += value; }

    // Here is the friend declaration that makes non-member function void print(const Accumulator& accumulator) a friend of Accumulator
	// member function but it have friend keyword, it is instead treated as a non-member function
    friend void print(const Accumulator& accumulator);
};

// non-member function
void print(const Accumulator& accumulator)
{
    // Because print() is a friend of Accumulator
    // it can access the private members of Accumulator
    std::cout << accumulator.m_value;
}

int main()
{
    Accumulator acc{};
    acc.add(5); // add 5 to the accumulator

    print(acc); // call the print() non-member function

    return 0;
}
```

- We can also define a friend non-member inside a class.
- Prefer non-friend functions to friend functions
- **Multiple friends:** A function can be a friend of more than one class at the same time. ( Class forward declarations serve the same role as function forward declarations)

### 19.24. Friend classes and friend member functions
- **A friend class** is a class that can access the private and protected members of another class.
- e.g. 
```cpp
#include <iostream>

class Storage
{
private:
    int m_nValue {};
    double m_dValue {};
public:
    Storage(int nValue, double dValue)
       : m_nValue { nValue }, m_dValue { dValue }
    { }

    // Make the Display class a friend of Storage
    friend class Display;
};

class Display
{
private:
    bool m_displayIntFirst {};

public:
    Display(bool displayIntFirst)
         : m_displayIntFirst { displayIntFirst }
    {
    }

    // Because Display is a friend of Storage, Display members can access the private members of Storage
    void displayStorage(const Storage& storage)
    {
        if (m_displayIntFirst)
            std::cout << storage.m_nValue << ' ' << storage.m_dValue << '\n';
        else // display double first
            std::cout << storage.m_dValue << ' ' << storage.m_nValue << '\n';
    }

    void setDisplayIntFirst(bool b)
    {
         m_displayIntFirst = b;
    }
};

int main()
{
    Storage storage { 5, 6.7 };
    Display display { false };

    display.displayStorage(storage);

    display.setDisplayIntFirst(true);
    display.displayStorage(storage);

    return 0;
}
    
```

- **A friend member function** is a specific member function of one class that is granted access to the private and protected members of another class.
- e.g. 
```cpp
#include <iostream>
class Storage; // forward declaration for class Storage
class Display
{
private:
	bool m_displayIntFirst {};

public:
	Display(bool displayIntFirst)
		: m_displayIntFirst { displayIntFirst }
	{
	}

	void displayStorage(const Storage& storage); // forward declaration for Storage needed for reference here
};

class Storage // full definition of Storage class
{
private:
	int m_nValue {};
	double m_dValue {};
public:
	Storage(int nValue, double dValue)
		: m_nValue { nValue }, m_dValue { dValue }
	{
	}

	// Make the Display::displayStorage member function a friend of the Storage class
	// Requires seeing the full definition of class Display (as displayStorage is a member)
	friend void Display::displayStorage(const Storage& storage);
};

// Now we can define Display::displayStorage
// Requires seeing the full definition of class Storage (as we access Storage members)
void Display::displayStorage(const Storage& storage)
{
	if (m_displayIntFirst)
		std::cout << storage.m_nValue << ' ' << storage.m_dValue << '\n';
	else // display double first
		std::cout << storage.m_dValue << ' ' << storage.m_nValue << '\n';
}

int main()
{
    Storage storage { 5, 6.7 };
    Display display { false };
    display.displayStorage(storage);

    return 0;
}
```

### 19.25. Ref qualifiers
- C++11 introduced a little known feature called a **ref-qualifier** that allows us to overload a member function based on whether it is being called on an lvalue or an rvalue implicit object. Using this feature, we can create two versions of getName() -- one for the case where our implicit object is an lvalue, and one for the case where our implicit object is an rvalue.
- e.g.
```cpp
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
private:
	std::string m_name{};

public:
	Employee(std::string_view name): m_name { name } {}

	const std::string& getName() const &  { return m_name; } //  & qualifier overloads function to match only lvalue implicit objects
	std::string        getName() const && { return m_name; } // && qualifier overloads function to match only rvalue implicit objects
};

// createEmployee() returns an Employee by value (which means the returned value is an rvalue)
Employee createEmployee(std::string_view name)
{
	Employee e { name };
	return e;
}

int main()
{
	Employee joe { "Joe" };
	std::cout << joe.getName() << '\n'; // Joe is an lvalue, so this calls std::string& getName() & (returns a reference)

	std::cout << createEmployee("Frank").getName() << '\n'; // Frank is an rvalue, so this calls std::string getName() && (makes a copy)

	return 0;
}
```

## 20. Inheritance 
### 20.1. Object Relationships
- Learned about some different kinds of relationships between two objects.

- The process of building complex objects from simpler ones is called **object composition**.
- There are two types of object composition: `composition`, and `aggregation`.
- **Composition** exists when a member of a class has a part-of relationship with the class. In a composition relationship, the class manages the existence of the members. 
  - To qualify as a composition, an object and a part must have the following relationship:
    - The part (member) is part of the object (class)
    - The part (member) can only belong to one object (class) at a time
    - The part (member) has its existence managed by the object (class)
    - The part (member) does not know about the existence of the object (class)
  - Typically implemented via normal member variables, or by pointers where the class manages all the memory allocation and deallocation. If you can implement a class as a composition, you should implement a class as a composition.

- **Aggregations** exists when a class has a has-a relationship with the member. In an aggregation relationship, the class does not manage the existence of the members. 
  - To qualify as an aggregation, an object and its parts must have the following relationship:
    - The part (member) is part of the object (class)
    - The part (member) can belong to more than one object (class) at a time
    - The part (member) does not have its existence managed by the object (class)
    - The part (member) does not know about the existence of the object (class)
  - Typically implemented via pointer or reference.

- **Associations** are a looser type of relationship, where the class uses-an otherwise unrelated object. 
  - To qualify as an association, an object and an associated object must have the following relationship:
    - The associated object (member) is otherwise unrelated to the object (class)
    -  associated object (member) can belong to more than one object (class) at a time
    - The associated object (member) does not have its existence managed by the object (class)
    - The associated object (member) may or may not know about the existence of the object (class)
  - May be implemented via pointer or reference, or by a more indirect means (such as holding the index or key of the associated object).

- In a **dependency**, one class uses another class to perform a task. The dependent class typically is not a member of the class using it, but rather is temporarily created, used, and then destroyed, or passed into a member function from an external source.

- In a **container** class one class provides a container to hold multiple objects of another type. A value container is a composition that stores copies of the objects it is holding. A reference container is an aggregation that stores pointers or references to objects that live outside the container.

- **std::initializer_lis**t can be used to implement constructors, assignment operators, and other functions that accept a list initialization parameter. 
- std::initializer_list lives in the <initializer_list> header.

|Property\Type 	|Composition 	|Aggregation 	|Association 	|Dependency|
|---|---|---|---|---|
|Relationship type 	|Whole/part 	|Whole/part 	|Otherwise unrelated 	|Otherwise unrelated|
|Members can belong to multiple classes 	|No 	|Yes 	|Yes 	|Yes|
|Members existence managed by class 	|Yes 	|No 	|No 	|No|
|Directionality 	|Unidirectional 	|Unidirectional 	|Unidirectional or bidirectional 	|Unidirectional|
|Relationship verb 	|Part-of 	|Has-a 	|Uses-a 	|Depends-on|

### 20.2. Introduce Inheritance 
- `Inheritance` allows us to reuse classes by having other classes inherit their members.
- Syntax: after the class declaration, we use a colon `:`, the word `public` and the name of the class we wish to inherit.
- e.g.

```Cpp
class Base
{
public:
    int m_id {};

    Base(int id=0)
        : m_id { id }
    {
    }

    int getId() const { return m_id; }
};

class Derived: public Base
{
public:
    double m_cost {};

    Derived(double cost=0.0)
        : m_cost { cost }
    {
    }

    double getCost() const { return m_cost; }
};
```

- `C++ constructs` derived classes in phases, starting with the most-base class (at the top of the inheritance tree) and finishing with the most-child class (at the bottom of the inheritance tree).

### 20.3. Constructors and initialization of derived classes
  - `Constructors`: the derived class constructor is responsible for determining which base class constructor is called. If no base class constructor is specified, the default base class constructor will be used. In that case, if no default base class constructor can be found (or created by default), the compiler will display an error.
  - `Destructors:` When a derived class is destroyed, each destructor is called in the reverse order of construction.
  - In more detail:
    - Memory for the derived class is set aside (enough for both the base and derived portions).
    - The appropriate derived class constructor is called.
    - The base class object is constructed first using the appropriate base class constructor. If no base class constructor is specified, the default constructor will be used.
    - The initialization list of the derived class initializes members of the derived class.
    - The body of the derived class constructor executes.
    - Control is returned to the caller.
### 20.4. Inheritance and access specifiers
  - If we do not choose an inheritance type, C++ defaults to **private inheritance**
  - `private-inaccessible` does not affect the way that the derived class accesses members inherited from its parent! It only affects the code trying to access those members through the derived class.
  - e.g.
```cpp
// Inherit from Base publicly
class Pub: public Base
{
    // Public inheritance means:
    // Public inherited members stay public (so m_public is treated as public)
    // Protected inherited members stay protected (so m_protected is treated as protected)
    // Private inherited members stay inaccessible (so m_private is inaccessible)
};

// Inherit from Base protectedly
class Pro: protected Base
{
    // Protected inheritance means:
    // Public inherited members stay protected (so m_public is treated as protected)
    // Protected inherited members stay protected (so m_protected is treated as protected)
    // Private inherited members stay inaccessible (so m_private is inaccessible)
};

// Inherit from Base privately
class Pri: private Base
{
    // Private inheritance means:
    // Public inherited members become private (so m_public is treated as private)
    // Protected inherited members become private (so m_protected is treated as private)
    // Private inherited members stay inaccessible (so m_private is inaccessible)
};

class Def: Base // Defaults to private inheritance
{
};
```

### 20.5. Calling inherited functions and overriding behavior
- **Adding new functionality to a derived class:** we can inherit the base class functionality and then add new functionality, modify existing functionality, or hide functionality we don’t wan
- **Calling inherited functions:** Just call as base class do, we derived.baseFunction() is called, the compiler looks to see if a function named baseFunction() has been defined in the Derived class. It hasn’t. So it moves to the parent class (in this case, Base), and tries again there. Base has defined an baseFunction() function, so it uses that one.
- **Redefining behaviors:** To modify the way a function defined in a base class works in the derived class, simply redefine the function in the derived class.
- **Adding to existing functionality:** To have a derived function call a base function of the same name, simply do a normal function call, but prefix the function with the scope qualifier of the base class.
- **Overload resolution in derived classes:** By putting the using-declaration `using Base::function()`; inside `Derived`, we are telling the compiler that all `Base` functions named `function` should be visible in `Derived`, which will cause them to be eligible for overload resolution. As a result, `Base::function()` is selected over `Derived::function(double)` when call `baseObject.function(5)`.
- e.g.
```cpp
#include <iostream>
using namespace std;

// ===== Base class =====
class Base {
public:
    void baseFunction() {
        cout << "Base::baseFunction()" << endl;
    }

    void greet() {
        cout << "Hello from Base!" << endl;
    }

    void function(int x) {
        cout << "Base::function(int) called with " << x << endl;
    }
};

// ===== Derived class =====
class Derived : public Base {
public:
    // 1. Redefining (overriding) behavior
    void greet() {
        cout << "Hello from Derived!" << endl;
    }

    // 2. Adding to existing functionality
    void greetWithBase() {
        cout << "Derived part first -> ";
        Base::greet(); // call the Base version explicitly
    }

    // 3. Hiding base function by defining same name
    void baseFunction() {
        cout << "Derived::baseFunction()" << endl;
    }

    // 4. Overload resolution
    void function(double x) {
        cout << "Derived::function(double) called with " << x << endl;
    }

    // Bring Base::function(int) into scope for overload resolution
    using Base::function;
};

int main() {
    Derived d;

    cout << "\n--- Calling inherited function ---" << endl;
    d.Base::baseFunction();  // explicitly call base version
    d.baseFunction();        // calls Derived::baseFunction()

    cout << "\n--- Redefining behavior ---" << endl;
    d.greet();               // calls Derived version

    cout << "\n--- Adding to existing functionality ---" << endl;
    d.greetWithBase();       // calls Derived + Base

    cout << "\n--- Overload resolution ---" << endl;
    d.function(10);          // selects Base::function(int)
    d.function(3.14);        // selects Derived::function(double)

    return 0;
}
```

### 20.6. Hiding inherited functionality
- **Changing an inherited member’s access level**: we can change an inherited member’s access specifier in the derived class, by using a using declaration to identify the (scoped) base class member that is having its access changed in the derived class, under the new access specifier.
- e.g.
```cpp
#include <iostream>
class Base
{
private:
    int m_value {};

public:
    Base(int value)
        : m_value { value }
    {
    }

protected:
    void printValue() const { std::cout << m_value; }
};

class Derived: public Base
{
public:
    Derived(int value)
        : Base { value }
    {
    }

    // Base::printValue was inherited as protected, so the public has no access
    // But we're changing it to public via a using declaration
    using Base::printValue; // note: no parenthesis here
};
```

- **Hiding functionality:** we can hide functionality that exists in the base class, so that it can not be accessed through the derived class, by changing the relevant access specifier.
- e.g.
```cpp
#include <iostream>

class Base
{
public:
	int m_value{};
};

class Derived : public Base
{
private:
	using Base::m_value;

public:
	Derived(int value) : Base { value }
	{
	}
};

int main()
{
	Derived derived{ 7 };
	std::cout << derived.m_value; // error: m_value is private in Derived

	Base& base{ derived };
	std::cout << base.m_value; // okay: m_value is public in Base

	return 0;
}
```
- **Deleting functions in the derived class:** we can also mark member functions as deleted in the derived class, which ensures they can’t be called at all through a derived object. By using `delete` keyworld.
- We can still call the base class version either by explicitly qualifying it with the base class name or by upcasting the derived object to the base type.
- e.g.
```cpp
#include <iostream>
class Base
{
private:
	int m_value {};

public:
	Base(int value)
		: m_value { value }
	{
	}

	int getValue() const { return m_value; }
};

class Derived : public Base
{
public:
	Derived(int value)
		: Base { value }
	{
	}


	int getValue() const = delete; // mark this function as inaccessible
};

int main()
{
	Derived derived { 7 };

	// The following won't work because getValue() has been deleted!
	std::cout << derived.getValue();
    
	// 1. We can call the Base::getValue() function directly
	std::cout << derived.Base::getValue();

	// 2. we can upcast Derived to a Base reference and getValue() will resolve to Base::getValue()
    // Base& ref = static_cast<Base&>(derived);
    // std::cout << ref.getValue();

	std::cout << static_cast<Base&>(derived).getValue();

	return 0;
}
```

### 20.7. Multiple inheritance
- C++ provides the ability to do multiple inheritance. Multiple inheritance enables a derived class to inherit members from more than one parent.
- Avoid multiple inheritance unless alternatives lead to more complexity.

## 21. Virtual Functions - Polymorphism
- A virtual function that resolves to `the most-derived version of the function` that exists `between the base and derived class`.

> C++ allows you to set base class pointers and references to a derived object. This is useful when we want to write a function or array that can work with any type of object derived from a base class.
Without virtual functions, base class pointers and references to a derived class will only have access to base class member variables and versions of functions.
A virtual function is a special type of function that resolves to the most-derived version of the function (called an override) that exists between the base and derived class. To be considered an override, the derived class function must have the same signature and return type as the virtual base class function. The one exception is for covariant return types, which allow an override to return a pointer or reference to a derived class if the base class function returns a pointer or reference to the base class.
A function that is intended to be an override should use the override specifier to ensure that it is actually an override.
The final specifier can be used to prevent overrides of a function or inheritance from a class.
If you intend to use inheritance, you should make your destructor virtual, so the proper destructor is called if a pointer to the base class is deleted.
You can ignore virtual resolution by using the scope resolution operator to directly specify which class’s version of the function you want: e.g. base.Base::getName().
Early binding occurs when the compiler encounters a direct function call. The compiler or linker can resolve these function calls directly. Late binding occurs when a function pointer is called. In these cases, which function will be called can not be resolved until runtime. Virtual functions use late binding and a virtual table to determine which version of the function to call.
Using virtual functions has a cost: virtual functions take longer to call, and the necessity of the virtual table increases the size of every object containing a virtual function by one pointer.
A virtual function can be made pure virtual/abstract by adding “= 0” to the end of the virtual function prototype. A class containing a pure virtual function is called an abstract class, and can not be instantiated. A class that inherits pure virtual functions must concretely define them or it will also be considered abstract. Pure virtual functions can have a body, but they are still considered abstract.
An interface class is one with no member variables and all pure virtual functions. These are often named starting with a capital I.
A virtual base class is a base class that is only included once, no matter how many times it is inherited by an object.
When a derived class is assigned to a base class object, the base class only receives a copy of the base portion of the derived class. This is called object slicing.
Dynamic casting can be used to convert a pointer to a base class object into a pointer to a derived class object. This is called downcasting. A failed conversion will return a null pointer.
The easiest way to overload operator<< for inherited classes is to write an overloaded operator<< for the most-base class, and then call a virtual member function to do the printing.
### 21.1. Pointers and references to the base class of derived objects
- **Pointers, references, and derived classes:**  We can not only assign `Derived pointers and references` to `Derived objects`, but also assign `Base pointers and references` to `Derived objects`.
>  **chỉ có thể gọi các hàm/thành viên có trong Base -> need virtual**
> Derived chứa phần dữ liệu của Base ở đầu đối tượng, nên con trỏ Base* có thể trỏ đến vùng đó mà không sai về mặt địa chỉ.
> Compiler đảm bảo phần đầu tiên của Derived có cùng layout với Base.
- e.g.
```cpp
#include <iostream>

int main()
{
    Derived derived{ 5 };
    Derived& rDerived{derived};
    Derived* rDerived{&derived};


    // These are both legal!
    Base& rBase{ derived }; // rBase is an lvalue reference (not an rvalue reference)
    Base* pBase{ &derived };

    std::cout << "derived is a " << derived.getName() << " and has value " << derived.getValue() << '\n';
    std::cout << "rBase is a " << rBase.getName() << " and has value " << rBase.getValue() << '\n';
    std::cout << "pBase is a " << pBase->getName() << " and has value " << pBase->getValue() << '\n';

    return 0;
}

```
- **Use for pointers and references to base classes:** let us pass any derived object to a single function instead of writing one for each class.
However, since `rBase` is an `Based reference`, calling `Base.fucntion()` runs `Base::fucntion()` unless the function is `virtual`.

### 21.2. Virtual functions and polymorphism
- **Virtual functions:**
  - is a special type of member function that, when called, resolves to the `most-derived version` of the function for the actual type of the object being referenced or pointed to.
  - is considered a match if it has the same signature (name, parameter types, and whether it is const) and return type as the base version of the function. Such functions are called overrides.
  - to make a function virtual, simply place the `virtual` keyword before the function declaration.

- e.g.
```cpp
#include <iostream>
#include <string_view>

class Base
{
public:
    virtual std::string_view getName() const { return "Base"; } // note addition of virtual keyword
    virtual ~Base() = default;    // virtual destructor
};

class Derived: public Base
{
public:
    virtual std::string_view getName() const { return "Derived"; }
};

int main()
{
    Derived derived {};
    Base& rBase{ derived };
    std::cout << "rBase is a " << rBase.getName() << '\n';

    return 0;
}

// RESULT: rBase is a Derived
```
- **Return types of virtual functions:** the return type of a virtual function and its override must match, otherwise the compilation will fail. 
- **Do not call virtual functions from constructors or destructors:** because of the class hadn’t even been created yet

- **polymorphism** refers to the ability of an entity to have multiple forms (the term “polymorphism” literally means “many forms”)
  - **Compile-time polymorphism** refers to forms of polymorphism that are resolved by the compiler. These include **function overload resolution**, as well as **template resolution**.
  - **Runtime polymorphism** refers to forms of polymorphism that are resolved at runtime. This includes virtual function resolution.

### 21.3. The override and final specifiers, and covariant return types
- **The override specifier:**  the `override` specifier can be applied to any virtual function to tell the compiler to enforce that the function is an override.
  - `override` specifier is placed at the end of a member function declaration.
  - If a member function is const and an override, the const must come before override.
  - We should use the virtual keyword on virtual functions in a base class.

- **The final specifier:** The final specifier can be used to tell the compiler to enforce a virtual function to be unable to override.
  - used in the same place the override specifier is
- e.g. 
```cpp
#include <iostream>
using namespace std;

// Base class
class Animal {
public:
    virtual void speak() const { cout << "Animal speaking\n"; }
    virtual void move() { cout << "Animal moving\n"; }
};

// Derived class
class Dog : public Animal {
public:
    // override: tells compiler this must override a virtual function
    void speak() const override { cout << "Dog barking\n"; }

    // final: prevents further overriding in subclasses
    void move() final { cout << "Dog running\n"; }
};

// Further derived class
class Puppy : public Dog {
public:
    // OK: override speak()
    void speak() const override { cout << "Puppy yapping\n"; }

    // ERROR: move() is final in Dog, cannot override
    // void move() override { cout << "Puppy hopping\n"; }
};

int main() {
    Animal* a = new Puppy();
    a->speak(); // → "Puppy yapping"
    a->move();  // → "Dog running"
    delete a;
}

```
- **Covariant return types**
```cpp
#include <iostream>
#include <string_view>

class Base
{
public:
	// This version of getThis() returns a pointer to a Base class
	virtual Base* getThis() { std::cout << "called Base::getThis()\n"; return this; }
	void printType() { std::cout << "returned a Base\n"; }
};

class Derived : public Base
{
public:
	// Normally override functions have to return objects of the same type as the base function
	// However, because Derived is derived from Base, it's okay to return Derived* instead of Base*
	Derived* getThis() override { std::cout << "called Derived::getThis()\n";  return this; }
	void printType() { std::cout << "returned a Derived\n"; }
};

int main()
{
	Derived d{};
	Base* b{ &d };
	d.getThis()->printType(); // calls Derived::getThis(), returns a Derived*, calls Derived::printType
	b->getThis()->printType(); // calls Derived::getThis(), returns a Base*, calls Base::printType

	return 0;
}
```

### 21.4. Virtual destructors, virtual assignment, and overriding virtualization
- **Virtual destructors:** in the case that you will want to provide your own destructor (particularly if the class needs to deallocate memory).
- e.g.
```cpp
#include <iostream>
class Base
{
public:
    // virtual ~Base() = default; // generate a virtual default destructor
    virtual ~Base() // note: virtual
    {
        std::cout << "Calling ~Base()\n";
    }
};

class Derived: public Base
{
private:
    int* m_array {};

public:
    Derived(int length)
      : m_array{ new int[length] }
    {
    }

    virtual ~Derived() // note: virtual
    {
        std::cout << "Calling ~Derived()\n";
        delete[] m_array;
    }
};

int main()
{
    Derived* derived { new Derived(5) };
    Base* base { derived };

    delete base;

    return 0;
}

// OUTPUT:
// Calling ~Derived()
// Calling ~Base()
```

- **Ignoring virtualization**: incase we want to ignore the virtualization of a function.
- e.g.
```cpp
#include <string_view>
class Base
{
public:
    virtual ~Base() = default;
    virtual std::string_view getName() const { return "Base"; }
};

class Derived: public Base
{
public:
    virtual std::string_view getName() const { return "Derived"; }
};

#include <iostream>
int main()
{
    Derived derived {};
    const Base& base { derived };

    // Calls Base::getName() instead of the virtualized Derived::getName()
    std::cout << base.Base::getName() << '\n';

    return 0;
}
```
> If we intend our class to be inherited from, make sure our destructor is virtual and public.
If we do not intend our class to be inherited from, mark our class as final. 

### 21.5. Early binding and late binding
- `early binding (static binding)`: when a direct call is made to a non-member function or a non-virtual member function, the compiler can determine which function definition should be matched to the call. 
- e.g.
    ```cpp
    #include <iostream>
    using namespace std;
    
    class Animal {
    public:
        void speak() {        // NOT virtual
            cout << "Animal speaks\n";
        }
    };
    
    class Dog : public Animal {
    public:
        void speak() {        // Hides Animal::speak()
            cout << "Dog barks\n";
        }
    };
    
    void print(int x) {
        cout << "int\n";
    }
    
    void print(double x) {
        cout << "double\n";
    }
    
    int main() {
        Animal* a = new Dog();
        a->speak();   // Early binding: Non-virtual member function
    
        print(5);      // Early binding: int version chosen at compile time
    }
    ```


- `late binding (or in the case of virtual function resolution, dynamic dispatch)`: when a function call can’t be resolved until runtime. 
    ```cpp
    #include <iostream>
    using namespace std;
    
    class Animal {
    public:
        virtual void speak() {   // Virtual!
            cout << "Animal speaks\n";
        }
    };
    
    class Dog : public Animal {
    public:
        void speak() override {
            cout << "Dog barks\n";
        }
    };
    
    int main() {
        Animal* a = new Dog();
        a->speak();   // Late binding
    }
    ```
- `virtual table` is a lookup table of functions used to resolve function calls in a dynamic/late binding manner. 

> Early binding/static dispatch = direct function call overload resolution
Late binding = indirect function call resolution
Dynamic dispatch = virtual function override resolution 

### 21.6. Pure virtual functions, abstract base classes, and interface classes
- `Pure virtual (abstract) functions and abstract base classes`(like abstract class/function in java): 
	- syntax: assign the function the value 0. e.g. `virtual int getValue() const = 0; // a pure virtual function`
	- the class will becomes and abstract base class, it can not be instantiated. (create a object using abstract base class as the type)
	- the derived class must define a body for the pure virtual function, if not, it will be considered abstract base class as well.
	- Any class with pure virtual functions should also have a virtual destructor
- `Pure virtual functions with definitions` (like default in java):
	- We can create pure virtual functions that have definitions. Abstract function
	- When providing a definition for a pure virtual function, the definition must be provided separately (not inline)
- e.g.
```cpp
#include <iostream>
#include <string>
#include <string_view>

class Animal // This Animal is an abstract base class
{
protected:
    std::string m_name {};

public:
    Animal(std::string_view name)
        : m_name(name)
    {
    }

    const std::string& getName() const { return m_name; }
    virtual std::string_view speak() const = 0; // note that speak is a pure virtual function

    virtual ~Animal() = default;
};

std::string_view Animal::speak() const
{
    return "buzz"; // some default implementation
}

class Dragonfly: public Animal
{

public:
    Dragonfly(std::string_view name)
        : Animal{name}
    {
    }

    std::string_view speak() const override// this class is no longer abstract because we defined this function
    {
        return Animal::speak(); // use Animal's default implementation
    }
};

int main()
{
    Dragonfly dfly{"Sally"};
    std::cout << dfly.getName() << " says " << dfly.speak() << '\n';

    return 0;
}

```

- `Interface classes`: is a class that has no member variables, and where all of the functions are pure virtual
- e.g. 
```cpp
#include <string_view>

class IErrorLog
{
public:
    virtual bool openLog(std::string_view filename) = 0;
    virtual bool closeLog() = 0;

    virtual bool writeError(std::string_view errorMessage) = 0;

    virtual ~IErrorLog() {} // make a virtual destructor in case we delete an IErrorLog pointer, so the proper derived destructor is called
};
```

- `Virtual base classes`: 
  - are used in virtual inheritance in a way of preventing multiple "instances" of a given class appearing in an inheritance hierarchy when using multiple inheritances.  (e.g. we may want only one copy of `PoweredDevice` (contructing) to be shared by both `Scanner` and `Printer`. )
  - using `virtual` keyworld in the inheritance list of the derived class
  - there is only one base object. The base object is shared between all objects in the inheritance tree and it is only constructed once
  - for the constructor of the most derived class, virtual base classes are always created before non-virtual base classes, which ensures all bases get created before their derived classes.
  - the most derived class is responsible for constructing the virtual base class. (`Copier` is responsible for creating `PoweredDevice`)
- e.g.
```cpp
class PoweredDevice
{
};

class Scanner: virtual public PoweredDevice
{
};

class Printer: virtual public PoweredDevice
{
};

class Copier: public Scanner, public Printer
{
};
```

### 21.7. Object slicing
- `slicing` means the assigning of a Derived class object to a Base class object is called object. 
- When a derived class object is assigned to a base class object in C++, the derived class object's extra attributes are sliced off (not considered) to generate the base class object
- e.g.
```cpp
#include <iostream>
#include <iostream>
#include <string_view>

class Base
{
protected:
    int m_value{};

public:
    Base(int value)
        : m_value{ value }
    {
    }

    virtual ~Base() = default;

    virtual std::string_view getName() const { return "Base"; }
    int getValue() const { return m_value; }
};

class Derived: public Base
{
public:
    Derived(int value)
        : Base{ value }
    {
    }

   std::string_view getName() const override { return "Derived"; }
};

//  slicing is much more likely to occur accidentally with
void printNameAc(const Base base) // note: base passed by value, not reference
{
    std::cout << "Name is a " << base.getName() << '\n';
}

// slicing here can all be easily avoided by making the function parameter a reference instead of a pass by value
void printName(const Base& base) // note: base now passed by reference
{
    std::cout << "Name is a " << base.getName() << '\n';
}

int main()
{
    Derived derived{ 5 };
    std::cout << "derived is a " << derived.getName() << " and has value " << derived.getValue() << '\n';

    Base& ref{ derived };
    std::cout << "ref is a " << ref.getName() << " and has value " << ref.getValue() << '\n';

    Base* ptr{ &derived };
    std::cout << "ptr is a " << ptr->getName() << " and has value " << ptr->getValue() << '\n';
	// Because ref and ptr are of type Base, ref and ptr can only see the Base part of derived -- the Derived part of derived still exists, but simply can’t be seen through ref or // ptr. However, through use of virtual functions, we can access the most-derived version of a function. 
	
	 Base base{ derived }; // what happens here?
    std::cout << "base is a " << base.getName() << " and has value " << base.getValue() << '\n';
	// only the Base portion of the Derived object is copied. The Derived portion is not
	
	// WARNING
	printNameAc(derived); // we expect that this should print "Name is a Derived", but accidentally led to unbehaviour // Derived
	
	// OK
	printName(derived); 
   return 0;
}

// OUTPUT
derived is a Derived and has value 5
ref is a Derived and has value 5
ptr is a Derived and has value 5
base is a Base and has value 5
Name is a Base
Name is a Derived
```

### 21.8. Dynamic casting
- Parent -> Child: DownCasting
- Child -> Parent: UpCasting
- We can use `dynamic_cast` to perform downcasting.
  - If a dynamic_cast fails, the result of the conversion will be a null pointer.
  - e.g. 
```cpp
    int main()
    {
    	Base* b{ getObject(true) };
    
    	Derived* d{ dynamic_cast<Derived*>(b) }; // use dynamic cast to convert Base pointer into Derived pointer
    
    	if (d) // make sure d is non-null
    		std::cout << "The name of the Derived is: " << d->getName() << '\n';
        
        // DOwncasting for references
        Derived apple{1, "Apple"}; // create an apple
    	Base& b{ apple }; // set base reference to object
    	Derived& d{ dynamic_cast<Derived&>(b) }; // dynamic cast using a reference instead of a pointer
    	std::cout << "The name of the Derived is: " << d.getName() << '\n'; // we can access Derived::getName through d

    	delete b;
    
    	return 0;
    }
```

- We can also use the `static_cast` to do perform downcasting.
  -  it will “succeed” even if the Base pointer isn’t pointing to a Derived object.
  -  the result will be in undefined behavior
  
- `dynamic_cast vs static_cast`: use `static_cast` unless you’re `downcasting`, in which case `dynamic_cast` is usually a better choice. However, you should also consider avoiding casting altogether and just use `virtual functions`.

### 21.8. Printing inherited classes using operator<<
- Refer to the learncpp.com

## 22. Overloading Operators
- Almost any existing operator in C++ can be overloaded. The exceptions are:` conditional (?:), sizeof, scope (::), member selector (.), pointer member selector (.*), typeid, and the casting operators`.
- We can only overload the operators that exist.
- At least one of the operands in an overloaded operator must be a user-defined type.

- Overloading the plus operator (+) is as simple as declaring a function named `operator+`, giving it two parameters of the type of the operands we want to add, picking `an appropriate return type`, and then writing the function.

- Not everything can be overloaded as a friend function: The assignment (=), subscript ([]), function call (()), and member selection (->) operators must be overloaded as member functions, because the language requires them to be.

> The following rules of thumb can help you determine which form is best for a given situation:
If you’re overloading assignment (=), subscript ([]), function call (()), or member selection (->), do so as a member function.
If you’re overloading a unary operator, do so as a member function.
If you’re overloading a binary operator that does not modify its left operand (e.g. operator+), do so as a normal function (preferred) or friend function.
If you’re overloading a binary operator that modifies its left operand, but you can’t add members to the class definition of the left operand (e.g. operator<<, which has a left operand of type ostream), do so as a normal function (preferred) or friend function.
If you’re overloading a binary operator that modifies its left operand (e.g. operator+=), and you can modify the definition of the left operand, do so as a member function.

- e.g.
```cpp
#include <iostream>

class Cents
{
private:
	int m_cents {};

public:
	Cents(int cents) : m_cents{ cents } { }

	// add Cents + Cents using a friend function
	friend Cents operator+(const Cents& c1, const Cents& c2);
	
	// 3. Using member function
	// Overload Cents + int
    Cents operator+(int value) const;

	int getCents() const { return m_cents; }
};

// 1. Using friend function note: this function is not a member function!
Cents operator+(const Cents& c1, const Cents& c2)
{
	// use the Cents constructor and operator+(int, int)
	// we can access m_cents directly because this is a friend function
	return c1.m_cents + c2.m_cents;
}

// 2. Using normal function
// note: this function is not a member function nor a friend function!
Cents operator-(const Cents& c1, const Cents& c2)
{
  // use the Cents constructor and operator+(int, int)
  // we don't need direct access to private members here
  return { c1.getCents() - c2.getCents() };
}

// 3. Using member function
// note: this function is a member function!
// the cents parameter in the friend version is now the implicit *this parameter
Cents Cents::operator+ (int value) const
{
    return Cents { m_cents + value };
}

int main()
{
	Cents cents1{ 6 };
	Cents cents2{ 8 };
	Cents centsSum{ cents1 + cents2 };
	std::cout << "I have " << centsSum.getCents() << " cents.\n";
	
	Cents centsSub{ cents1 - cents2 };
	std::cout << "I have " << centsSub.getCents() << " cents.\n";
	
	const Cents cents3 { cents1 + 2 };
	std::cout << "I have " << cents3.getCents() << " cents.\n";

	return 0;
}
```

### 22.1. Arithmetic Using Friend Functions ( + - * /)
- All of the arithmetic operators are binary operators, so they take two operands.
- There are 2 common ways to overide the operators
  - Member function: The assignment (=), subscript ([]), function call (()), and member selection (->) operators must be overloaded as member functions, because the language requires them to be.
  - Nonmember function (friend): we won’t be able to use a member overload if the left operand is either not a class (e.g. int), or it is a class that we can’t modify (e.g. std::ostream).
- A member operator only works if the left side is an object of the class.
- e.g.
```cpp

```
## 23. I/O 
### 23.1. I/O Streams
- It is a part of the STL.
- I/O is implemented with `streams`.
<br>

- **stream** is a sequence of bytes that can be accessed sequentially. It may produce or consume amounts data over time.
	- **input stream**: used to hold input data from a data producer.
	- **output stream**: used to hold output data for a particular data consumer.
	- *e.g.* when writing/input data to an device, the device may not be ready to accept that data, so the data will sit in the stream. 
	- [C++] <=> streams <=> [os]
<br>

- **I/O in C++**: we can use the STL classes to deal with streams
	- **input stream**: `istream` & extraction operation (`>>`) is used to remove values from the stream
	- **output stream**: `ostream` & insertio operation (`<<`) is used to put vlaues in the stream.\
	- `iostream` can handle both i & o
<br>

- **Standard streams**:
	- `cin`: an `istream` object tied to the standard input
	- `cout`: an `ostream` object tied to the standard output
	- `cerr`: an `ostream` object tied to the standard error - unbuffered output
	- `clog`: an `ostream` object tied to the standard error - buffered output

### 23.2. Input with istream
- Use `extraction operator (>>)` to read information from an input stream. It skips **whitespace (blanks, tabs, and newlines)**. Use `get(), getLine()` to not discard the whitespace.
- `manipulator` is an object that is used to modify a stream when applied with the `extraction (>>)` or `insertion (<<)` operators. <iomanip>

### 23.3. Output with ostream 
https://www.learncpp.com/cpp-tutorial/output-with-ostream-and-ios/

### 23.4. Stream classes for string <sstream>
- String streams provide a buffer to hold data but are not connected to an I/O channel.
- The primary uses of the string stream:
  - Display data later then or proccess i/o data.
  - Get data into a string stream
  - Get data from a string stream
  - Convertion strings/numbers
  - Clear a string stream

- e.g:
```cpp
#include <sstream>
#include <string>
int main() {
  std::stringstream os{};
  // input
  os << "0xF";
  std::cout << os.str();
  
  os.str("0x1 0x2");
  std::cout << os.str();

  // output
  std::string bytesStr  = os.str();
  std::cout << bytesStr;

  os.str("0x0 0xF 0xE 0x2");
  os >> bytesStr;
  std::cout <<bytesStr;

  // conversions
  int byte_low = 0xFFF;
  int byte_high = 0x001;
  os.clear();
  os << byte_low << ' ' << byte_high;
  std::cout << os.str();
}
```

### 23.5. Validation
https://www.learncpp.com/cpp-tutorial/stream-states-and-input-validation/

### 23.6. File I/O <fstream>
- There are three basics file I/O classes:
  - `ifstream`: derived from `istream`
  - `ofstream`: derived from `ostream`
  - `fstream`: derived from `stream`
<br>

- **important:** We have to explicitly setup to use file streams:
  - 1. **Open a file: instantiate an object of file I/O class, with the name of the file as a param**
  - 2. **Use the insertion (<<) or extraction (>>) to r/w**
  - 3. **Close a file: call the close() or let file I/O go out of scope.**
  - 4. **To delete a file, simply use the remove() function.**
<br>

- **File output:**
```cpp
#include <fstream>
#include <iostream>
#include <string>

static void fileOutput() {
  std::ofstream outfile{"grs_bytes.csv"};
  if (!outfile || !outfile.is_open()) {
    std::cerr << "[E] Cannot create the output file";
    return;
  }

  // Put bytes data to the file
  // put string
  std::string elfBytes{
      R"""(
        time_s,
        gsr_value 0.0, 45.27761157741693 0.005, 41.69912812397066 0.01,
        38.13110177547114 0.015, 35.32162785580394 0.02,
        31.75617843363382 0.025, 28.352875321528607 0.03,
        25.23210282654006 0.035, 21.769688905641132 0.04,
        17.99031153059391 0.045, 15.073732055666543 0.05,
        15.13550182371759 0.055, 14.69547985289048 0.06,
        14.867397107985468 0.065, 14.982082556093832 0.07,
        14.893751010484861 0.075, 14.877034044202343 0.08,
        14.820590581790071 0.085, 15.1065350504897 0.09,
        15.152287796727098 0.095, 14.764395300078201 0.1,
        15.118189654760348 0.105, 15.473255351586635 0.11,
        14.913896402347113
    \n )"""};
  outfile << elfBytes;

  elfBytes =
      "0x0 0x0"
      "0x0 0x0"
      "0x0 0x0";
  outfile << elfBytes;

  elfBytes =
      "0xF 0xA \
              0xE 0xB \
              0x0 0x0";

  outfile << elfBytes;

  outfile.put('E');  // put char
  outfile.close();
}
```
<br>

- **File input:**
```cpp
static void fileInput() {
  std::ifstream inFile{"grs_bytes.csv"};
  if (!inFile.is_open()) {
    std::cerr << "Error to open file";
  }

  std::string inputStr{};
  std::cout << "========" << std::endl;
  while (
      inFile >>
      inputStr) {  //  Note that ifstream returns a 0 if we’ve reached the end of the file (EOF)
    std::cout << inputStr;
  }
  std::cout << "========" << std::endl;

  // not skip whitespace
  inFile.close();
  inFile.open("grs_bytes.csv"); // explicitly call open()
  inputStr.clear();
  while (std::getline(inFile, inputStr)) {
    std::cout << inputStr << std::endl;
  }
  std::cout << "========" << std::endl;

  inFile.close();
}
```
<br>

- **File Mode:**
  - `app`: opens the file in append mode
  - `ate`: seeks to the end of the file before reading/writing
  - `binary`: opens the file in binary mode (instead of text mode)
  - `in`: opens the file in read mode (default for ifstream)
  - `out`: opens the file in write mode (default for ofstream)
  - `trunc`: erases the file if it already exists
<br>

### 23.6 Ramdom File I/O
- **File pointer:** Each file stream class contains a file pointer that is used to keep track of the current read/write position within the file
- **Ramdom access** with `seekg() & seekp()`
- **IOS seek flags:**
	- `beg`: the offset is relative to the beginning of the file (default)
	- `cur`: the offset is relative to the current location of the file pointer
	- `end`: the offset is relative to the end of the file

> In programming, a newline (‘\n’) is actually an abstraction.
On Windows, a newline is represented as sequential CR (carriage return) and LF (line feed) characters (thus taking 2 bytes of storage).
On Unix, a newline is represented as a LF (line feed) character (thus taking 1 byte of storage).

> // assume iofile is an object of type fstream
iofile.seekg(iofile.tellg(), std::ios::beg); // seek to current file position
<br>