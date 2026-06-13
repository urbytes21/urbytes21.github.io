---
author: "Phong Nguyen"
title: "C++ - Chapter 5: Data Types"
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
# Data Types
A data type defines the kind of value a variable can store, how much memory it uses, and what operations can be performed on it.

## 1. Type Conversion
```cpp
Type conversions
│
├── Implicit conversions (compiler performed automatically)
│   │
│   ├── Numeric promotions
│   │    ├── bool  -> int
│   │    ├── char  -> int
│   │    └── float -> double
│   │
│   └── Numeric conversions
│        ├── Widening conversions
│        │    ├── int -> long long
│        │    └── int -> double
│        │
│        └── Narrowing conversions
│             ├── double -> int
│             ├── int -> char
│             └── long long -> int
│
└── Explicit conversions (casts)
     ├── static_cast<T>(expr)
     ├── dynamic_cast<T>(expr)
     ├── const_cast<T>(expr)
     ├── reinterpret_cast<T>(expr)
     └── (T)expr                    ///< C-style cast
```

### 1.1 Implicit
**Implicit type conversion** is performed `automatically` by the compiler when an expression of some type is supplied in a context where some other type is expected.
- A **numeric promotion** converts `smaller numeric types` to `larger numeric types` (typically `int` or `double`). It guarantees to preserve the value being converted.
- A **numeric conversion** is any arithmetic type conversion that is not a numeric promotion. Numeric conversions may lose data or precision.
    ```cpp
    /// @brief Numeric promotion: 
    char c = 100;     // char
    int i = c;        // char -> int   

    /// @brief Numeric conversion: 
    double d = 3.14;
    int x = d;        // double -> int
    ```
<br>

### 1.2 Explicit
**Explicit type conversion** is requested directly by the programmer
C++ supports five cast operators:
  - `static_cast`: used for well-defined conversions between related types
  - `dynamic_cast`: used for safe downcasting in polymorphic class hierarchies
  - `const_cast`: used to add or remove `const`
  - `reinterpret_cast`: used for low-level reinterpretation of bits or addresses
  - C-style cast: inherited from C
    ```cpp
    /// @brief static_cast
    double d = 3.14;
    int i = static_cast<int>(d);

    /// @brief dynamic_cast
    Base* b = new Derived;
    Derived* p = dynamic_cast<Derived*>(b);

    /// @brief const_cast
    const int x = 10;
    int* px = const_cast<int*>(&x);

    /// @brief reinterpret_cast
    int x = 65;
    char* p = reinterpret_cast<char*>(&x);

    /// @brief C-style cast
    double d = 3.14;
    int i = (int)d;
    ```

---
## 2. Type Aliases
**A type alias** creates an alternative name for an existing type.
C++ supports two ways to create type aliases:
- **Type Alias**:
   - Introduced in C++11.
   - Uses the `using` keyword.

- **Typedef**:
  - The traditional way of creating type aliases.
  - Uses the `typedef` keyword.

    ```cpp
    /// @brief Type Alias
    using MyDouble = double;
    using IntVector = std::vector<int>;

    template <typename T>
    using Vec = std::vector<T>;

    Vec<int> numbers;
    Vec<double> values;

    /// @brief Typedef
    typedef double MyDouble;
    typedef std::vector<int> IntVector;
    ```

---
## 3. Type Deduction
**Type deduction** allows the compiler to determine the type of an object from its initializer.
- Type deduction can be performed using the `auto` keyword or `decltype`.
  - `auto` deduces a type from an initializer.
  - `decltype` deduces the exact type of an expression.

### 3.1. auto (C++11)
The `auto` keyword allows the compiler to automatically deduce the type of a variable from its initializer. 
- `auto` must have an initializer so the compiler has a type to deduce from.
    ```cpp
    auto i{42};      // int , 42 is the initializer
    auto d{3.14};    // double
    auto s{"hello"}; // const char*
    ```

- Top-level `const` is dropped during type deduction.
    ```cpp
    const int x{5};

    auto a{x};       // int
    const auto b{x}; // const int
    ```

- References are dropped unless explicitly requested.
    ```cpp
    int value{10};
    int& ref{value};

    auto a{ref};   // int
    auto& b{ref};  // int&
    ```

- Use `auto*` when deducing pointers.
    ```cpp
    int value{10};
    int* ptr{&value};

    auto p{ptr};   // int*
    auto* q{ptr};  // int*
    ```

- The `auto` keyword can also be used with a **trailing return type**, where the return type is written after the parameter list.
    ```cpp
    int add(int x, int y) {
        return x + y;
    }

    // Equivalent
    auto add(int x, int y) -> int {
        return x + y;
    }
    ```
<br>

### 3.2. `decltype`
`decltype(expr)` evaluates the type of an expression at compile time without executing the expression.
- It is commonly used in templates and generic programming when the exact type of an expression is not known in advance.
- Unlike `auto`, `decltype` preserves references and const qualifiers.
- Most commonly used in template and library code.
- For ordinary application code, `auto` is often sufficient.
    ```cpp
    int x{1};
    double y{2.0};
    decltype(x + y) result{3.5};    // Since the expression has type `double`, the declaration above is equivalent double
    ```

---
## 4. Runtime Type Flexibility
C++ is a **statically typed language**. The type of a variable is normally known at compile time.
C++ also provides several mechanisms that allow a program to work with objects whose types are not known until runtime.
<br>

### 4.1. **`void*`**
**A void pointer** can store the address of an object of any type.
```cpp
int value = 100;
void ptr(&value);
int int_ptr = static_cast<int*>(ptr); // not type-safe
```
<br>

### 4.2. **`<any>` (C++17)**
**std::any** is a type-safe container that can hold a value of any copyable type.
The non-member `any_cast` functions provide type-safe access to the contained object.
```cpp  
// any types
std::any a = 1;
std::cout << a.type().name() << ": " << std::any_cast<int>(a) << '\n';

// bad cast
try {
    a = 1;
    std::cout << std::any_cast<float>(a) << "\n";
} catch(const std::bad_any_cast& e){
    std::cout << e.what() << "\n";
}
// has value
a = 2;
if (a.has_value()){
    std::cout << a.type().name() << ": " << std::any_cast<int>(a) << '\n';
}

// reset
a.reset();
if (!a.has_value()){
    std::cout << "no value\n";
}

// pointer to contained data
a = 3;
int* i = std::any_cast<int>(&a);
std::cout << *i << '\n';
```
<br>

### 4.3. `dynamic_cast`
**Dynamic cast** performs safe conversions within a polymorphic inheritance hierarchy.
- Uses `RTTI` to verify conversions at runtime.
    ```cpp
    Base* ptr = new Derived;
    Derived* d = dynamic_cast<Derived*>(ptr);
    ```
<br>

### 4.4. Virtual Functions
Virtual functions are enable runtime polymorphism.
- The function called depends on the object's dynamic type.
    ```cpp
    Base* ptr = new Derived;
    ptr->print();
    ```
<br>

### 4.5. `std::variant` (C++17)
**std::variant** is a type-safe union that can store one value from a fixed set of types.
```cpp
std::variant<int, double, std::string> value;

value = 42;
value = "Hello";
```

---
## 5. Run-Time Type Information (RTTI)
**RTTI** provides information about an object's actual type during runtime. It is primarily supported through:
- `dynamic_cast`
- `typeid`
<br>

### 5.1. `typeid`
**typeid(obj)** returns type information for an expression or object.
- Defined in the `<typeinfo>` header.
- The returned type information is represented by `std::type_info`.
    ```cpp
    #include <typeinfo>

    int x{10};
    std::cout << typeid(x).name();

    /// @brief When RTTI is enabled and `Base` is polymorphic (has at least one virtual function),
    /// `typeid(*ptr)` reports the dynamic type (`Derived`) rather than the static type (`Base`).
    Base* ptr = new Derived;
    std::cout << typeid(*ptr).name();
    ```
- `typeid(ptr)` returns the type of the pointer (`Base*`).
- `typeid(*ptr)` returns the type of the object being pointed to (`Derived`).
- The result of `.name()` is implementation-defined and may not be human-readable.

---
## 6. Compile-Time Type Flexibility
### 6.1. Type Traits (`<type_traits>`)
**A type traits** provide information about types at compile time.
It 's commonly used in templates and generic programming.
```cpp
std::is_integral_v<int>;      // true
std::is_pointer_v<int*>;      // true
std::is_const_v<const int>;   // true

/// @brief Type transformations
std::remove_const_t<const int>; // int
std::add_pointer_t<int>;        // int*
```

### 6.2. Concepts (C++20)
- Specify compile-time requirements for template parameters.
- Improve template error messages and readability.
    ```cpp
    template<typename T>
    requires std::integral<T>
    T add(T a, T b)
    {
        return a + b;
    }
    ```
---

