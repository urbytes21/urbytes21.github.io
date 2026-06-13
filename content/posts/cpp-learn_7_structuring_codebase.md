---
author: "Phong Nguyen"
title: "C++ - Chapter 7: Structuring Codebase"
date: "2026-06-13"
description: "C++ Notes"
tags: ["cpp"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
# Structuring Codebase

## 1. Scope, Storage Duration
A variable's **storage duration** determines when it is created and destroyed.

### 1.1. Automatic Storage Duration
Variables are created when execution reaches their definition and destroyed when their enclosing block exits.
Includes:
- Local variables
- Function parameters

### 1.2. Static Storage Duration
Variables are created when the program begins and destroyed when the program ends.
Includes:
- Global variables
- Namespace variables
- Static local variables

### 1.3. Dynamic Storage Duration
Variables are created and destroyed under programmer control.
Includes:
- Dynamically allocated variables (`new`, `delete`)

### 1.3. Thread Storage Duration
Each thread has its own instance of the variable.

### 1.4. Storage Class Specifiers
| Specifier | Meaning |
|------------|----------|
| `extern` | Declares a name with external linkage |
| `static` | Gives internal linkage (namespace scope) or static storage duration (local scope) |
| `thread_local` | Specifies thread storage duration |
| `mutable` | Allows a class member to be modified even when the containing object is `const` |
| `auto` | Type deduction (since C++11) |
| `register` | Deprecated in C++17 |

### 1.5. Translation Units
A **translation unit** consists of:
- One source file (`.cpp`)
- Plus all headers included directly or indirectly through `#include`
```cpp
main.cpp
   |
   +-- config.h
   |
   +-- math.h

        |
        v
+-------------------+
|   Translation     |
|      Unit         |
+-------------------+
| main.cpp          |
| config.h          |
| math.h            |
+-------------------+
/// After preprocessing, the compiler sees a single expanded source file, which is called a translation unit.
```

---
## 2. Linkage
- An identifier's linkage determines whether a declaration of that same identifier in a different scope refers to the same entity (object, function, reference,..) or not.
- An identifier with **no linkage** means another declaration of the same identifier refers to a unique entity. 
  - Local variables
  - Program-defined type identifiers (such as enums and classes) declared inside a block
<br>

### 2.1. Internal Linkage
- An identifier with **internal linkage** can only be accessed within the translation unit (source file .c/.cpp) in which it is declared.
    - static global variables
    - static functions
    - const global variables
    - unnamed namespaces and anything defined within them
- Unlike C++, in C, `const` does not imply internal linkage.
<br>

- `static` <global_variable>: make the global variable  to internal linkage
- `static` <local_variable>: changes its duration from automatic duration to static duration. And its initializer is only executed once.
- `static` <const/constexpr local_varialbe>: used to avoid expensive local object initialization each time a function is called because it inits once time.

- Internal linkage for `const global variables` can change to `external` with the keyword `extern`. e.g. `extern const PI = 3.14;`
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
<br>

### 2.2. External linkage
- An identifier with **external linkage** can be accessed from multiple translation units.
- All declarations of the same identifier throughout the program refer to the same object or function.
- Include:
    - non-static functions
    - non-const global variables
    - extern const global variables
    - inline const global variables
    - namespaces
- Functions have external linkage by default.
- Global Variables:
  - `non-const globals`: have external by default.
  - `const/constexpr globals` have internal by default
  - To give a const global variable external linkage, use extern.

- To access a global variable defined in another source file, use an `extern` declaration without an initializer.
    ```cpp
    // a.cpp ============================================================
    #include <iostream>

    int g_external { 100 };            // external by default
    extern const int g_limit { 200 };  // const made external

    void say_hello() {                  // external by default
        cout << "Hello from a.cpp\n";
    }

    // main.cpp ============================================================
    #include <iostream>

    extern int g_external;       // forward declaration 
    extern const int g_limit;    // forward declaration > < const int g_limit: definition
    void say_hello();            // forward declaration

    int main() {
        say_hello();
        std::cout << g_external << " / " << g_limit << '\n';
        return 0;
    }
    ```

---
## 3. Inline
- A function call incurs overhead, including argument passing, saving the return address, transferring control to the function, and returning to the caller. For small functions that are called frequently, this overhead may be noticeable.

- **Inline expansion** is a compiler optimization in which a function call is replaced with the function's body, eliminating function call overhead.
  - It may improve performance for small, frequently called functions.
  - Modern optimizing compilers automatically decide whether a function should be inline-expanded.
  - Therefore, the `inline` keyword should not be used solely to request inline expansion.

- **Modern use of `inline`** allows identical function or variable definitions to be placed in header files and shared across multiple translation units without causing multiple-definition errors.
  - Functions with external linkage should generally not be fully defined in header files, as including the header in multiple translation units may result in `multiple-definition errors`.
  - This makes `inline` particularly useful for functions defined in header files and for header-only libraries.

---
## 4. Sharing Global Constants
### 4.1. Global Constants As inline Variables 
    ```cpp
    // constants.h:============================================================
    #ifndef CONSTANTS_H
    #define CONSTANTS_H

    // define your own namespace to hold constants
    namespace constants
    {
        inline constexpr double pi { 3.14159 };
    }
    #endif

    // main.cpp::============================================================
    #include "constants.h"
    ```
  - Prefer defining inline constexpr global variables in a header file.
  - **Advantages**:
    - Can be used in constant expressions in any translation unit that includes them.
    - Only one copy of each variable is required.
  - **Downsides**:
    - Only works in C++17 onward.
    - Changing anything in the header file requires recompiling files including the header.
<br>

### 4.2. Global Constants As Internal Variables 
    ```cpp
    // constants.h:============================================================
    #ifndef CONSTANTS_H
    #define CONSTANTS_H

    // Define your own namespace to hold constants
    namespace constants
    {
        // Global constants have internal linkage by default
        constexpr double pi { 3.14159 };
    }
    #endif

    // main.cpp::============================================================
    #include "constants.h" // include a copy of each constant in this file
    #include <iostream>
    ```

  - **Advantages**:
    - Works prior to C++16.
    - Can be used in constant expressions in any translation unit that includes them.
  - **Downsides**:
    - Changing anything in the header file requires recompiling files including the header.
    - Each translation unit including the header gets its own copy of the variable.
<br>

### 4.3. Global Constants As External Variables 
```cpp
// constants.h:============================================================
#ifndef CONSTANTS_H
#define CONSTANTS_H

namespace constants
{
    // Since the actual variables are inside a namespace, the forward declarations need to be inside a namespace as well
    // We can't forward declare variables as constexpr, but we can forward declare them as (runtime) const
    extern const double pi;
}

#endif

// constants.cpp:============================================================
#include "constants.h"

namespace constants
{
    extern constexpr double pi { 3.14159 }; /// Use extern to ensure these have external linkage
}

// main.cpp::============================================================
#include "constants.h" // include all the forward declarations
```

**Advantages**:
  - Works prior to C++16.
  - Only one copy of each variable is required.
  - Only requires recompilation of one file if the value of a constant changes.

**Downsides**:
  - Forward declarations and variable definitions are in separate files, and must be kept in sync.
  - Variables not usable in constant expressions outside of the file in which they are defined.

---
## 5. Forward Declaration
**Forward declaration** allows us to tell the compiler about the existence of an identifier before actually defining the identifier.
```cpp
returnType functionName(); ///< Forward Declaration

/// @brief Function Definition
returnType functionName(){} 
```

---
## 6. Namespace
**A namespace** is a declarative region that provides a scope for identifiers (types, functions, variables, etc.) declared within it. It is used to organize code and avoid name conflicts between identifiers.
- Syntax:
    ```cpp
    namespace my_namespace{ const int pi = 100;}
    ```
- The **scope resolution operator (`::`)** is used to access namespace members.
- **Using directives** allow names from a namespace to be used without a namespace prefix.
    ```cpp
    using namespace my_namespace;
    int pi_copy = pi; // without my_namespace::pi;
    ```
<br>

**Namespace aliases** provide a shorter name for a namespace. `using namespace m_n = my_namespace;`

**Nested namespaces** allow namespaces to be declared inside other namespaces.

**Inline namespaces** (C++11) are commonly used for versioning and allow members of a nested namespace to be accessed through the enclosing namespace.
```cpp
    namespace MyLibrary {
        inline namespace v2 {
            void print() {}
        }

        namespace v1 {
            void print() {}
        }
    }
    MyLibrary::print();      // Calls v2::print() , access via MyLibrary namespace by using inline
    MyLibrary::v1::print();  // Calls v1::print()
    MyLibrary::v2::print();  // Calls v2::print()
``` 

**Anonymous (unnamed) namespaces** provide internal linkage for their members within a translation unit.

---
## 7. Preprocessor
The `preprocessor` is a process that runs on the code before it is compiled.
```cpp
#include <iostream>     // insert file contents
#define NAME "Alex"     // replace NAME -> "Alex"

#ifdef NAME_DEFINED     // only compile if defined
std::cout << NAME;
#endif
```

---
## 8. Headers / CPP Files
**Header files** are files designed to propagate declarations to code files. 
`Header guards` prevent the contents of a header from being included more than once into a given code file. 
- For cross-platform library code, `#ifndef` is safest.
- For modern projects, `#pragma once` is simpler and safe.

  ```cpp
  #include <iostream>     // search system only 
  #include "my_header.h"  // search local first, then system

  /// @brief Add include directories with -I
  // g++ -o main -I./source/includes -I/home/abc/moreHeaders main.cpp
  ```
----