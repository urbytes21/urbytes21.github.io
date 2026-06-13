# Template
A C++ template is a mechanism for creating generic functions and classes that can operate on different data types without duplicating code.
- Template types are often called **generic types** `T`, and programming with templates is known as **generic programming**.
- A **placeholder type** `T` can be used for function parameters, return types, and local variables whose actual type will be determined later.
- A **template parameter declaration** `template <typename T>` defines the template parameters that can be used within the template.

## 1. Function Template
A function template defines a family of functions.
**Syntax:**
    ```cpp
    template <parameter-list>
    function-declaration
    ```
- E.g.
    ```cpp
    /// @brief The function template declaration
    template <int N, typename T, typename U>
    auto max(T x, U y) {
        std::cout << "N = " << N << '\n';
        return (x < y) ? y : x;
    }

    int main() {
        // Explicit template arguments
        auto result1 = max<5, int, double>(2, 3.5);
        std::cout << result1 << '\n';

        // Template argument deduction
        auto result2 = max<10>(4, 5.5);
        std::cout << result2 << '\n';
        return 0;
    }
    ```
- `template <typename T, typename U, int N>` is the **template parameter declaration**.
- `T` and `U` are **generic (placeholder) types**.
- `N` is a **non-type template parameter** representing a compile-time constant value.
- **Template argument deduction** allows the compiler to determine template arguments automatically from the function call.

---

## 2. Class Template
**A class template** is a template definition for instantiating class types (structs, classes, or unions). 
Class template argument deduction (CTAD) is a C++17 feature that allows the compiler to deduce the template type arguments from an initializer.
**Syntax:**
    ```cpp
    template <parameter-list>
    class-declaration
    ```
- e.g.
    ```cpp
    template <typename T>
    struct Pair {
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
        Pair<int> p1{ 5, 6 };        // instantiates Pair<int>
        Pair<double> p1{ 5, 6 };     // instantiates Pair<int>
        auto result = max<int>(p1); 
    }

    // ===================================================================================
    // Compiler
    // ===================================================================================

    // A declaration for our Pair class template
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
    ```


- **Alias templates** define a family of type aliases.
    ```cpp
    template <typename T>
    using Coord = Pair<T>;

    Coord<int> point; // Equivalent to Pair<int>
    ```

---
## 3. Variadic Templates (C++11)
A **variadic template** is a template that can accept a variable number of template arguments.
- A **template parameter pack** is a template parameter that accepts zero or more template arguments (types, non-types, or templates).
- A **function parameter pack** is a function parameter that accepts zero or more function arguments.
**Syntax:**
    ```cpp
    type... pack_name
    template <typename... Args>
    class MyClass;
    ```
- e.g.
    ```cpp
    template <typename... Args>  // template parameter pack
    void print(Args... args) // function parameter pack
    {
        (std::cout << ... << args);
    }

    int main()
    {
        print(1, " ", 2.5, '\n');
        return 0;
    }
    ```
  
---
## 4. Template Specialization
Template specialization allows customized behavior for specific template arguments.
### 4.1. Full Template Specialization
A full specialization provides a completely different implementation for a specific type.
```cpp
template<typename T>
struct Printer {
    static void print() {}
};

template<>
struct Printer<int> {
    static void print() {}
};

// if (T == int)
//     use "specialized version";
// else
//     use primary template;
```

### 4.2. Partial Template Specialization
A partial specialization customizes a subset of template parameters.
Partial specialization is only allowed for class templates.

```cpp
template<typename T, typename U>
struct Pair {};

template<typename T>
struct Pair<T, int> {};

// if (U == int)
//     use "partial specialization";
// else
//     use primary template;
```

---
### 5. Type Traits
Type traits provide compile-time information about types.
<type_traits> defines a series of classes to obtain type information on compile-time.

```cpp
// https://cplusplus.com/reference/type_traits/
std::is_integral
std::is_floating_point
std::is_pointer
std::is_reference
std::is_const
std::is_same
```

---
### 6. SFINAE - "Substitution Failure Is Not An Error"
This rule applies during overload resolution of function templates: 
`When substituting the explicitly specified or deduced type for the template parameter fails, the specialization is discarded from the overload set instead of causing a compile error.`
This feature is used in template meta-programming.
- Conditional function overloads
- Compile-time type selection
- Template constraints
- Generic library development
- e.g.
    ```cpp
    // https://en.cppreference.com/cpp/language/sfinae
    template<typename T>
    typename std::enable_if<std::is_integral<T>::value>::type   // default T is void in std::enable_if
    foo(T value) {}

    void main() {
        foo(10);      // OK
        foo(3.14);    // Not considered during overload resolution
    }
    ```

---
### 7. Concepts (C++20)
A concept is a named set of requirements. The definition of a concept must appear at namespace scope.
Concepts provide a more readable way to constrain templates.
Concepts are generally preferred over SFINAE in modern C++ because they produce clearer code and better compiler diagnostics.
Syntax:
    ```cpp
    template<class T, class U>
    concept Derived = std::is_base_of<U, T>::value;
    ```
https://www.cppstories.com/2021/concepts-intro/